---
title: 记录一次java堆外内存泄漏排查
date: 2023-12-28 11:28:33
categories:
- Java
- JVM
tags:
- JVM
---

线上运行的网络请求代理服务通过Grafana面板查看内存占用一直在上涨，给程序设置的内存上限为20G，已经占用8G，而且还在持续上涨，遂对程序进行排查。
通常直接使用jcmd直接查看堆内存占用情况，然后使用jmap工具导出堆内存快照，使用MAT工具分析堆内存快照，查看内存泄漏情况。
jcmd查看GC堆内存占用情况如下：
```bash
bash-4.4# jcmd 1 GC.heap_info
1:
 garbage-first heap   total 7536640K, used 6807512K [0x0000000300000000, 0x0000000800000000)
  region size 16384K, 16 young (262144K), 1 survivors (16384K)
 Metaspace       used 116022K, committed 116736K, reserved 1155072K
  class space    used 13513K, committed 13952K, reserved 1048576K
```
<!--more-->

可以看到堆内存已经使用了6.8G.
使用jmap导出堆内存快照：
```bash
bash-4.4# jmap -dump:format=b,file=dumpfile.hprof 1 

```

使用MAT工具分析堆内存快照，查看内存泄漏情况，但堆内存显示只有340m，和实际GC堆内存有6.8g，完全不符合，这是为什么呢？

**这种情况一般是因为程序使用了堆外内存，而堆外内存是不会被jmap导出的，所以导出的堆内存快照中不会包含堆外内存的信息。**

排查代码发现程序序列化使用了protobuf，protobuf使用了java.nio.ByteBuffer来存储序列化后的数据，而ByteBuffer是堆外内存，所以导致了堆外内存泄漏。
这种使用的堆外内存称为Direct Buffer，它是通过java.nio.ByteBuffer.allocateDirect()方法分配的，它在堆外内存中存储数据。这种缓冲区适用于需要与底层I/O系统进行直接交互的场景，如网络编程或高性能文件操作。

默认不设置jvm参数时Direct Buffer与-Xmx（堆最大大小）参数相同,因此可是配置启动参数来限制此分配内存的大小：
```bash
# jvm参数
-XX:MaxDirectMemorySize=256M
# jdk nio 包中设置最高分配内存的大小
-Djdk.nio.maxCachedBufferSize=262144
```
同时可以使用`-XX:NativeMemoryTracking`来弃用跟踪本地内存使用情况，这样可以使用jcmd来查看本地内存使用情况：
```bash
jcmd 1 VM.native_memory
```


[Fixing Java's ByteBuffer native memory "leak"](https://www.evanjones.ca/java-bytebuffer-leak.html)
[openjdk nio utils](https://github.com/AdoptOpenJDK/openjdk-jdk11/blob/master/src/java.base/share/classes/sun/nio/ch/Util.java)