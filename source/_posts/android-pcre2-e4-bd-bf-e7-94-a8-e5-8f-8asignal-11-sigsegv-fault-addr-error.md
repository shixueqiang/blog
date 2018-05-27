---
title: android pcre2使用及signal 11 (SIGSEGV) fault addr error
id: 140
categories:
  - android
  - c++
date: 2018-03-11 11:42:14
tags:
---

pcre2源码[ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre2-10.31.zip](ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre2-10.31.zip)

我是直接用的aosp源码树下external/pcre模块的代码，然后先将pcre2编译成静态库，直接用ndk-build编译。

Android studio 新建支持c++的native工程，将pcrecpp代码导入到工程中，以下是配置文件：
<pre>externalNativeBuild {
    &nbsp;
    // For ndk-build, instead use ndkBuild {}
    cmake {
        &nbsp;
        // Passes optional arguments to CMake.
        arguments "-DANDROID_ARM_NEON=TRUE", "-DANDROID_TOOLCHAIN=clang", '-DANDROID_STL=gnustl_shared'
        &nbsp;
        // Sets optional flags for the C compiler.
        cFlags "-D_EXAMPLE_C_FLAG1", "-D_EXAMPLE_C_FLAG2"
        &nbsp;
        // Sets a flag to enable format macro constants for the C++ compiler.
        cppFlags "-D__STDC_FORMAT_MACROS"
        &nbsp;
        cppFlags "-std=c++11"
    }
}
ndk {
    // Specifies the ABI configurations of your native
    // libraries Gradle should build and package with your APK.
    abiFilters 'armeabi-v7a', 'arm64-v8a'
}
</pre>
需要注意的是gnustl需要使用动态库，我最初用的静态库在arm64位的手机上string assign就会报signal 11 (SIGSEGV) fault addr，原因可能是[https://stackoverflow.com/questions/12590581/crash-on-stl-string-assignment-ndk](https://stackoverflow.com/questions/12590581/crash-on-stl-string-assignment-ndk)

pcrecpp的使用：
<pre>JNIEXPORT jboolean Java_com_shixq_www_pcre2test_PhoneUtil_isPhoneMatch
        (JNIEnv *env, jclass thiz, jstring phone, jstring regex) {
    char* _phone = jstringToChar(env, phone);
    char* _regex = jstringToChar(env, regex);
    LOGE("phone is %s,regex is %s", _phone, _regex);
    pcrecpp::RE re(_regex);
    bool isMatch = re.FullMatch(_phone);
    return isMatch;
}
&nbsp;
JNIEXPORT jstring Java_com_shixq_www_pcre2test_PhoneUtil_getPhonePrefix
        (JNIEnv *env, jclass thiz, jstring phone, jstring regex) {
    char* _phone = jstringToChar(env, phone);
    char* _regex = jstringToChar(env, regex);
    pcrecpp::RE re(_regex);
    string prefix;
    string minPhone;
    re.FullMatch(_phone, &amp;prefix, &amp;minPhone);
    LOGE("phone prefix is %s", prefix.c_str());
    return env-&gt;NewStringUTF(prefix.c_str());
}
&nbsp;
JNIEXPORT jstring Java_com_shixq_www_pcre2test_PhoneUtil_getMinPhone
        (JNIEnv *env, jclass thiz, jstring phone, jstring regex) {
    char* _phone = jstringToChar(env, phone);
    char* _regex = jstringToChar(env, regex);
    pcrecpp::RE re(_regex);
    string prefix;
    string minPhone;
    re.FullMatch(_phone, &amp;prefix, &amp;minPhone);
    LOGE("phone un prefix is %s", minPhone.c_str());
    return env-&gt;NewStringUTF(minPhone.c_str());
}
</pre>
完整代码地址：[https://github.com/shixueqiang/pcre2test](https://github.com/shixueqiang/pcre2test)
&nbsp;

&nbsp;

&nbsp;