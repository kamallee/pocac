title: mybatis学习笔记
date: 2016-07-04 18:16:45
tags: mybatis
categories: mybatis
---
<h2>什么是mybatis</h2>
MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以对配置和原生Map使用简单的 XML 或注解，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。
<!--more-->
<h2>Mybatis SQL注解方式</h2>


报错：
org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type [com.crazyfish.kamal.dao.ICategoryDAO] found for dependency: expected at least 1 bean which qualifies as autowire candidate for this dependency. Dependency annotations: {@javax.annotation.Resource(shareable=true, mappedName=, description=, name=, type=class java.lang.Object, lookup=, authenticationType=CONTAINER), @org.springframework.beans.factory.annotation.Qualifier(value=categoryDAO)}
DAO层使用mybatis注解方式，接口无法创建实例，所以@Resource注入的时候无法注入，导致失败，该方法会扫描该文件下的DAO接口，生成实例。
在spring配置文件中增加：<mybatis:scan base-package="com.crazyfish.kamal.dao"/>

报错：
org.springframework.beans.factory.xml.XmlBeanDefinitionStoreException: Line 40 in XML document from class path resource [conf/applicationContext.xml] is invalid; nested exception is org.xml.sax.SAXParseException; lineNumber: 40; columnNumber: 59; cvc-complex-type.2.4.c: 通配符的匹配很全面, 但无法找到元素 'mybatis:scan' 的声明。
只增加了这一行：
xmlns:mybatis="http://mybatis.org/schema/mybatis-spring （声明xml文件默认的命名空间，表示未使用其他命名空间的所有标签的默认命名空间，告知XML解析器根据某个schema来验证此文档）
没有添加schemaLocation:
http://mybatis.org/schema/mybatis-spring
http://mybatis.org/schema/mybatis-spring.xsd  描述xml文件的文档结构

<h2>参考文献</h2>
1.[mybatis官网](http://www.mybatis.org/mybatis-3/zh/)