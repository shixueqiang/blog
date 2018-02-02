---
title: android将so打到jar包中并运行
id: 74
categories:
  - 未分类
date: 2016-09-13 14:57:07
tags:
---

加载so有两种方法

System.load() 和System.loadLibrary(); 前者需传入库文件的绝对路径，后者只需传入库文件名。

首先我的jar包目录如下：

![](http://img.blog.csdn.net/20160704165107689?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

Loader是加载类：
<pre class="html"> static {
        try {
            InputStream is = null;
            if(isCPUInfo64()) {
                is = Loader.class.getResource("arm64/libhellojni.so").openStream();
            }else {
                is = Loader.class.getResource("arm32/libhellojni.so").openStream();
            }
            File tempFile = File.createTempFile("hellojni", ".so");
            FileOutputStream fos = new FileOutputStream(tempFile);
            int i;
            byte[] buf = new byte[1024];
            while ((i = is.read(buf)) != -1) {
                fos.write(buf, 0, i);
            }
            is.close();
            fos.close();
            System.load(tempFile.getAbsolutePath());
            tempFile.deleteOnExit();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }</pre>
&nbsp;
<pre class="html">private static boolean isCPUInfo64() {
        File cpuInfo = new File("/proc/cpuinfo");
        if (cpuInfo != null &amp;&amp; cpuInfo.exists()) {
            InputStream inputStream = null;
            BufferedReader bufferedReader = null;
            try {
                inputStream = new FileInputStream(cpuInfo);
                bufferedReader = new BufferedReader(new InputStreamReader(inputStream), 512);
                String line = bufferedReader.readLine();
                if (line != null &amp;&amp; line.length() &gt; 0 &amp;&amp; line.toLowerCase(Locale.US).contains("arch64")) {
                    return true;
                } else {
                    return false;
                }
            } catch (Throwable t) {
                Log.d("isCPUInfo64", " error = " + t.toString());
            } finally {
                try {
                    if (bufferedReader != null) {
                        bufferedReader.close();
                    }
                    if (inputStream != null) {
                        inputStream.close();
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
        return false;
    }</pre>
&nbsp;