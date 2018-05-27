---
title: QT解压缩zlib和quazip的使用
id: 68
categories:
  - 未分类
date: 2016-09-13 14:55:30
tags:
---

首先我的开发环境是windows10，qtvs2015，vs2015。

一、zlib的编译

1、官网下载最新的zlib源码，最新的是1.2.8

2、用vs自带的命令行工具（编译32位的dll选x86，64位的选x64）<span id="Label3">进入到zlib根目录，执行 nmake -f win32/Makefile.msc ,在根目录下生成：zlib.lib(静态库)  zdll.lib(动态库的导入库)  zlib1.dll(动态库) 。必要的头文件有zlib.h和zconf.h</span>。

二、quazip的编译

1、quazip是zlib的封装库，下载源码后用QT打开，编译需要依赖zlib库，右键项目添加库选外部库就好。我添加后在.pro文件末尾会生成下面配置,注意编译64位的quazip，使用的zlib也必须是64位的
<pre class="html">win32:CONFIG(release, debug|release): LIBS += -L$$PWD/../zlib64/ -lzdll
&nbsp;
else:win32:CONFIG(debug, debug|release): LIBS += -L$$PWD/../zlib64/ -lzdll
&nbsp;
else:unix: LIBS += -L$$PWD/../zlib64/ -lzdll
&nbsp;
INCLUDEPATH += $$PWD/../zlib64
&nbsp;
DEPENDPATH += $$PWD/../zlib64</pre>
&nbsp;

INCLUDEPATH是头文件路径

2、右键项目构建会生成quazip.lib和quazip.dll，注意zlib和quazip的默认字符集都是ANSI，在调用的文件的字符集也必须是ANSI，否则编译不过,使用utf-8的话，都必须是utf-8.

三、使用

1、使用时同样需要添加外部库zlib和quazip，字符集需要和dll一样，下面是一段解压缩代码
<pre class="html">bool extract(const QString&amp; in_file_path, const QString&amp; out_file_path)
{
    &nbsp;
    QuaZip archive(in_file_path);
    if (!archive.open(QuaZip::mdUnzip))
        return false;
    &nbsp;
    QString path = out_file_path;
    if (!path.endsWith("/") &amp;&amp; !out_file_path.endsWith("\\"))
        path += "/";
    &nbsp;
    QDir dir(out_file_path);
    if (!dir.exists())
        dir.mkpath(out_file_path);
    &nbsp;
    for( bool f = archive.goToFirstFile(); f; f = archive.goToNextFile() )
    {
        QString filePath = archive.getCurrentFileName();
        QuaZipFile zFile(archive.getZipName(), filePath);
        zFile.open(QIODevice::ReadOnly );
        QByteArray ba = zFile.readAll();
        zFile.close();
        &nbsp;
        if (filePath.endsWith("/"))
        {
            dir.mkpath(filePath);
        }
        else
        {
            QFile dstFile(path + filePath);
            if (!dstFile.open(QIODevice::WriteOnly))
                return false;
            dstFile.write(ba);
            dstFile.close();
        }
    }
    return true;
}</pre>
2、构建运行，新版QT运行需要拷贝qtvs2015下的plugins下的platforms文件夹到debug或release里边运行，还需要zlib1.dll和quazip.dll放到生成的exe同级目录下。运行没反应的话检查防火墙和杀毒。