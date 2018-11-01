---
title: Mybatis运行常见错误汇总（持续更新）
tags: java
categories: 笔记
date: 2018-7-15 02:13:32
copyright: ture
---

# 1. 找不到类中的 get 属性

```java
org.mybatis.spring.MyBatisSystemException: nested exception is org.apache.ibatis.reflection.ReflectionException: There is no getter for property named 'userName' in 'class com.q18idc.Xxxx'
```

检查取值表达式中的属性名是否写错了，例如：{username,jdbcType=INTEGER}把 userName 写成了 username

# 2. BaseResultMap 重复了

因为 mybatis 的代码生成插件，xml 文件是追加，如果你执行了两次生成的话，表的映射 xml 里的代码会生成两遍，所以就会报错

```java
Error parsing Mapper XML. Cause: java.lang.IllegalArgumentException: Result Maps collection already contains value for com.q18idc.news.dao.SysLogMapper.BaseResultMap  
```

解决方法：检查对应的 xml 文件中是否有两个相同的 BaseResultMap 结果集

# 3. jdbcType 写错了

```java
Cause: org.apache.ibatis.builder.BuilderException: Error resolving JdbcType. Cause: java.lang.IllegalArgumentException: No enum constant org.apache.ibatis.type.JdbcType.xxxxx  
```

<!-- more -->

## 解决方法

1.  检查 resultMap 节点中的 jdbcType 属性是否写错了，例如：jdbcType="DECIMAL"

2.  检查取值表达式中的 jdbcType 属性是否写错了，例如：#{userid,jdbcType=DECIMAL}

# 4. 类名写错了

```java
Cause: org.apache.ibatis.type.TypeException: Could not resolve type alias 'com.q18idc.xxxx'.  Cause: java.lang.ClassNotFoundException: Cannot find class: com.q18idc.xxxx
```

## 解决方法

1.  检查配置文件中的 parameterType、或者 resultMap 节点中的 type 属性对应的类是否写错了。

# 5. 结果集 ID 写错了

```java
org.apache.ibatis.builder.IncompleteElementException: Could not find result map com.q18idc.news.dao.UserMapper.BaseResultMap2
```

## 解决方法

1.  检查 CRUD 对应 resultMap 属性名是否写错了

# 6. 找不到类中的 set 属性

```java
org.mybatis.spring.MyBatisSystemException: nested exception is org.apache.ibatis.reflection.ReflectionException: Could not set property 'userName' of 'class com.q18idc.Xxxx' with value '25' Cause: org.apache.ibatis.reflection.ReflectionException: There is no setter for property named 'userName' in 'class com.q18idc.Xxxx'
```

## 解决方法

1.  检查 resultMap 节点中的，id 或者 result 节点中的 property 属性名是否写错了

# 7. 创建 MYBATIS 配置文件实例出错

```java
Cause: org.xml.sax.SAXParseException; lineNumber: 3; columnNumber: 53; 文档根元素 "mapper" 必须匹配 DOCTYPE 根 "con"。
```

## 解决方法

1.  这个问题引起的原因是 mybatis 的 mapper 配置文件的 DOCTYPE 有误，正确的应该是

```xml
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
```

# 8. 解析 MYBATIS 配置文件出错

错误日志

```java
 [*.xml]: Invocation of init method failed; nested exception is org.springframework.core.NestedIOException: Failed to parse mapping resource: 'file [*.xml]'; nested exception is org.apache.ibatis.builder.BuilderException: Error parsing Mapper XML.
```
