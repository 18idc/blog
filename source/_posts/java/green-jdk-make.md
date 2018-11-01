---
title: 绿色版本JDK制作方法
tags: java
categories: 笔记
date: 2018-7-26 22:08:59
copyright: ture
---
# 1. 下载官网EXE版本的jdk安装包
# 2. 提取JDK文件
## 以下为JDK1.8的提取方式，不同版本路径有所不同。
### 1. 使用7-ZIP压缩工具打开EXE文件
### 2. 找到JDK文件的目录

<!-- more -->

#### 1. JDK1.8 32位的目录为：
```xml
\jdk-8u161-windows-i586.exe\tools.zip\
```
#### 2. JDK1.8 64位的目录为：
```xml
\jdk-8u161-windows-x64.exe\.rsrc\1033\JAVA_CAB10\111\tools.zip\
```

熟悉JDK程序目录的朋友，应该可以认出，这个目录下的文件就是JDK的文件了
### 3. 把上面目录的文件解压出来
### 4. 解压.pack文件
以管理员的身份运行CMD窗口，打开第3步提取文件的目录，输入如下命令
```batch
for /r %x in (*.pack) do .\bin\unpack200 -r "%x" "%~dx%~px%~nx.jar"
```
上面命令的作用是，使用unpack200这个程序，把*.pack的文件解压为.jar的文件。