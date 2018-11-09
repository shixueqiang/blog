---
title: apprtc(webrtc) 本地搭建服务器
id: 66
categories:
  - 未分类
date: 2018-11-09 14:53:44
tags:
---

### 一、服务器组成

1、AppRTC 房间服务器  [https://github.com/webrtc/apprtc](https://github.com/webrtc/apprtc)
2、Collider  信令服务器  上边源码里自带
3、coTurn   穿透服务器   [https://github.com/coturn/coturn](https://github.com/coturn/coturn)
4、需要自己实现coTurn连接信息接口，主要返回用户名、密码和turn配置信息，通常叫做TURN REST API,不实现这个接口的话AppRTCMobile连不上服务器，浏览器访问的话可以正常访问。

### 二、AppRTC房间服务器

1、下载代码
`git clone https://github.com/webrtc/apprtc.git`
2、编译
安装[Node.js](https://nodejs.org/en/)、[Grunt](https://gruntjs.com/)、[Google App Engine SDK for Python](https://cloud.google.com/appengine/downloads#Google_App_Engine_SDK_for_Python)，参考官方介绍。
切换到源码目录
安装依赖 `npm install`
3、修改配置文件
主要是src/app_engine目录下的apprtc.py和constants.py
首先是constants.py：
修改`ICE_SERVER_BASE_URL = 'http://192.168.0.114:8088'`这个是上边提到的连接信息接口的地址
```
WSS_INSTANCES = [{
    WSS_INSTANCE_HOST_KEY: '192.168.0.114:8089',
    WSS_INSTANCE_NAME_KEY: 'wsserver-std',
    WSS_INSTANCE_ZONE_KEY: 'us-central1-a'
}, {
    WSS_INSTANCE_HOST_KEY: '192.168.0.114:8089',
    WSS_INSTANCE_NAME_KEY: 'wsserver-std-2',
    WSS_INSTANCE_ZONE_KEY: 'us-central1-f'
}]
```
apprtc.py:
修改get_wss_parameters(request) 下的
```
if wss_tls and wss_tls == 'false':
    wss_url = 'ws://' + wss_host_port_pair + '/ws'
    wss_post_url = 'http://' + wss_host_port_pair
else:
    wss_url = 'ws://' + wss_host_port_pair + '/ws'
    wss_post_url = 'http://' + wss_host_port_pair
```
主要是把原来的wss和https的scheme都改为ws和http，不要让客户端和浏览器去使用ssl连接，如果有第三   方根证书的签名机构颁发的证书就不需要这样了。
编译 `grunt build`
4、启动
`<Google App Engine SDK PATH>/dev_appserver.py --host=192.168.0.114 ./out/app_engine`
### 三、Collider信令服务器

1、参考官网安装[golang](http://golang.org/doc/install)
2、设置环境变量GOPATH
`export GOPATH=<YOUR PATH>`
3、创建软连接
```
ln -s `pwd`/apprtc/src/collider/collider $GOPATH/src
ln -s `pwd`/apprtc/src/collider/collidermain $GOPATH/src
ln -s `pwd`/apprtc/src/collider/collidertest $GOPATH/src
```
4、安装依赖
`go get collidermain`
5、安装collidermain
`go install collidermain`
6、运行
修改apprtc/src/collidermain/main.go填上自己ip地址
```
var roomSrv = flag.String("room-server", "http://192.168.0.114:8080/", "The origin of the room server")
```
启动
`$GOPATH/bin/collidermain -port=8089 -tls=false`
### 四、coTurn 打洞服务器

1、下载[coturn](https://github.com/coturn/coturn)源码,编译安装
```
./configure --prefix=<install path>
make
make install
```
2、编辑配置文件turnserver.conf
```
#监听的网卡设备，根据自己机器填写
listening-device=enp0s25
listening-port=3478
relay-device=enp0s25
min-port=49152
max-port=65535
#日志级别
Verbose
fingerprint
lt-cred-mech
use-auth-secret
static-auth-secret=秘钥
stale-nonce
cert=/usr/local/etc/turn_server_cert.pem
pkey=/usr/local/etc/turn_server_pkey.pem
no-loopback-peers
no-multicast-peers
mobility
no-cli
```
3、生成签名证书
```
sudo openssl req -x509 -newkey  rsa:2048 --keyout /usr/local/etc/turn_server_pkey.pem -out /usr/local/etc/turn_server_cert.pem -days 99999 -nodes
```
4、启动
```
sudo ./turnserver -v -L 192.168.0.114  -a -f -r 192.168.0.114 -c ../etc/turnserver.conf
```
### 五、coTurn连接信息接口

返回json结果示例：
```
{"iceServers":[{
        "urls":["stun:192.168.0.114:3478","turn:192.168.0.114:3478"],
        "username":"1541754497:shixq",
        "credential":"yNC1AoeKuMfCvN6fTEvUZSVW77A="}]}
```

1、username字段需要以timestamp + ":"  + username的形式输出
2、接口示例，golang版本(使用gin)
```
package main

import (
	"crypto/hmac"
	"crypto/sha1"
	"encoding/base64"
	"fmt"
	"net/http"
	"strconv"
	"time"

	"github.com/gin-gonic/gin"
)

type IceServer struct {
	Urls       []string `json:"urls"`
	Username   string   `json:"username"`
	Credential string   `json:"credential"`
}

func main() {
	r := gin.Default()
	r.GET("/v1alpha/iceconfig", func(c *gin.Context) {
		username := c.DefaultQuery("username", "shixq")
		key := c.Query("key")
		key = "4080218913"
		time_to_live := 600
		timestamp := time.Now().Unix() + int64(time_to_live)
		ip := "192.168.0.114"
		var uris = []string{fmt.Sprintf("stun:%s:3478", ip), fmt.Sprintf("turn:%s:3478", ip)}
		c.JSON(200, gin.H{
			"iceServers": []IceServer{IceServer{uris, strconv.FormatInt(timestamp, 10) + ":" + username, signature(strconv.FormatInt(timestamp, 10)+":"+username, key)}},
		})
	})
	r.POST("/v1alpha/iceconfig", func(c *gin.Context) {
		username := c.DefaultPostForm("username", "shixq")
		key := c.PostForm("key")
		key = "4080218913"
		time_to_live := 600
		timestamp := time.Now().Unix() + int64(time_to_live)
		ip := "192.168.0.114"
		var uris = []string{fmt.Sprintf("stun:%s:3478", ip), fmt.Sprintf("turn:%s:3478", ip)}
		c.JSON(200, gin.H{
			"iceServers": []IceServer{IceServer{uris, strconv.FormatInt(timestamp, 10) + ":" + username, signature(strconv.FormatInt(timestamp, 10)+":"+username, key)}},
		})
	})
	http.ListenAndServe(":8088", r)
}

//使用HMAC-SHA1算法生成签名值
func signature(username, key string) string {
	mac := hmac.New(sha1.New, []byte(key))
	mac.Write([]byte(username))
	encoded := base64.StdEncoding.EncodeToString(mac.Sum(nil))
	return encoded
}
```
### 六、测试
部署成功后可在浏览器输入http://192.168.0.114:8080创建房间
appRTCMobile连接也改成http://192.168.214.129:8080即可

### 七、参考连接</span></span></span>

[http://www.mamicode.com/info-detail-513556.html](http://www.mamicode.com/info-detail-513556.html)
[http://www.jianshu.com/p/c55ecf5a3fcf](http://www.jianshu.com/p/c55ecf5a3fcf)
[http://www.bubuko.com/infodetail-938737.html](http://www.bubuko.com/infodetail-938737.html)l

### 八、关于webrtc编译
请参考[https://webrtc.org/native-code/android/](https://webrtc.org/native-code/android/)
编译好的android项目[https://github.com/shixueqiang/AppRTCMobile](https://github.com/shixueqiang/AppRTCMobile)