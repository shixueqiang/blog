---
title: android根据包名获取签名MD5信息
id: 79
categories:
  - 未分类
date: 2017-05-11 22:14:18
tags:
---
<pre>
private Signature[] getRawSignature(Context paramContext, String paramString) {
    if ((paramString == null) || (paramString.length() == 0)) {
      errout("getSignature, packageName is null");
      return null;
    }
    PackageManager localPackageManager = paramContext.getPackageManager();
    PackageInfo localPackageInfo;
    try {
      localPackageInfo=localPackageManager.getPackageInfo(paramString,PackageManager.GET_SIGNATURES);
      if (localPackageInfo == null) {
        errout("info is null, packageName = " + paramString);
        return null;
      }
    } catch (PackageManager.NameNotFoundException localNameNotFoundException) {
      errout("NameNotFoundException");
      return null;
    }
    return localPackageInfo.signatures;
}
private Signature[] getRawSignature(Context paramContext, String paramString) {
    if ((paramString == null) || (paramString.length() == 0)) {
      errout("getSignature, packageName is null");
      return null;
    }
    PackageManager localPackageManager = paramContext.getPackageManager();
    PackageInfo localPackageInfo;
    try {
      localPackageInfo=localPackageManager.getPackageInfo(paramString,PackageManager.GET_SIGNATURES);
      if (localPackageInfo == null) {
        errout("info is null, packageName = " + paramString);
        return null;
      }
    } catch (PackageManager.NameNotFoundException localNameNotFoundException) {
        errout("NameNotFoundException");
        return null;
    }
    return localPackageInfo.signatures;
}</pre>
源码地址：[GenSignature](https://github.com/shixueqiang/GenSignature)