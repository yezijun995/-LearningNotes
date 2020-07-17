# Spring Boot中使用Restful风格

在学习 Spring Boot 过程中，在实现 restful 的更新操作时，需要将表单数据以 PUT 方法提交。

按照视频中操作，直接在表单中添加

```java
<input type="hidden" name="_method" value="put" />
```

语句后，再次提交时，依然是使用 POST 方法。

原因是在 Spring Boot 的 META-INF/spring-configuration-metadata.json 配置文件中



默认是关闭 Spring 的 hiddenmethod 过滤器的。



所以直接在表单提交的数据中添加 "_method" 数据并不起作用。

解决办法就是在 Spring Boot 的配置文件 application.properties 中将 hiddenmethod 过滤器设置为启用即可。

```yml
spring: 
	mvc:
    	hiddenmethod:
      		filter:
        		enabled: true
```

