title: spring源码编译
author: wen
tags:
  - 源码
categories:
  - spring
date: 2020-08-07 16:02:00
---
# 安装环境配套
Gradle 5.6.2 +java version "1.8.0_251"+idea+spring framework5.1.x
* Spring framework下载：https://github.com/spring-projects/spring-framework/tree/5.1.x
* Gradle 6.5.1下载:https://gradle.org/next-steps/?version=5.6.2&format=bin

# 配置gradle环境
  配置gradle环境变量 新建GRADLE_HOME，path中增加%GRADLE_HOME%/bin;即可
  ![gradle](spring源码编译/1.png)
  ![gradle](spring源码编译/2.png)
  配置完成以后win+r，输入cmd进入dos界面，键入命令gradle -v，若如下图正确输出版本信息即可
    ![gradle](spring源码编译/3.png)
  配置gradle默认的本地仓库，gradle和maven类似，工作时也需要一个本地仓库，管理工程jar包，可做如下配置，新增GRADLE_USER_HOME
     ![gradle](spring源码编译/4.png)
 # Git 下载代码
Gi地址：https://github.com/spring-projects/spring-framework.git
 ![gradle](spring源码编译/5.png)
 下载完成切换分支5.1.x
  ![gradle](spring源码编译/6.png)
  在import-into-idea.md 你编译spring work源码之前你需要spring-core和spring-oxm
  ![gradle](spring源码编译/7.png)
  
  # 注意
  注意几点：
 * a，因为其他项目需要依赖spring-core和spring-oxm，所以我们导入后需要先编译这两个包 （在cmd 命令下执行gradle build -x test ）,需要
  等候一定时间，保证网络稳定
 ![gradle](spring源码编译/8.png)
  ![gradle](spring源码编译/9.png)

成功截图： 显示Build successful
 ![gradle](spring源码编译/10.png)

*  b，spring-aspects需要依赖三方jar（精通aop的应该知道spring-aop和aspectj的关系，在使用注解方式实现spring-aop的时候，需要导入aspectj的jar，有兴趣的可以去查阅下官方文档aop相关文档），这里编译可能会报错，不影响我们编译使用，我们先不处理
  
* c，使用工具编译时，可能会出现内存溢出情况，这里我们编译时需要增加相关参数
 ```
         -XX:MaxPermSize=2048m -Xmx2048m -XX:MaxHeapSize=2048m
         ```
* d) 以上工作完成之后，我们可以开始导入编辑编译配置了，如下图（需要注意的地方已在图中标注）

# idea使用导入springframework项目
