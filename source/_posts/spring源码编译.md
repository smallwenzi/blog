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
  等候一定时间(我这边花了1小时时间)，保证网络稳定
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
 ![gradle](spring源码编译/11.png)
 点击 import project
![gradle](spring源码编译/12.png)
使用gradle导入项目
![gradle](spring源码编译/13.png)
最后点击finish （需要从gradle仓库下载所需依赖包 这个过程可能比较久 决定因素是 网络环境及机器环境 ）


导入成功：
![gradle](spring源码编译/15.png)

# 常见问题：

* 导入项目的时候还需要下载gradle 软件
Setting--》gradle 选择gradle软件目录

![gradle](spring源码编译/16.png)

* 可能由于网络原因构建失败 在gradle窗口选择spring项目右击 菜单选择refreshGradle dependencies

![gradle](spring源码编译/17.png)

# 新建Model
![gradle](spring源码编译/18.png)
选择Gradle项目 next

![gradle](spring源码编译/19.png)
输入项目名称
![gradle](spring源码编译/20.png)
确认项目存储路径 最后点击finish
![gradle](spring源码编译/21.png)
新建完成：
![gradle](spring源码编译/22.png)
编辑build.gradle
compile(project(":spring-context"))
![gradle](spring源码编译/23.png)

新增AppConfig.java和MemberService
```
package com.yanwen.springTest;

import com.yanwen.springTest.service.MemberService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Configuration;

@ComponentScan("com.yanwen.springTest")
@Configuration
public class AppConfig {
   public static void main(String[] args){
      AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
      System.out.println(annotationConfigApplicationContext.getBean(MemberService.class));
    }
}

package com.yanwen.springTest.service;

import org.springframework.stereotype.Service;

@Service
public class MemberService {
}

```
运行Appconfig main方法

![gradle](spring源码编译/25.png)
