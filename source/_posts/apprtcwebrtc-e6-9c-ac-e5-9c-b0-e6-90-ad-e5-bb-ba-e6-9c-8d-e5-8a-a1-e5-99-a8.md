---
title: apprtc(webrtc) 本地搭建服务器
id: 66
categories:
  - 未分类
date: 2016-09-13 14:53:44
tags:
---

<span style="font-size: large;">这两天测试了下webrtc的效果，不知道是不是手机比较渣，画面并不是很清晰，先来说说服务器的部署。部署环境ubuntu14.04 32位。
</span>

<span style="font-size: large;">一、服务器组成</span>

<span style="font-size: large;">      1、AppRTC 房间服务器  [https://github.com/webrtc/apprtc](https://github.com/webrtc/apprtc)</span>

<span style="font-size: large;">      2、Collider  信令服务器  上边源码里自带</span>

<span style="font-size: large;">      3、coTurn   穿透服务器   [https://github.com/coturn/coturn](https://github.com/coturn/coturn)</span>

<span style="font-size: large;">      4、需要自己实现coTurn连接信息接口，主要返回用户名、密码和turn配置信息，通常叫做TURN REST API,不实现这个接口的话AppRTCDemo连不上服务器，浏览器访问的话可以正常访问。</span>

<span style="font-size: large;">二、AppRTC房间服务器</span>

<span style="font-size: large;">      1、下载代码</span>

<span style="font-size: large;">      2、安装依赖</span>

<span style="font-size: large;">            sudo apt-get install nodejs</span>

<span style="font-size: large;">            sudo npm install -g npm</span>

<span style="font-size: large;">            sudo apt-get install nodejs-legacy</span>

<span style="font-size: large;">            sudo npm -g install grunt-cli</span>

<span style="font-size: large;">            切换到源码目录</span>

<span style="font-size: large;">            cd apprtc-master</span>

<span style="font-size: large;">            npm install</span>

<span style="font-size: large;">            sudo apt-get install python-webtest</span>

<span style="font-size: large;">            grunt build</span>

<span style="font-size: large;">            编译之后会多出out目录</span>

<span style="font-size: large;">            运行还依赖 Google App Engine SDK for  Python [需翻墙
](https://cloud.google.com/appengine/downloads#Google_App_Engine_SDK_for_Python)</span>

<span style="font-size: large;">            下载完后设置环境变量
</span>

<span style="font-size: large;">            sudo gedit /etc/profile
</span>

<span style="font-size: large;">            export PATH=$PATH:/home/google_appengine</span>

<span style="font-size: large;">            source /etc/profile
</span>

<span style="font-size: large;">        3、修改配置文件</span>

<span style="font-size: large;">             主要是src/app_engine目录下的apprtc.py和constants.py</span>

<span style="font-size: large;">             首先是<span style="font-size: large;">constants.py</span>：</span>

<span style="font-size: large;">             修改TURN_BASE_URL = 'http://192.168.214.129:80'  这个是上边提到的连接信息接口的地址</span>

<span style="font-size: large;">             TURN_URL_TEMPLATE = '%s/turn.php?username=%s&amp;key=%s'</span>

<span style="font-size: large;">             CEOD_KEY = 和coturn turnserver.conf <span style="font-size: large;">static-auth-secret一致</span>
</span>

<span style="font-size: large;">             WSS_INSTANCES = [{
WSS_INSTANCE_HOST_KEY: '192.168.214.129:8089',
WSS_INSTANCE_NAME_KEY: 'wsserver-std',
WSS_INSTANCE_ZONE_KEY: 'us-central1-a'
}, {
WSS_INSTANCE_HOST_KEY: '192.168.214.129:8089',
WSS_INSTANCE_NAME_KEY: 'wsserver-std-2',
WSS_INSTANCE_ZONE_KEY: 'us-central1-f'
}]</span>

<span style="font-size: large;">             apprtc.py:</span>

<span style="font-size: large;">             修改get_wss_parameters(request) 下的</span>

<span style="font-size: large;">              if wss_tls and wss_tls == 'false':
wss_url = 'ws://' + wss_host_port_pair + '/ws'
wss_post_url = 'http://' + wss_host_port_pair
else:
wss_url = 'ws://' + wss_host_port_pair + '/ws'
wss_post_url = 'http://' + wss_host_port_pair</span>

<span style="font-size: large;">              主要是把原来的wss和https的scheme都改为ws和http，不要让客户端和浏览器去使用ssl连接，如果有第三   方根证书的签名机构颁发的证书就不需要这样了。</span>

<span style="font-size: large;">               修改完后重新grunt build下。</span>

<span style="font-size: large;">           4、启动</span>

<span style="font-size: large;">                dev_appserver.py --host=0.0.0.0 ./out/app_engine</span>

<span style="font-size: large;">三、Collider信令服务器</span>

<span style="font-size: large;">        1、安装依赖</span>

<span style="font-size: large;">         sudo apt-get install golang-go</span>

<span style="font-size: large;">         2、在home目下创建文件夹</span>

<span style="font-size: large;">         mkdir -p ~/collider_root        并在collider_root目录下创建src目录</span>

<span style="font-size: large;">         设置GOPATH环境变量    export GOPATH=~/collider_root</span>

<span style="font-size: large;">         将apprtc/src/collider目录下的三个文件夹都拷贝到collider_root/src下</span>

<span style="font-size: large;">         进入到collider_root/src,开始编译安装collider，准备好翻墙</span>

<span style="font-size: large;">         go get collidermain</span>

<span style="font-size: large;">         go install collidermain</span>

<span style="font-size: large;">         成功编译后会在collider_root目录下生成bin和pkg目录，执行文件在bin下。</span>

<span style="font-size: large;">        3、运行</span>

<span style="font-size: large;">          修改collider_root/src/collidermain/main.go填上自己ip地址</span>

<span style="font-size: large;">          var roomSrv = flag.String("room-server", "http://192.168.214.129:8080/", "The origin of the room server")</span>

<span style="font-size: large;">          启动</span>

<span style="font-size: large;">          ~/collider_root/bin/collidermain -port=8089 -tls=false</span>

<span style="font-size: large;">四、coTurn 打洞服务器</span>

<span style="font-size: large;">        1、下载[http://turnserver.open-sys.org/downloads/](http://turnserver.open-sys.org/downloads/)</span>

<span style="font-size: large;">         找个适合自己linux系统的，我这里是ubuntu32位所以选了turnserver-4.2.1.2-debian-wheezy-ubuntu-mint-x86-32bits.tar.gz</span>

<span style="font-size: large;">         下载完后解压进入解压目录</span>

<span style="font-size: large;">          cat INSTALL     查看安装须知</span>

<span style="font-size: large;">          sudo apt-get install gdebi-core</span>

<span style="font-size: large;">          sudo gdebi coturn_4.2.2.2-1_i386.deb</span>

<span style="font-size: large;">         2、编辑配置文件</span>

<span style="font-size: large;">           sudo gedit /etc/turnserver.conf</span>

<span style="font-size: large;">           listening-device=eth0</span>

<span style="font-size: large;">           listening-port=3478
</span>

<span style="font-size: large;">           relay-device=eth1</span>

<span style="font-size: large;">           min-port=49152
max-port=65535
</span>

<span style="font-size: large;">           Verbose</span>

<span style="font-size: large;">           fingerprint</span>

<span style="font-size: large;">           lt-cred-mech</span>

<span style="font-size: large;">           use-auth-secret</span>

<span style="font-size: large;">           static-auth-secret=填写自己的密钥可不修改
</span>

<span style="font-size: large;">           stale-nonce</span>

<span style="font-size: large;">           cert=/usr/local/etc/turn_server_cert.pem</span>

<span style="font-size: large;">           pkey=/usr/local/etc/turn_server_pkey.pem</span>

<span style="font-size: large;">           no-loopback-peers</span>

<span style="font-size: large;">           no-multicast-peers</span>

<span style="font-size: large;">           mobility</span>

<span style="font-size: large;">           no-cli</span>

<span style="font-size: large;">         3、生成签名证书</span>

<span style="font-size: large;">           sudo openssl req -x509 -newkey  rsa:2048 -keyout<span style="font-size: large;">/usr/local/etc/turn_server_pkey.pem</span> -out <span style="font-size: large;">/usr/local/etc/turn_server_cert.pem</span> -days 99999 -nodes</span>

<span style="font-size: large;">4、启动</span>

<span style="font-size: large;">          service coturn start</span>

<span style="font-size: large;">五、coTurn连接信息接口</span>

<span style="font-size: large;">       TURN REST API [标准参考文档](http://tools.ietf.org/html/draft-uberti-behave-turn-rest-00)</span>

<span style="font-size: large;">       返回json结果示例：</span>

<span style="font-size: large;">       {"username":"1456904882:1231244","password":"jAph7EHMLuPJuxLLC1uRiI3kvq4=","ttl":86400,"uris":["turn:192.168.214.129:3478?transport=udp","turn:192.168.214.129:3478?transport=tcp","turn:192.168.214.129:3479?transport=udp","turn:192.168.214.129:3479?transport=tcp"]}</span>

<span style="font-size: large;">       1、username字段需要以timestamp + ":"  + username的形式输出</span>

<span style="font-size: large;">       2、<span style="font-size: large;">响应的 password 字段，需要以 HMAC-SHA1 算法计算得出，**公式为：【base64_encode( hmac( key,
**</span></span>

<span style="font-size: large;">**            <span style="font-size: large;"><b>username ) )】**。</span></b><span style="font-size: large;">此处的 key，为 coTurn 服务器配置中的 “static-auth-secret”值。以 key 作为      hmac 算法的密<span style="font-size: large;">钥，turn-username 为被计算的内容，得出的 hmac 摘要后，经 base64 编码得到最终密码。</span></span></span>

<span style="font-size: large;"><span style="font-size: large;"><span style="font-size: large;">       3、uris为后台配置好的，我这里写死</span></span></span>

<span style="font-size: large;"><span style="font-size: large;"><span style="font-size: large;">       4、很简单的接口，用惯了java，这里我准备用php，写起来比java快</span></span></span>

<span style="font-size: large;"><span style="font-size: large;"><span style="font-size: large;">       </span></span></span>
<pre class="html">&lt;?php

	$request_username = $_GET["username"];
	if(empty($request_username)) {
		echo "username == null";
		exit;
	}
	$request_key = $_GET["key"];
	$time_to_live = 600;
	$timestamp = time() + $time_to_live;//失效时间
	$response_username = $timestamp.":".$_GET["username"];
	$response_key = $request_key;
	if(empty($response_key))
		$response_key = "密钥";//constants.py中CEOD_KEY

	$response_password = getSignature($response_username, $response_key);

	$jsonObj = new Response();
	$jsonObj-&gt;username = $response_username;
	$jsonObj-&gt;password = $response_password;
	$jsonObj-&gt;ttl = 86400;
	$jsonObj-&gt;uris = array("turn:192.168.214.129:3478?transport=udp","turn:192.168.214.129:3478?transport=tcp","turn:192.168.214.129:3479?transport=udp","turn:192.168.214.129:3479?transport=tcp");

	echo json_encode($jsonObj);

	/** 
         * 使用HMAC-SHA1算法生成签名值 
         * 
	 * @param $str 源串 
         * @param $key 密钥 
         * 
         * @return 签名值 
         */  
    function getSignature($str, $key) {  
        $signature = "";  
        if (function_exists('hash_hmac')) {  
            $signature = base64_encode(hash_hmac("sha1", $str, $key, true));  
        } else {  
            $blocksize = 64;  
            $hashfunc = 'sha1';  
            if (strlen($key) &gt; $blocksize) {  
                $key = pack('H*', $hashfunc($key));  
            }  
            $key = str_pad($key, $blocksize, chr(0x00));  
            $ipad = str_repeat(chr(0x36), $blocksize);  
            $opad = str_repeat(chr(0x5c), $blocksize);  
            $hmac = pack(  
                    'H*', $hashfunc(  
                            ($key ^ $opad) . pack(  
                                    'H*', $hashfunc(  
                                            ($key ^ $ipad) . $str  
                                    )  
                            )  
                    )  
            );  
            $signature = base64_encode($hmac);  
        }  
        return $signature;  
       }  

	class Response {
		public $username = "";
		public $password = "";
		public $ttl = "";
		public $uris = array("");
	}

?&gt;</pre>
5、拿nginx部署下，怎么部署自行百度

<span style="font-size: large;"><span style="font-size: large;"><span style="font-size: large;">六、测试</span></span></span>

<span style="font-size: large;"><span style="font-size: large;"><span style="font-size: large;">       部署成功后可在浏览器输入http://192.168.214.129:8080创建房间</span></span></span>

appRTCDemo连接也改成<span style="font-size: large;"><span style="font-size: large;"><span style="font-size: large;">http://192.168.214.129:8080</span></span></span>即可

<span style="font-size: large;"><span style="font-size: large;"><span style="font-size: large;">七、参考连接</span></span></span>

<span style="font-size: large;"><span style="font-size: large;"><span style="font-size: large;">       [http://www.mamicode.com/info-detail-513556.html](http://www.mamicode.com/info-detail-513556.html)</span></span></span>

<span style="font-size: large;"><span style="font-size: large;"><span style="font-size: large;">       [http://www.jianshu.com/p/c55ecf5a3fcf](http://www.jianshu.com/p/c55ecf5a3fcf)</span></span></span>

<span style="font-size: large;"><span style="font-size: large;"><span style="font-size: large;">       [http://www.bubuko.com/infodetail-938737.html](http://www.bubuko.com/infodetail-938737.html)l
</span></span></span>

<span style="font-size: large;"><span style="font-size: large;"><span style="font-size: large;">八、关于webrtc编译
</span></span></span>

<span style="font-size: large;"><span style="font-size: large;"><span style="font-size: large;">       我只编译了android的版本，公司网速慢死，找了个国内下好的代码，分享到网盘了[http://pan.baidu.com/s/1kUyNE55](http://pan.baidu.com/s/1kUyNE55)</span></span></span>

<span style="font-size: large;"><span style="font-size: large;"><span style="font-size: large;">       AppRTCDemo的工程源码连接:[AppRTCDemo工程源码](http://download.csdn.net/detail/s569646547/9451013)</span></span></span>

<span style="font-size: large;">       运行截图：</span>

<span style="font-size: large;">       ![](http://img.blog.csdn.net/20160302182354507?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)</span>
<div></div>
&nbsp;