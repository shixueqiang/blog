---
title: android应用proguard混淆打包
id: 56
categories:
  - 未分类
date: 2016-09-13 14:50:37
tags:
---

<span style="font-size: large;">android应用发布的时候一般需要对代码混淆打包，这里贴上我自己的proguard配置文件：</span>
<pre class="java">-libraryjars libs/gson-2.2.4.jar   #项目里用到的第三方jar包
-libraryjars libs/httpmime-4.1.3.jar
-libraryjars libs/ImageLoader.jar
-libraryjars libs/libammsdk.jar
-libraryjars libs/mta-sdk-1.6.2.jar
-libraryjars libs/nineoldandroids-2.4.0.jar
-libraryjars libs/open_sdk_r4889.jar
-libraryjars libs/rebound-core.jar
-libraryjars libs/universal-image-loader-1.9.3.jar
-libraryjars libs/volley.jar
-libraryjars libs/weibosdkcore.jar

-optimizationpasses 5   # 指定代码的压缩级别
-dontusemixedcaseclassnames  # 是否使用大小写混合  
-ignorewarning        # 忽略警告，避免打包时某些警告出现
-dontskipnonpubliclibraryclasses  #指定不去忽略非公共的库类
-dontoptimize  #不优化输入的类文件
-verbose   # 混淆时是否记录日志 

-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*   #混淆时采用的算法

-keep public class * extends android.app.Activity   # 保持哪些类不被混淆
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgentHelper
-keep public class * extends android.preference.Preference
-keep public class * extends android.support.v4.**
-keep public class com.android.vending.licensing.ILicensingService
-keep class com.sina.**{*;}
-keep class com.google.gson.stream.** { *; }
-keep class com.csm.model.** { *; }

-dontwarn android.support.v4.**   # 缺省proguard 会检查每一个引用是否正确,但是第三方库里面往往有些不会用到的类,没有正确引用。如果不配置的话,系统就会报错
-dontwarn android.annotation
-dontwarn org.apache.commons.net.**
-dontwarn android.webkit.WebView
-dontwarn com.umeng.**
-dontwarn com.tencent.weibo.sdk.**

-keepclasseswithmembernames class * { #保持本地方法不被混淆
    native &lt;methods&gt;;
}

-keepclasseswithmembers class * {  #保护指定的类和类的成员，但条件是所有指定的类和类成员是要存在。
    public &lt;init&gt;(android.content.Context, android.util.AttributeSet);
}

-keepclasseswithmembers class * {
    public &lt;init&gt;(android.content.Context, android.util.AttributeSet, int);
}

-keepclassmembers class * extends android.app.Activity { #保护指定类的成员
   public void *(android.view.View);
}

-keepclassmembers enum * {   #保持枚举enum类不被混淆
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

-keep class * implements android.os.Parcelable {  #保护继承Parcelable的类
  public static final android.os.Parcelable$Creator *;
}

-keepattributes *Annotation*   #保护注解&lt;/span&gt;</pre>
<span style="font-size: large;">下边附一篇比较详细的讲解：</span>
<pre class="java">&lt;span style="font-size:18px;"&gt;-include {filename}    从给定的文件中读取配置参数 
-basedirectory {directoryname}    指定基础目录为以后相对的档案名称 
-injars {class_path}    指定要处理的应用程序jar,war,ear和目录 
-outjars {class_path}    指定处理完后要输出的jar,war,ear和目录的名称 
-libraryjars {classpath}    指定要处理的应用程序jar,war,ear和目录所需要的程序库文件 
-dontskipnonpubliclibraryclasses    指定不去忽略非公共的库类。 
-dontskipnonpubliclibraryclassmembers    指定不去忽略包可见的库类的成员。

保留选项 
-keep {Modifier} {class_specification}    保护指定的类文件和类的成员 
-keepclassmembers {modifier} {class_specification}    保护指定类的成员，如果此类受到保护他们会保护的更好
-keepclasseswithmembers {class_specification}    保护指定的类和类的成员，但条件是所有指定的类和类成员是要存在。 
-keepnames {class_specification}    保护指定的类和类的成员的名称（如果他们不会压缩步骤中删除） 
-keepclassmembernames {class_specification}    保护指定的类的成员的名称（如果他们不会压缩步骤中删除） 
-keepclasseswithmembernames {class_specification}    保护指定的类和类的成员的名称，如果所有指定的类成员出席（在压缩步骤之后） 
-printseeds {filename}    列出类和类的成员-keep选项的清单，标准输出到给定的文件 

压缩 
-dontshrink    不压缩输入的类文件 
-printusage {filename} 
-whyareyoukeeping {class_specification}     

优化 
-dontoptimize    不优化输入的类文件 
-assumenosideeffects {class_specification}    优化时假设指定的方法，没有任何副作用 
-allowaccessmodification    优化时允许访问并修改有修饰符的类和类的成员 

混淆 
-dontobfuscate    不混淆输入的类文件 
-printmapping {filename} 
-applymapping {filename}    重用映射增加混淆 
-obfuscationdictionary {filename}    使用给定文件中的关键字作为要混淆方法的名称 
-overloadaggressively    混淆时应用侵入式重载 
-useuniqueclassmembernames    确定统一的混淆类的成员名称来增加混淆 
-flattenpackagehierarchy {package_name}    重新包装所有重命名的包并放在给定的单一包中 
-repackageclass {package_name}    重新包装所有重命名的类文件中放在给定的单一包中 
-dontusemixedcaseclassnames    是否使用大小写混写
-keepattributes {attribute_name,...}    保护给定的可选属性，例如LineNumberTable, LocalVariableTable, SourceFile, Deprecated, Synthetic, Signature, and 

InnerClasses. 
-renamesourcefileattribute {string}    设置源文件中给定的字符串常量&lt;/span&gt;</pre>
&nbsp;