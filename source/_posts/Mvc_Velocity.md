---
title: spring mvc中应用velocity
date: 2018-10-14 14:31:08
tags:
---

### Velocity

Velocity是一款基于java的模板引擎。相比于jsp而言，它被使用的人可能并不多，但的确也是一款出色的模板引擎。jsp中允许出现java代码，而Velocity不允许，也是为了方便维护模板，严格遵从MVC设计原则。模板只负责页面的规划，渲染页面。当然Velocity的定位不仅仅在web页面的开发上使得前后端开发者可以更明确的职责分离，它还可以应用在其它涉及模板应用的领域，比如代码生成，SQL生成等等。freemark也是类似于velocity这样一款模板引擎。

SpringMvc集成Velocity
在mvc配置文件中配置velocity引擎，及viewResolver

```xml
<bean id="velocityConfig"
	class="org.springframework.web.servlet.view.velocity.VelocityConfigurer">
	<property name="resourceLoaderPath" value="/WEB-INF/views/" />
	<property name="configLocation" value="classpath:properties/velocity.properties" />
</bean>
<bean id="viewResolver"
	class="org.springframework.web.servlet.view.velocity.VelocityLayoutViewResolver">
	<property name="suffix" value=".html" />
	<property name="cache" value="false" />
	<property name="contentType" value="text/html;charset=utf-8" />
	<property name="dateToolAttribute" value="date" /><!--日期函数名称-->
	<property name="numberToolAttribute" value="number" /><!--数字函数名称-->
	<property name="layoutUrl" value="layout/default.vm"/>
	<property name="toolboxConfigLocation" value="/WEB-INF/toolbox.xml" />

</bean>
```

velocity.properties

```properties
input.encoding=utf-8
output.encoding=utf-8
velocimacro.library=macro/macro.vm
velocimacro.library.autoreload=true
runtime.log.logsystem.class = org.apache.velocity.runtime.log.Log4JLogChute
userdirective=com.baomidou.framework.velocity.directive.BlockDirective,com.baomidou.framework.velocity.directive.OverrideDirective,com.baomidou.framework.velocity.directive.ExtendsDirective
#file.resource.loader.cache = true
```

velocimacro.library配置用户自定义的宏
userdirective配置用户自定义的指令
resourceLoaderPath配置velocity的模板文件位置

### Velocity特色

velocity的核心是模板引擎、数据上下文，灵活的表达式、控制渲染的指令、宏
通过表达式我们可以取用context数据
通过基本指令set、if、parset、include、foreach等等，我们可以定义变量、模块化模板、动态渲染页面
通过使用自定义指令，我们可以实现一些继承特性
通过使用宏或者spring内置宏，我们可以灵活复用渲染逻辑、结合mvc对接web Model

***关于Velocity，我们需要明白它在web领域主要是应用在后端视图解析渲染，类似于jsp。在模板中的js，css，html它并不会干涉，而涉及vtl的表达式，它会进行渲染。当服务器把VTL渲染后HTML推送到前端浏览器时，这时js，css，html才会起作用，浏览器会运行js，渲染界面，呈现给用户UI。***