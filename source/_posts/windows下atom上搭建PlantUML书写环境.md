title: 在windows下atom上搭建PlantUML书写环境
author: wen
tags:
  - plantuml
  - atom
categories:
  - atom
date: 2018-10-14 15:36:00
---
PlantUML是一款使用纯文本绘制UML图的开源软件。它的优点是能够帮助作者把精力集中到内容的书写上而不是格式的调整。本文介绍了一种在windows环境下atom编辑器上搭建PlantUML书写环境的方法。
# 工具准备
* atom
* graphviz
[Graphviz](http://www.graphviz.org/)是一款开源图形可视化软件。Atom的PlantUML插件使用graphviz将PlantUML语言绘制成相应的图形。
# 插件安装
github上已经有开发者为atom开发了PlantUML相关的插件。这里推荐以下两个插件
* language-plantuml
* plantuml-viewer
其中language-plantuml提供了PlantUML语法高亮支持，plantuml-viewer负责把PlantUML语句转化为对应的UML图。
```
有很多博客使用plantuml-preview。本人试用了这两款插件，发现都能很好的支持PlantUML。
但是plantuml-viewer更有优势：plantuml-viewer能够实时显示PlantUML语言描述的图形，
并且可以通过鼠标滚轮流畅的调节图形大小。此外，plantuml-preview需要显式的配置java和plantuml.jar，
而plantuml-viewer不需要，因此我推荐plantuml-viewer
```
安装方法很简单：atom下 File -> Settings -> Install 
![logo](windows下atom上搭建PlantUML书写环境/plantuml_plugin.png)

# 配置插件
plantuml-viewer按如下配置 
![logo](windows下atom上搭建PlantUML书写环境/plantuml_viewer_setting.png)
* Charset配置成utf-8在生成图形的时候中文不会乱码
* graphviz从官网下载后安装到自定义位置，在plant-viewer中指定dot.exe路径

# 验证
用PlantUML生成工厂模式UML
```
@startuml
skinparam classAttributeIconSize 0
class Product
class ConcreteProduct
class Creator {
  + {abstract} FactoryMethod()
}
class ConcreteCreator {
  + FactoryMethod()
}
Product <|-- ConcreteProduct
Creator <|-- ConcreteCreator
ConcreteProduct <.l. ConcreteCreator
note left of Product: 定义工厂方法所创建的对象的接口
note right of Creator: 声明工厂方法，该方法返回一个Product类型的对象
note left of ConcreteProduct: 具体产品，实现了Product的接口
note right of ConcreteCreator: 重定义工厂方法以返回一个ConcreteProduct实例
@enduml
```
效果图
![logo](windows下atom上搭建PlantUML书写环境/3.png)

[plantuml语法](http://plantuml.com/)
