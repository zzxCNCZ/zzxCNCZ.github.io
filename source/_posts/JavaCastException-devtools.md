---
title: JavaCastException-devtools
date: 2018-07-26 17:10:05
categories:
- Java
- SpringBoot
tags:
- JavaCastException
---
## 研究quartz的过程中,碰到一个bug,不过早晚也会遇到的bug.经过面向搜索引擎编程总算找到
## 了问题所在,再次记录一下。
- 先上代码
```javascript
@Override
 protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
 ScheduleJobEntity scheduleJobtest=new ScheduleJobEntity();
 System.out.println(context.getMergedJobDataMap()
     .get(ScheduleJobEntity.JOB_PARAM_KEY).getClass().getClassLoader());
 System.out.println(scheduleJobtest.getClass().getClassLoader());
 if (context.getMergedJobDataMap()
     .get(ScheduleJobEntity.JOB_PARAM_KEY) instanceof ScheduleJobEntity) {
   scheduleJobtest = (ScheduleJobEntity) context.getMergedJobDataMap()
       .get(ScheduleJobEntity.JOB_PARAM_KEY);
 }
 .....省略
 } catch (Exception e) {
   logger.error("任务执行失败，任务ID：" + scheduleJob.getJobId(), e);

   //任务执行总时长
   long times = System.currentTimeMillis() - startTime;
   log.setTimes((int)times);

   //任务状态    0：成功    1：失败
   log.setStatus(1);
   log.setError(StringUtils.substring(e.toString(), 0, 2000));
 }finally {
   scheduleJobLogService.insert(log);
 }
 }
```
- 上面代码中的context 包含和任务执行的entity,但是是个object,需要转换一下，一开始
- 使用强转,运行到转换的地方就会报出JavaCastException: com.zzx.springboot.modules.job.entity.ScheduleJobEntity can not be casted to
com.zzx.springboot.modules.job.entity.ScheduleJobEntity
- 明明是两个一样类型的entity,但就是不能转换,经过多方查找,总算找到问题所在,
- 原因是因为devtools的热部署导致了我的ScheduleJobEntit实体是由devtools
的restartClassLoader加载的,导致不是原生的appclassloader
- 解决方案
1. 不用devtools
2. idea安装Jrebel插件(好用的一笔)
