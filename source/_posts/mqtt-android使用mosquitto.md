---
title: mqtt-android使用mosquitto
date: 2018-11-09 17:33:05
tags:
---
本文介绍如何在android上使用mosquitto实现mqtt协议。Broker和Client一套代码方便扩展
### 一、Broker
1、下载代码
`git clone https://github.com/eclipse/mosquitto.git`
2、使用cmake编译
```
make dir build
cd build
cmake ..
make
```
mosquitto为代理服务器，mosquitto_pub为发布客户端，mosquitto_sub为订阅客户端，使用方式可参考[https://mosquitto.org/man/](https://mosquitto.org/man/)
### 二、Android
1、由于依赖openssl库，先交叉编译openssl
```
git clone https://github.com/openssl/openssl.git
ANDROID_NDK=/some/where/android-ndk-r14b
PATH=$ANDROID_NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64/bin:$PATH
./Configure android-arm -D__ANDROID_API__=19
make
```
2、修改CMakeLists.txt配置文件交叉编译android端lib库
```
if(${TARGET_ANDROID} STREQUAL ON)
	add_definitions("-DTARGET_ANDROID")
	message(STATUS "target system android")
	# Android 5.0 以上需要在此处设置 PIE
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIE")
	set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fPIE -pie")
	set(CMAKE_ANDROID_NDK /home/shixq/android/android-ndk-r14b)
	set(CMAKE_SYSTEM_NAME Android)
	if(${ANDROID_ABI} STREQUAL arm)
		set(CMAKE_SYSTEM_VERSION 19)
		set(CMAKE_ANDROID_ARCH_ABI armeabi-v7a)
	elseif(${ANDROID_ABI} STREQUAL arm64)
		set(CMAKE_SYSTEM_VERSION 21)
		set(CMAKE_ANDROID_ARCH_ABI arm64-v8a)
	endif(${ANDROID_ABI} STREQUAL arm)
endif(${TARGET_ANDROID} STREQUAL ON)

project(mosquitto)
```
```
if (${TARGET_ANDROID} STREQUAL ON)
    set(OPENSSL_INCLUDE_DIR ${mosquitto_SOURCE_DIR}/android/include)
    if ("${ANDROID_ABI}" STREQUAL arm)
        message(STATUS "ANDROID_ABI IS ARM")
        set(OPENSSL_LIBRARIES ${mosquitto_SOURCE_DIR}/android/lib/arm/libssl.a ${mosquitto_SOURCE_DIR}/android/lib/arm/libcrypto.a)
    elseif("${ANDROID_ABI}" STREQUAL arm64)
        message(STATUS "ANDROID_ABI IS ARM64")
        set(OPENSSL_LIBRARIES ${mosquitto_SOURCE_DIR}/android/lib/arm64/libssl.a ${mosquitto_SOURCE_DIR}/android/lib/arm64/libcrypto.a)
    endif("${ANDROID_ABI}" STREQUAL arm)
else()
	find_package(OpenSSL REQUIRED)
```
完整的配置文件地址[https://github.com/shixueqiang/mqtt-android/blob/master/CMakeLists.txt](https://github.com/shixueqiang/mqtt-android/blob/master/CMakeLists.txt)

3、添加android代码
主要是一些jni代码，以供java层调用
4、编译
`cmake -DTARGET_ANDROID=ON -DANDROID_ABI=arm ..`
### 三、完整项目地址
native代码[https://github.com/shixueqiang/mqtt-android](https://github.com/shixueqiang/mqtt-android)
android studio[https://github.com/shixueqiang/androidmqtt](https://github.com/shixueqiang/androidmqtt)