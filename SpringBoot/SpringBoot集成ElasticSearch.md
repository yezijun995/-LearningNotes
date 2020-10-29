# SpringBoot集成ElasticSearch



## 使用前配置

**添加依赖：**

```xml
<!--SpringBoot默认使用SpringData ElasticSearch模块进行操作-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

配置文件

```yaml
spring:
  rabbitmq:
    host: 192.168.106.131		# 主机地址
    port: 5672					#端口号
    virtual-host: /ems			# 虚拟主机
    username: ems				#用户名
    password: 123456			#密码
```



## 使用 ElasticSearch

