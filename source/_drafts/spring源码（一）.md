title: springFramework源码（一）
author: wen
tags: []
categories:
  - spring
date: 2020-08-08 14:45:00
---
# 为什么要用springFramework
* IOC容器

	* 不用spring之前，java对象实例可能多次通过new 创建或者单例模式或工厂方法反射获取，若java对象依赖关系复杂，实现起来就很复杂
    * 用了spring,统一管理对象的创建和生命周期的维护，自动维护对象的依赖关系--IOC(the Inversion of Control ) 也叫 依赖注入（dependency injection）
    	* 构造方法注入
        * setter方法属性
        * 成员属性注入
 * AOP       
  
        
# 如何理解“控制反转”好呢？
 
 理解好它的关键在于我们需要回答如下四个问题：

* 谁控制谁
* 控制什么
* 为何是反转
* 哪些方面反转了