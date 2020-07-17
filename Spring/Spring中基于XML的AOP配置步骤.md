## spring中基于XML的AOP配置步骤

##### 1、把通知Bean也交给spring来管理

##### 2、使用aop:config标签表明开始AOP的配置

##### 3、使用aop:aspect标签表明配置切面

​		id属性：是给切面提供一个唯一标识
​		 ref属性：是指定通知类bean的Id。

##### 4、在aop:aspect标签的内部使用对应标签来配置通知的类型

我们现在示例是让printLog方法在切入点方法执行之前之前：所以是前置通知

###### 		aop:before：表示配置前置通知

​                method属性：用于指定Logger类中哪个方法是前置通知

​				pointcut属性：用于指定切入点表达式，该表达式的含义指的是对业务层中哪些方法增强

切入点表达式的写法：

​		关键字：execution(表达式)

​		表达式：

​				访问修饰符  返回值  包名.包名.包名...类名.方法名(参数列表)

​		标准的表达式写法：

```
public void edu.fdzc.service.impl.AccountServiceImpl.saveAccount()
```

​		访问修饰符可以省略

```
void edu.fdzc.service.impl.AccountServiceImpl.saveAccount()
```

​		返回值可以使用通配符，表示任意返回值

```
edu.fdzc.service.impl.AccountServiceImpl.saveAccount()
```

​		包名可以使用通配符，表示任意包。但是有几级包，就需要写几个*.

```
* *.*.*.*.AccountServiceImpl.saveAccount())
```

包名可以使用..表示当前包及其子包

```
* *..AccountServiceImpl.saveAccount()
```

类名和方法名都可以使用*来实现通配

```
* *..*.*()
```

参数列表：

​		可以直接写数据类型：

​				 基本类型直接写名称           int

​				引用类型写包名.类名的方式   java.lang.String

​		可以使用通配符表示任意类型，但是必须有参数

 		可以使用..表示有无参数均可，有参数可以是任意类型

全通配写法：

```
* *..*.*(..)
```

实际开发中切入点表达式的通常写法：

​			切到业务层实现类下的所有方法

```
 * edu.fdzc.service.impl.*.*(..)
```





