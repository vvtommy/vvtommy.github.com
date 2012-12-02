---
layout: post
title: "Linux下的source命令"
date: 2012-12-02 15:50
comments: true
categories: dev evernote搬家
---
用法

    source FileName

在**当前Shell**下执行文件,通常使用 `.` 代替。例如:`. ./process.sh`

`source`在当前shell环境下执行命令，而直接执行是脚本启动一个子shell来执行命令。这样如果把设置环境变量（或alias等等）的命令写进脚本中，就只会影响子shell,无法改变当前的`shell`,所以通过文件（命令列）设置环境变量时，要用`source`命令。