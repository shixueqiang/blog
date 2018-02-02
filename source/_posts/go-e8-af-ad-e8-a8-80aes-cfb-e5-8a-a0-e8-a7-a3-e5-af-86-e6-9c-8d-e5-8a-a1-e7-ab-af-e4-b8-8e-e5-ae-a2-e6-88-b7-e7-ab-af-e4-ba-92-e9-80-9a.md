---
title: go语言AES CFB加解密服务端与客户端互通
id: 91
categories:
  - android
  - go
date: 2017-08-04 00:09:20
tags:
---

<pre class="html">package utils

import (
	"bytes"
	"crypto/aes"
	"crypto/cipher"
	"encoding/base64"
	"fmt"
	"log"
)

var commonIV = []byte{0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08, 0x09, 0x0a, 0x0b, 0x0c, 0x0d, 0x0e, 0x0f}

var key = "32位key"

func AesEncrypt(content string) string {
	// 创建加密算法aes
	c, err := aes.NewCipher([]byte(key))
	if err != nil {
		log.Fatalln("Error: NewCipher(%d bytes) = %s", len(key), err)
	}

	//加密字符串
	cfb := cipher.NewCFBEncrypter(c, commonIV)
	ciphertext := make([]byte, len(content))
	cfb.XORKeyStream(ciphertext, []byte(content))
	fmt.Printf("%s=&gt;%x\n", content, ciphertext)
	ciphertext = Base64Encode(ciphertext)
	return string(ciphertext)
}

func AesDecrypt(content string) string {
	text := make([]byte, len(content))
	var err error
	text, err = Base64Decode([]byte(content))
	if err != nil {
		log.Fatalln(err)
	}
	// 创建加密算法aes
	c, err := aes.NewCipher([]byte(key))
	if err != nil {
		log.Fatalln("Error: NewCipher(%d bytes) = %s", len(key), err)
	}
	// 解密字符串
	decryptText := make([]byte, len(text))
	cfbdec := cipher.NewCFBDecrypter(c, commonIV)
	cfbdec.XORKeyStream(decryptText, []byte(text))
	fmt.Printf("%x=&gt;%s\n", text, decryptText)
	return string(decryptText)
}</pre>
&nbsp;
<div><span style="font-size: 18px;">客户端解密时不同语言容易出现问题，一种简单的处理办法就是对go代码交叉编译出android端和ios端的lib库，我们可以使用go mobile，[https://github.com/golang/go/wiki/Mobile](https://github.com/golang/go/wiki/Mobile)</span></div>
<div><span style="font-size: 18px;">获取：</span></div>
<div><span style="font-size: 18px;">$ go get golang.org/x/mobile/cmd/gomobile</span></div>
<div><span style="font-size: 18px;">$ gomobile init # it might take a few minutes</span></div>
<div><span style="font-size: 18px;">编译：</span></div>
<div><span style="font-size: 18px;">$ gomobile bind -target=android golang.org/x/mobile/example/bind/hello</span></div>
<div><span style="font-size: 18px;">会生成一个aar</span></div>
&nbsp;