---
title: flink实际应用-低频数据窗口关闭不掉的问题
date: 2023-05-08 16:04:14
categories:
- flink 
tags:
- flink
---

### 问题描述
flink消费kafka流式数据时，使用数据的时间作为水印，当数据流中出现低频数据时，窗口不会关闭，导致数据一直在内存中。
**实际场景：**
假设当前时间为 2023-05-08 16:01:30，从kafka消费一条数据中的时间戳为2023:05:08 16:01:30，id为:A,并且以id为key，每5分钟统计一次。那么此时窗口的结束时间为2023-05-08 16:05:00，此时窗口不会关闭，因为在2023:05:08 16:05:00之前没有数据进来，要触发此窗口关闭必须消费一条超过2023-05-08 16:05:00并且id为A的数据。假如id为A的数据迟迟不来，则此窗口一直不会关闭，导致数据一直在内存中。

<!-- more -->

### 解决方案
出现上述情况可能也与使用方式问题有关，flink一般用于处理高频数据，大量数据时使用。还有就是窗口设置的过大等都可能遇到上述窗口关闭不掉的情况。

如何使窗口关闭，可以从两方面入手：
1. 改变现有窗口机制。
2. 修改数据的水印。



#### 改变现有窗口机制（自定义触发器trigger）
改变窗口机制，即原本依赖事件时间EventTime来生成窗口关闭时间，改为使用处理时间ProcessingTime来生成窗口关闭时间。
整体的思想就是：当watermark不能满足关窗条件时，我们给注册一个晚于事件时间的处理时间定时器使它一定能达到关窗条件。
这样处理的话，还需要在事件时间方法中删除处理时间定时器，同时在处理时间中删除事件时间定时器，最后别忘记清除两个定时器。

```java

import org.apache.flink.streaming.api.windowing.triggers.Trigger;
import org.apache.flink.streaming.api.windowing.triggers.TriggerResult;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;

public class HttpTraceTrigger extends Trigger<Object, TimeWindow> {
    private long processTime = 0L;

    @Override
    public TriggerResult onElement(Object obj, long l, TimeWindow window, TriggerContext ctx) throws Exception {
        if (window.maxTimestamp() <= ctx.getCurrentWatermark()) {
            // if the watermark is already past the window fire immediately
            return TriggerResult.FIRE;
        } else {
            ctx.registerEventTimeTimer(window.maxTimestamp());
//            System.out.println( DateUtil.format(new Date(window.maxTimestamp()), "yyyy-MM-dd hh:mm:ss"));
//            System.out.println( DateUtil.format(new Date(window.maxTimestamp() + 30000L), "yyyy-MM-dd hh:mm:ss"));
            long systemTime = System.currentTimeMillis();
            if (systemTime < window.maxTimestamp()) {
                processTime = window.maxTimestamp() + 30 * 1000L;
            } else {
                processTime = systemTime + 30 * 1000L;
            }
//            System.out.println( DateUtil.format(new Date(processTime), "yyyy-MM-dd hh:mm:ss"));
            ctx.registerProcessingTimeTimer(processTime);
            return TriggerResult.CONTINUE;
        }
    }

    @Override
    public TriggerResult onProcessingTime(long l, TimeWindow window, TriggerContext ctx) throws Exception {
        ctx.deleteEventTimeTimer(window.maxTimestamp());
        return TriggerResult.FIRE;
    }

    @Override
    public TriggerResult onEventTime(long time, TimeWindow window, TriggerContext ctx) throws Exception {
        if (time == window.maxTimestamp()) {
            ctx.deleteProcessingTimeTimer(processTime);
            return TriggerResult.FIRE;
        } else {
            return TriggerResult.CONTINUE;
        }
    }

    @Override
    public void clear(TimeWindow window, TriggerContext ctx) throws Exception {
        ctx.deleteEventTimeTimer(window.maxTimestamp());
        ctx.deleteProcessingTimeTimer(processTime);
    }

    public String toString() {
        return "HttpTraceTrigger()";
    }

    public static HttpTraceTrigger create() {
        return new HttpTraceTrigger();
    }
}

```
以上代码定义了一个触发器，在onElement时判断窗口结束时间与当前系统时间比较,如果大于当前系统时间，表明消费的是实时数据，如果小于则为历史数据。如果是实时数据。

则注册一个为窗口结束时间+30秒的ProcessingTimeTimer，如果是历史数据，则注册一个为当前系统时间+30秒的ProcessingTimeTimer。

这样在消费数据后，都会等待30秒时间，然后窗口就会自动关闭。在实际业务中可以依据实际消费速度和业务需要来定义需要等待的时间。


#### 修改数据的水印
主要思路是，周期性的产生水印watermark，来触发eventTime窗口关闭。比如以上例子中id为A的数据，因为只来了一条，需要后续数据来
才能出发窗口关闭，如果周期性的产生watermark就能关闭窗口。
实际解决方案是：自定义watermark strategy,重写getCurrentWatermark方法。getCurrentWatermark方法对于实时数据和历史数据的
产生策略也需要按实际业务惊醒判断。
此思路不推荐，也比较麻烦。