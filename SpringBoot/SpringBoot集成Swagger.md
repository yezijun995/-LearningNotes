# Spring Boot集成Swagger

## Swagger简介

Swagger 是一个规范和完整的框架，用于生成、描述、调用和可视化 RESTful 风格的 Web 服务。

总体目标是使客户端和文件系统作为服务器以同样的速度来更新。文件的方法、参数和模型紧密集成到服务器端的代码，允许 API 来始终保持同步。Swagger 让部署管理和使用功能强大的 API 从未如此简单。



## SpringBoot集成Swagger

### 1. 导入maven依赖

````xml
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>

<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
````

### 2.配置Swagger

````java
/**
 * Swagger配置类
 */
@Configuration  //配置类
@EnableSwagger2 //开启Swagger自动配置
public class SwaggerConfig {

    /**
     * 因为Swagger实例是Docket，所以构建一个DocketBean实例
     * 函数名可以任意取名
     * @return
     */
    @Bean
    public Docket docket(){
        return  new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(this.apiInfo())   //配置文档信息
                .enable(true)              //是否启用Swagger
                .groupName("Yifelix")      //配置API文档分组
                .select()
                //控制暴露的接口实例
                //RequestHandlerSelectors 配置要扫描接口的方式
                //basePackage   指定需要扫描的包
                //any           扫描全部
                //none          全部都不扫描
                //withClassAnnotation   扫描类上的注解 例如:RequestHandlerSelectors.withClassAnnotation(RestController.class)
                //withMethodAnnotation  扫描方法上的注解 例如:RequestHandlerSelectors.withClassAnnotation(GetMapper.class)
                .apis(RequestHandlerSelectors.basePackage("edu.fdzc.swagger.controller"))
                //过滤路径
                //ant 指定路径 例如:ant("/fdzc/**")
                .paths(PathSelectors.any())
                .build();
    }

    /**
     *  配置API文档的详细信息
     * @return
     */
    private ApiInfo apiInfo(){
        //作者
        Contact contact = new Contact("Yifelix","https://www.yifelix.cn","1016445037@qq.com");

        return new ApiInfoBuilder()
                .title("Swagger学习")     //标题
                .description("加油呀，你是比自己想象中的自己还要好的人")   //描述
                .termsOfServiceUrl("https://www.yifelix.cn")
                .version("v1.0")        //版本号
                .contact(contact)       //作者
                .build();
    }
}

````

### 3.测试登录   访问：http://localhost:8080/swagger-ui.html

![](http://picturebed.yifelix.cn/swagger-1.png)

### 4.配置多个分组

```java
@Bean
public Docket docket1(){
    return new Docket(DocumentationType.SWAGGER_2).groupName("zhangsan");
}

@Bean
public Docket docket2(){
    return new Docket(DocumentationType.SWAGGER_2).groupName("zhangsan");
}

@Bean
public Docket docket3(){
    return new Docket(DocumentationType.SWAGGER_2).groupName("zhangsan");
}
```

**效果展示:**

![](D:\笔记\image\swagger-2.png)

### 5.实体类配置

```java
//只要返回值是实体类，就会被扫描到Swagger中
@PostMapping("/user")
public User user(){
    User user = new User();
    user.setId(1);
    user.setUsername("zhangsan");
    user.setPassword("123456");
    return user;
}
```

**效果展示：**

![](D:\笔记\image\swagger-3.png)

### 6.swagger常用注解

| 作用范围           | **API**            | **使用位置**                     |
| ------------------ | ------------------ | -------------------------------- |
| 对象属性           | @ApiModelProperty  | 用在出入参数对象的字段上         |
| 描述返回对象的意义 | @ApiModel          | 用在返回对象类上                 |
| 协议集描述         | @Api               | 用于controller类上               |
| 协议描述           | @ApiOperation      | 用在controller的方法上           |
| Response集         | @ApiResponses      | 用在controller的方法上           |
| Response           | @ApiResponse       | 用在 @ApiResponses里边           |
| 非对象参数集       | @ApiImplicitParams | 用在controller的方法上           |
| 非对象参数描述     | @ApiImplicitParam  | 用在@ApiImplicitParams的方法里边 |

