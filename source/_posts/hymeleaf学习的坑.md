title: thymeleaf学习的坑
author: wen
tags:
  - thymeleaf
categories: []
date: 2018-05-08 14:30:00
---
# 引入css和js文件
```
<link th:href="@{/resources/admin/css/common.css}" rel="stylesheet" type="text/css" />
<script  type="text/javascript" th:src="@{/resources/admin/js/jquery.js}"></script>
```

# 网页标题(多语言)
使用#{key}
```
  <title th:text="#{system.name}"></title>
# thymeleaf结合js
<script  th:inline="javascript">
       //controller 使用Model设置属性
         var error=[[${error}]];
       //多语言
       var usernameRequired=[[#{admin.login.usernameRequired}]]
    </script>
    ```
# thymeleaf结合js 冲突
```
//使用<![CDATA[ ]]>
<script th:inline="javascript">
//<![CDATA[
  js code
	// ]]>
</script>
```
# th:include
compontent/commonJs.html
```
<div th:fragment="commonJs">
<script th:inline="javascript">
js代码（国际化语言）
</script>
</div>
```

引用
```
 <div th:include="compontent/commonJs::commonJs"></div>
 ```
# th:href
```
<a th:href="@{/admin/publicInfo/info(id=${publicInfo.id})}" target="iframe" th:text="#{admin.menu.publicManage.name}"></a>
```
# th:each
```
<tr th:each="admin:${page.content}">
			  <td th:text="${admin.username}"></td>
	      </tr>```
# 格式化日期
```
<td th:text="${#calendars.format(admin.createDate,'yyyy年MM月dd日')}"></td>
```
# th:if
•	单个条件
```
<td th:if="${admin.username}=='admin'"><input type="checkbox" name="ids" th:value="${admin.id}" disabled="disabled"/></td>
```
•	并且条件
```
<td th:if="${session.username}!='admin' and ${admin.username}==${session.username}">
<a th:href="@{initEdit(id=${admin.id})}" th:text="#{admin.menu.admin.edit}">
</a>
</td>
```

