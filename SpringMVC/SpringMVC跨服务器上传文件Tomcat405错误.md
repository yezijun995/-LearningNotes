---
title: SpringMVC跨服务器上传文件Tomcat405错误
date: 2020-04-14 13:28:13
tags:
---
## 如何在SpringMVC跨服务器上传文件405方法响应状态，请检查Tomcat是否设置读写
### 设置Tomcat读写
在web.xml文件中，<servlet>下添加:

		<init-param>
	        <param-name>readonly</param-name>
	        <param-value>false</param-value>
		</init-param>