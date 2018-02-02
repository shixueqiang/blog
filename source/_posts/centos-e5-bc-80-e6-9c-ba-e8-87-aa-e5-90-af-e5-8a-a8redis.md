---
title: Centos开机自启动redis
id: 27
categories:
  - 未分类
date: 2016-07-22 17:12:40
tags:
---

<div class="blog-abstract">**摘要**
<div>Centos开机自启动redis</div>
</div>
<div class="blog-body">
<div class="BlogContent">

*   修改redis.conf，打开后台运行选项：

    <span class="hljs-comment"># By default Redis does not run as a daemon. Use 'yes' if you need it.</span>
    <span class="hljs-comment"># Note that Redis will write a pid file in /var/run/redis.pid when daemonized.</span>
    <span class="hljs-attribute">daemonize</span> <span class="hljs-literal">yes</span>`</pre>

*   编写脚本，vim /etc/init.d/redis:
    <pre class="brush:shell; toolbar: true; auto-links: false;">`<span class="hljs-comment"># chkconfig: 2345 10 90</span>
    <span class="hljs-comment"># description: Start and Stop redis</span>

    PATH=/usr/<span class="hljs-built_in">local</span>/bin:/sbin:/usr/bin:/bin

    REDISPORT=6379 <span class="hljs-comment">#实际环境而定</span>
    EXEC=/usr/<span class="hljs-built_in">local</span>/redis/src/redis-server <span class="hljs-comment">#实际环境而定</span>
    REDIS_CLI=/usr/<span class="hljs-built_in">local</span>/redis/src/redis-cli <span class="hljs-comment">#实际环境而定</span>

    PIDFILE=/var/run/redis.pid
    CONF=<span class="hljs-string">"/usr/local/redis/redis.conf"</span> <span class="hljs-comment">#实际环境而定</span>

    <span class="hljs-keyword">case</span> <span class="hljs-string">"<span class="hljs-variable">$1</span>"</span> <span class="hljs-keyword">in</span>
            start)
                    <span class="hljs-keyword">if</span> [ <span class="hljs-_">-f</span> <span class="hljs-variable">$PIDFILE</span> ]
                    <span class="hljs-keyword">then</span>
                            <span class="hljs-built_in">echo</span> <span class="hljs-string">"<span class="hljs-variable">$PIDFILE</span> exists, process is already running or crashed."</span>
                    <span class="hljs-keyword">else</span>
                            <span class="hljs-built_in">echo</span> <span class="hljs-string">"Starting Redis server..."</span>
                            <span class="hljs-variable">$EXEC</span> <span class="hljs-variable">$CONF</span>
                    <span class="hljs-keyword">fi</span>
                    <span class="hljs-keyword">if</span> [ <span class="hljs-string">"$?"</span>=<span class="hljs-string">"0"</span> ]
                    <span class="hljs-keyword">then</span>
                            <span class="hljs-built_in">echo</span> <span class="hljs-string">"Redis is running..."</span>
                    <span class="hljs-keyword">fi</span>
                    ;;
            stop)
                    <span class="hljs-keyword">if</span> [ ! <span class="hljs-_">-f</span> <span class="hljs-variable">$PIDFILE</span> ]
                    <span class="hljs-keyword">then</span>
                            <span class="hljs-built_in">echo</span> <span class="hljs-string">"<span class="hljs-variable">$PIDFILE</span> exists, process is not running."</span>
                    <span class="hljs-keyword">else</span>
                            PID=$(cat <span class="hljs-variable">$PIDFILE</span>)
                            <span class="hljs-built_in">echo</span> <span class="hljs-string">"Stopping..."</span>
                            <span class="hljs-variable">$REDIS_CLI</span> -p <span class="hljs-variable">$REDISPORT</span> SHUTDOWN
                            <span class="hljs-keyword">while</span> [ -x <span class="hljs-variable">$PIDFILE</span> ]
                            <span class="hljs-keyword">do</span>
                                    <span class="hljs-built_in">echo</span> <span class="hljs-string">"Waiting for Redis to shutdown..."</span>
                                    sleep 1
                            <span class="hljs-keyword">done</span>
                            <span class="hljs-built_in">echo</span> <span class="hljs-string">"Redis stopped"</span>
                    <span class="hljs-keyword">fi</span>
                    ;;
            restart|force-reload)
                    <span class="hljs-variable">${0}</span> stop
                    <span class="hljs-variable">${0}</span> start
                    ;;
            *)
                    <span class="hljs-built_in">echo</span> <span class="hljs-string">"Usage: /etc/init.d/redis {start|stop|restart|force-reload}"</span> &gt;&amp;2
                    <span class="hljs-built_in">exit</span> 1
    <span class="hljs-keyword">esac</span>`</pre>

*   执行权限：
    <pre class="brush:shell; toolbar: true; auto-links: false;">`<span class="hljs-attribute">chmod</span> +x /etc/init.d/redis`</pre>

*   开机自启动：
    <pre class="brush:shell; toolbar: true; auto-links: false;">`<span class="hljs-comment"># 尝试启动或停止redis</span>
    <span class="hljs-attribute">service</span> redis start
    service redis stop

    <span class="hljs-comment"># 开启服务自启动</span>
    chkconfig redis <span class="hljs-literal">on</span>

</div>
</div>