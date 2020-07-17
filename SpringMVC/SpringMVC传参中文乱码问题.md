---
title: SpringMVC传参中文乱码问题
date: 2020-04-13 15:52:23
tags:
---
## 在发送中文表单的POST请求时会出现中文乱码，解决方法-------配置解决中文乱码的过滤器（Tomcat8版本之后解决了Get请求的中文乱码问题）
### 1.在web.xml中配置过滤器
	<!--配置解决中文乱码的过滤器-->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
		<!--SpringMVC的过滤器-->
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
			<!--初始化CharacterEncodingFilter中的属性-->
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
	<!--过滤器映射-->
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
	注:过滤器需在Servlet之前，要不标签会报错。
----
