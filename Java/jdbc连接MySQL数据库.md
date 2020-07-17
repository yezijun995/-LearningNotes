---
title: jdbc连接MySQL数据库
date: 2020-04-08 01:35:23
tags:
---
### 1.准备mysql驱动
&nbsp;&nbsp;&nbsp;&nbsp;Java连接Mysql需要先下载驱动包, [mysql-connector-java-5.1.37-bin](/download/mysql-connector-java-5.1.37-bin.jar)

----
### 2.加载驱动
**在项目下导入数据库的jar包，在代码中加入：** <br>
&nbsp;&nbsp;&nbsp;&nbsp;加载注册数据库驱动<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Class.forName(DRIVER);
	
----
### 3.数据库连接
**连接数据库**  <br>
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/abc", "root", 1234455);<br>
**第一个参数：** 需要连接的数据库的URL地址，端口号之后是需要连接的数据库名。<br>
**第二个参数：** 登陆数据库的账号。<br>
**第三个参数：** 登陆数据库的密码。<br>
