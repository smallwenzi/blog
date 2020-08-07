title: tomcat 三种部署方式
author: wen
tags:
  - tomcat
categories: []
date: 2020-01-03 10:26:00
---


# 一Tomcat自动部署：
只要将一个Web应用的WebContent级的AppName直接扔进%Tomcat_Home%\webapps文件夹下，系统会把该web应用直接部署到Tomcat中
因为:

# 二增加自定义的Web部署文件
我们需要在%Tomcat_Home%\conf路径下新建一个文件夹catalina——再在其中新建一个localhost文件夹——最后再新建一个XML文件，即增加两层目录并新增XML文件：%Tomcat_Home%\conf\Catalina\localhost\web应用配置文件.xml ，该文件就是部署Web应用的配置文件。例如，我们新建一个%Tomcat_Home%\conf\Catalina\localhost\XXX.xml，  该文件的内容如下：
```
<Context path="/XXX" reloadable="true" docBase="D:\workspace\WebApp\AppName" workDir="D:\workspace\WebApp\work"/>  
```

# 手动修改%Tomcat_Home%\conf\server.xml文件来部署web应用

打开%Tomcat_Home%\conf\server.xml文件并在其中增加以下元素：

```

<Context docBase="D:\workspace\WebApp\AppName" path="/XXX" debug="0" reloadable="false" /> 
```

总结：第一种和第二种：
tomcat服务器在运行状态下会监视在WEB-INF/classes和WEB-INF/lib目录下class文件的改动，如果监测到有class文件被更新的，服务器会自动重新加载Web应用
