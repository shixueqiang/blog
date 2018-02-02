---
title: CentOS7安装配置redis-3.0.0
id: 25
categories:
  - 未分类
date: 2016-07-22 17:11:44
tags:
---

<a name="top"></a>
<div id="main">
<div id="header">

# [清园](http://www.cnblogs.com/kreo/)

沉没的Atlantis

</div>
<div id="post_detail">
<div class="post">

## [CentOS7安装配置redis-3.0.0](http://www.cnblogs.com/kreo/p/4399612.html)

<div class="postText">
<div id="cnblogs_post_body">

**一.安装必要包**
<div class="cnblogs_code">
<pre>yum install gcc</pre>
</div>
**二.linux下安装**
<div class="cnblogs_code">
<div class="cnblogs_code_toolbar"><span class="cnblogs_code_copy"><a title="复制代码">![复制代码](http://common.cnblogs.com/images/copycode.gif)</a></span></div>
<pre>#下载
wget http://download.redis.io/releases/redis-3.0.0.tar.gz
tar zxvf redis-3.0.0.tar.gz
cd redis-3.0.0
#如果不加参数,linux下会报错
make MALLOC=libc</pre>
<div class="cnblogs_code_toolbar"><span class="cnblogs_code_copy"><a title="复制代码">![复制代码](http://common.cnblogs.com/images/copycode.gif)</a></span></div>
</div>
** 安装好之后,启动文件**
<div class="cnblogs_code">
<pre>#启动redis
src/redis-server &amp;

#关闭redis
src/redis-cli shutdown</pre>
</div>
**测试redis**
<div class="cnblogs_code">
<div class="cnblogs_code_toolbar"><span class="cnblogs_code_copy"><a title="复制代码">![复制代码](http://common.cnblogs.com/images/copycode.gif)</a></span></div>
<pre>$ src/redis-cli
127.0.0.1:6379&gt; set foo bar
OK
127.0.0.1:6379&gt; get foo
"bar"
$</pre>
<div class="cnblogs_code_toolbar"><span class="cnblogs_code_copy"><a title="复制代码">![复制代码](http://common.cnblogs.com/images/copycode.gif)</a></span></div>
</div>
测试成功

&nbsp;

**3.redis cluster集群搭建**

**建立本机测试环境**

建立运行目录
<div class="cnblogs_code">
<div class="cnblogs_code_toolbar"><span class="cnblogs_code_copy"><a title="复制代码">![复制代码](http://common.cnblogs.com/images/copycode.gif)</a></span></div>
<pre>#建立redis运行目录
mkdir -p redis-server/7000/
#复制默认的配置文档
cp redis-3.0.0/redis.conf redis-server/redis.default.conf
#把编译好的server复制到运行目录
cp redis-3.0.0/src/redis-server redis-server/7000/</pre>
<div class="cnblogs_code_toolbar"><span class="cnblogs_code_copy"><a title="复制代码">![复制代码](http://common.cnblogs.com/images/copycode.gif)</a></span></div>
</div>
建立独立配置文件
<div class="cnblogs_code">
<pre>#在7000目录下建立redis的配置文档
vim redis-server/7000/redis.conf</pre>
</div>
文件内容
<div class="cnblogs_code">
<pre>#redis-server/7000/redis.conf
include /root/redis-server/redis.default.conf
pidfile /var/run/redis-7000.pid
port 7000
cluster-enabled yes
cluster-config-file redis-node-7000.conf
cluster-node-timeout 5000
appendonly yes</pre>
</div>
复制运行目录(模拟集群环境)
<div class="cnblogs_code">
<pre>#复制目录
cp -R 7000/ 7001/
cp -R 7000/ 7002/
cp -R 7000/ 7003/
cp -R 7000/ 7004/
cp -R 7000/ 7005/</pre>
</div>
修改相应配置文件的端口和文件名

建立启动脚本 redis-server/redis-start.sh
<div class="cnblogs_code">
<pre>#!/bin/sh
/root/redis-server/7000/redis-server /root/redis-server/7000/redis.conf &amp;
/root/redis-server/7001/redis-server /root/redis-server/7001/redis.conf &amp;
/root/redis-server/7002/redis-server /root/redis-server/7002/redis.conf &amp;
/root/redis-server/7003/redis-server /root/redis-server/7003/redis.conf &amp;
/root/redis-server/7004/redis-server /root/redis-server/7004/redis.conf &amp;
/root/redis-server/7005/redis-server /root/redis-server/7005/redis.conf &amp;</pre>
</div>
**配置集群**

安装ruby
<div class="cnblogs_code">
<pre>yum install ruby-devel.x86_64</pre>
</div>
安装redis gem
<div class="cnblogs_code">
<pre># gem install redis
Fetching: redis-3.2.1.gem (100%)
Successfully installed redis-3.2.1
Parsing documentation for redis-3.2.1
Installing ri documentation for redis-3.2.1
1 gem installed</pre>
</div>
使用脚本建立集群机制

_在create的时候,加上参数--replicas 1 表示为每个master分配一个salve,如例子,则是3个master 3个salve_
<div class="cnblogs_code">
<pre># ./redis-trib.rb create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
&gt;&gt;&gt; Creating cluster
Connecting to node 127.0.0.1:7000: OK
Connecting to node 127.0.0.1:7001: OK
Connecting to node 127.0.0.1:7002: OK
Connecting to node 127.0.0.1:7003: OK
Connecting to node 127.0.0.1:7004: OK
Connecting to node 127.0.0.1:7005: OK
&gt;&gt;&gt; Performing hash slots allocation on 6 nodes...
Using 6 masters:
127.0.0.1:7000
127.0.0.1:7001
127.0.0.1:7002
127.0.0.1:7003
127.0.0.1:7004
127.0.0.1:7005
M: f3dd250e4bc145c8b9f864e82f65e00d1ba627be 127.0.0.1:7000
   slots:0-2730 (2731 slots) master
M: 1ba602ade59e0770a15128b193f2ac29c251ab5e 127.0.0.1:7001
   slots:2731-5460 (2730 slots) master
M: 4f840a70520563c8ef0d7d1cc9d5eaff6a1547a2 127.0.0.1:7002
   slots:5461-8191 (2731 slots) master
M: 702adc7ae9caf1f6702987604548c6fc1d22e813 127.0.0.1:7003
   slots:8192-10922 (2731 slots) master
M: 4f87a11d2ea6ebe9caf02c9dbd827a3dba8a53cf 127.0.0.1:7004
   slots:10923-13652 (2730 slots) master
M: 216bbb7da50bd130da16a327c76dc6d285f731b3 127.0.0.1:7005
   slots:13653-16383 (2731 slots) master
Can I set the above configuration? (type 'yes' to accept): yes
&gt;&gt;&gt; Nodes configuration updated
&gt;&gt;&gt; Assign a different config epoch to each node
&gt;&gt;&gt; Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join...
&gt;&gt;&gt; Performing Cluster Check (using node 127.0.0.1:7000)
M: f3dd250e4bc145c8b9f864e82f65e00d1ba627be 127.0.0.1:7000
   slots:0-2730 (2731 slots) master
M: 1ba602ade59e0770a15128b193f2ac29c251ab5e 127.0.0.1:7001
   slots:2731-5460 (2730 slots) master
M: 4f840a70520563c8ef0d7d1cc9d5eaff6a1547a2 127.0.0.1:7002
   slots:5461-8191 (2731 slots) master
M: 702adc7ae9caf1f6702987604548c6fc1d22e813 127.0.0.1:7003
   slots:8192-10922 (2731 slots) master
M: 4f87a11d2ea6ebe9caf02c9dbd827a3dba8a53cf 127.0.0.1:7004
   slots:10923-13652 (2730 slots) master
M: 216bbb7da50bd130da16a327c76dc6d285f731b3 127.0.0.1:7005
   slots:13653-16383 (2731 slots) master
[OK] All nodes agree about slots configuration.
&gt;&gt;&gt; Check for open slots...
&gt;&gt;&gt; Check slots coverage...
[OK] All 16384 slots covered.</pre>
</div>
如果需要全部重新自动配置,则删除所有的配置好的cluster-config-file,重新启动所有的redis-server,然后重新执行配置命令即可

</div>
</div>
</div>
</div>
</div>