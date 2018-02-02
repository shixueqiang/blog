---
title: >-
  gyp ERR! stack You can pass the --python switch to point to Python >= v2.5.0 &
  < 3.0.0
id: 119
categories:
  - 未分类
date: 2017-09-27 00:02:58
tags:
---

npm install 的时候报了如下错误：

gyp ERR! configure error
gyp ERR! stack Error: Python executable "C:\Users\Administrator\AppData\Local\Programs\Python
\Python36\python.EXE" is v3.6.2, which is not supported by gyp.
gyp ERR! stack You can pass the --python switch to point to Python >= v2.5.0 & < 3.0.0.
gyp ERR! stack     at failPythonVersion (D:\nodejs\node_modules\npm\node_modules\node-gyp\lib
\configure.js:454:14)
gyp ERR! stack     at D:\nodejs\node_modules\npm\node_modules\node-gyp\lib\configure.js:443:9
gyp ERR! stack     at ChildProcess.exithandler (child_process.js:189:7)
gyp ERR! stack     at emitTwo (events.js:106:13)
gyp ERR! stack     at ChildProcess.emit (events.js:191:7)
gyp ERR! stack     at maybeClose (internal/child_process.js:891:16)
gyp ERR! stack     at Socket.<anonymous> (internal/child_process.js:342:11)
gyp ERR! stack     at emitOne (events.js:96:13)
gyp ERR! stack     at Socket.emit (events.js:188:7)
gyp ERR! stack     at Pipe._handle.close [as _onclose] (net.js:497:12)
gyp ERR! System Windows_NT 10.0.14393
gyp ERR! command "D:\\nodejs\\node.exe" "D:\\nodejs\\node_modules\\npm\\node_modules\\node-gy

解决方法：
安装不同版本python，然后使用时 npm install --(两个-)python=python2.7