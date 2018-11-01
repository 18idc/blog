---
title: Maven安装配置
tags: java
categories: 笔记
date: 2018-8-4 02:55:24
copyright: ture
---

# 下载Mmaven
[Maven官网](https://maven.apache.org/)

# 安装Maven
绿色无公害软件，找个地方解压即可。建议不要解压到C盘（系统盘），因为是绿色软件，所以即使重装了系统，只要配置一下环境变量就可以继续使用。

<!-- more -->

# 配置环境变量
打开 我的电脑->右击->属性->高级系统设置->高级->环境变量窗口 编辑path变量，在最后追加 ;E:\Maven\bin  
注：不同变量值之间使用;分隔 如果想修改MAVEN运行JVM的内容，可以新增环境变量MAVEN_OPTS,变量值为：-Xms256m -Xmx512m

# 检测Maven
检查MAVEN是否安装好之前，先把系统环境变量的窗口确认关闭了，再重新打开CMD窗口 注：Maven 3.3版本或更高版本 要求 JDK 1.7或更高版本 打开CMD窗口，输入 mvn -v 命令，如果能看到Maven版本及相关信息，说明已经安装好了

# 配置本地仓库路径
打开E:\Maven\conf\settings.xml配置文件
在<settings>节点中新增
```xml
<!-- 指向本地仓库路径 -->
<localRepository>E:\Maven\repository</localRepository>
```

# 配置伟大阿里的MAVEN镜像库
如果你想要使项目依赖库下载更快、更稳，那就需要在MAVEN的配置文件settings.xml中设置镜像属性 同样打开E:\Maven\conf\settings.xml配置文件 在mirrors节点新增如下内容
```xml
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>*</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```