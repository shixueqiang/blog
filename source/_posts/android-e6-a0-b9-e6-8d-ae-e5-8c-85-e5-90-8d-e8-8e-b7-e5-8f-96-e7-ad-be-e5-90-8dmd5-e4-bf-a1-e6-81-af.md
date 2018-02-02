---
title: android根据包名获取签名MD5信息
id: 79
categories:
  - 未分类
date: 2017-05-11 22:14:18
tags:
---

1.  <span class="keyword">private</span> Signature[] getRawSignature(Context paramContext, String paramString) {
2.  <span class="keyword">if</span> ((paramString == <span class="keyword">null</span>) || (paramString.length() == <span class="number">0</span>)) {
3.  errout(<span class="string">"getSignature, packageName is null"</span>);
4.  <span class="keyword">return</span> <span class="keyword">null</span>;
5.  }
6.  PackageManager localPackageManager = paramContext.getPackageManager();
7.  PackageInfo localPackageInfo;
8.  <span class="keyword">try</span> {
9.  localPackageInfo = localPackageManager.getPackageInfo(paramString, PackageManager.GET_SIGNATURES);
10.  <span class="keyword">if</span> (localPackageInfo == <span class="keyword">null</span>) {
11.  errout(<span class="string">"info is null, packageName = "</span> + paramString);
12.  <span class="keyword">return</span> <span class="keyword">null</span>;
13.  }
14.  } <span class="keyword">catch</span> (PackageManager.NameNotFoundException localNameNotFoundException) {
15.  errout(<span class="string">"NameNotFoundException"</span>);
16.  <span class="keyword">return</span> <span class="keyword">null</span>;
17.  }
18.  <span class="keyword">return</span> localPackageInfo.signatures;
19.  }

1.  <span class="keyword">private</span> Signature[] getRawSignature(Context paramContext, String paramString) {
2.  <span class="keyword">if</span> ((paramString == <span class="keyword">null</span>) || (paramString.length() == <span class="number">0</span>)) {
3.  errout(<span class="string">"getSignature, packageName is null"</span>);
4.  <span class="keyword">return</span> <span class="keyword">null</span>;
5.  }
6.  PackageManager localPackageManager = paramContext.getPackageManager();
7.  PackageInfo localPackageInfo;
8.  <span class="keyword">try</span> {
9.  localPackageInfo = localPackageManager.getPackageInfo(paramString, PackageManager.GET_SIGNATURES);
10.  <span class="keyword">if</span> (localPackageInfo == <span class="keyword">null</span>) {
11.  errout(<span class="string">"info is null, packageName = "</span> + paramString);
12.  <span class="keyword">return</span> <span class="keyword">null</span>;
13.  }
14.  } <span class="keyword">catch</span> (PackageManager.NameNotFoundException localNameNotFoundException) {
15.  errout(<span class="string">"NameNotFoundException"</span>);
16.  <span class="keyword">return</span> <span class="keyword">null</span>;
17.  }
18.  <span class="keyword">return</span> localPackageInfo.signatures;
19.  }
源码地址：[GenSignature](https://github.com/shixueqiang/GenSignature)