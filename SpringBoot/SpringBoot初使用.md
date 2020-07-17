# SpringBoot初配置



## 1.IDEA的SprintBoot配置



### 1.1Maven依赖

```xml
<profile>   
    <id>jdk‐1.8</id>   
    <activation>     
      	<activeByDefault>true</activeByDefault>     
      	<jdk>1.8</jdk>   
    </activation>   
    <properties>     
   	 	<maven.compiler.source>1.8</maven.compiler.source>    	 						    <maven.compiler.target>1.8</maven.compiler.target>                      			<maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>   </properties> 
</profile>
```

```xml
<dependencies>
        <!--引入web模块-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    
         <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
</dependencies>
```

### 1.2编写主程序

```java
/**
 * @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWordMainApplication {

    public static void main(String[] args) {
        //Spring应用启动起来
        SpringApplication.run(HelloWordMainApplication.class, args);
    }
}

```

1.3编写先关Controller、Service

```java
@Controller
public class HelloController {

    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "Hello Word!";
    }
}
```



