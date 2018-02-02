---
title: idea failed to create a child event loop
id: 72
categories:
  - 未分类
date: 2016-09-13 14:56:35
tags:
---

idea14 有时候编译出现错误
<pre class="html">Error:Abnormal build process termination: 
13:15:08,830 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Found resource [logback.groovy] at [jar:file:/D:/Program%20Files/IntelliJ%20IDEA%2014.0.2/plugins/gradle/lib/gradle.jar!/logback.groovy]
13:15:08,831 |-ERROR in ch.qos.logback.classic.LoggerContext[default] - Groovy classes are not available on the class path. ABORTING INITIALIZATION.
Build process started. Classpath: /D:/Program Files/IntelliJ IDEA 14.0.2/lib/jps-launcher.jar;D:/Program Files (x86)/java/jdk1.7/lib/tools.jar;/D:/Program Files/IntelliJ IDEA 14.0.2/lib/optimizedFileManager.jar;D:/Program Files/IntelliJ IDEA 14.0.2/lib/ecj-4.4.jar
Error connecting to 127.0.0.1:61174; reason: failed to create a child event loop
java.lang.IllegalStateException: failed to create a child event loop
	at io.netty.util.concurrent.MultithreadEventExecutorGroup.&lt;init&gt;(MultithreadEventExecutorGroup.java:81)
	at io.netty.channel.MultithreadEventLoopGroup.&lt;init&gt;(MultithreadEventLoopGroup.java:50)
	at io.netty.channel.nio.NioEventLoopGroup.&lt;init&gt;(NioEventLoopGroup.java:72)
	at io.netty.channel.nio.NioEventLoopGroup.&lt;init&gt;(NioEventLoopGroup.java:58)
	at org.jetbrains.jps.cmdline.BuildMain.main(BuildMain.java:97)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:601)
	at org.jetbrains.jps.cmdline.Launcher.main(Launcher.java:58)
Caused by: io.netty.channel.ChannelException: failed to open a new selector
	at io.netty.channel.nio.NioEventLoop.openSelector(NioEventLoop.java:127)
	at io.netty.channel.nio.NioEventLoop.&lt;init&gt;(NioEventLoop.java:119)
	at io.netty.channel.nio.NioEventLoopGroup.newChild(NioEventLoopGroup.java:97)
	at io.netty.channel.nio.NioEventLoopGroup.newChild(NioEventLoopGroup.java:31)
	at io.netty.util.concurrent.MultithreadEventExecutorGroup.&lt;init&gt;(MultithreadEventExecutorGroup.java:77)
	... 9 more
Caused by: java.io.IOException: Unable to establish loopback connection</pre>
一、检测杀毒和防火墙；

二、 检查ping www.baidu.com 是否是通的，如果不通 ping ip是否是通的，重置网络环境，打开cmd ，输入 netsh winsock reset，重置网络环境，一般可恢复正常，有次我的乌龟svn也出现连接不上服务器的问题，重置网络后正常，严重怀疑360修改了网络配置文件。。。