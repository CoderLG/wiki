昨天简略的看了一下thymeleaf，今天抛砖引玉，主要给大家熟悉一下里面的标签，深层次的东西我们以后一同探索

### 一、概述

```

1.Thymeleaf 在有网络和无网络的环境下皆可运行，即它可以让美工在浏览器查看页面的静态效果，也可以让程序员在服务器查看带数据的动态页面效果。这是由于它支持 html 原型，然后在 html 标签里增加额外的属性来达到模板+数据的展示方式。浏览器解释 html 时会忽略未定义的标签属性，所以 thymeleaf 的模板可以静态地运行；当有数据返回到页面时，Thymeleaf 标签会动态地替换掉静态内容，使页面动态显示。

2.Thymeleaf 开箱即用的特性。它提供标准和spring标准两种方言，可以直接套用模板实现JSTL、 OGNL表达式效果，避免每天套模板、该jstl、改标签的困扰。同时开发人员也可以扩展和创建自定义的方言。

3. Thymeleaf 提供spring标准方言和一个与 SpringMVC 完美集成的可选模块，可以快速的实现表单绑定、属性编辑器、国际化等功能。

html模板

官方地址：https://www.thymeleaf.org/download.html
官方教程：http://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html#what-is-thymeleaf

		
环境 maven 阿里的镜像 端口8080 编辑器 
		
```

###  二、基础语法

```
创建HTML
	　由上文也可以知道需要在html中添加：
		<html xmlns:th="http://www.thymeleaf.org">
　　　　这样，下文才能正确使用th:*形式的标签！
　　　　
	jsp-struts
    	<%@taglib uri="/struts-tags" prefix="s" %>
	
获取变量值${...}
	通过${…}进行取值，这点和ONGL表达式语法一致！
	 <p th:text="'Hello！, ' + ${name} + '!'">3333</p>
	 
	jsp-struts
	<input type="hidden" name="tel" value='<s:property value="user.tel"/>' />

有了个浅显的认识，今天就带大家看看thymeleaf的标签

选择变量表达式*{...}
<div th:object="${session.user}">
    <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p> 
    <p>Nationality: <span th:text={nationality}">Saturn</span>.</p>
</div> 
等价于
<div>
    <p>Name: <span th:text="${session.user.firstName}">Sebastian</span>.</p> 
    <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p> 
    <p>Nationality: <span th:text="${session.user.nationality}">Saturn</span>.</p>
</div>

```

