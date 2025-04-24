说明：
我计划用idea，创建第一个springboot程序，但是作为新手完全不会弄，今天我就亲自尝试一边，并且出一期详细，完美的教程，亲测可以运行 


step1.

```bash
点击file ，  
选new，
选择project
```

step2.

```bash
选择左侧的spring boot，
右侧language勾选java，
type选择maven，
jdk选择18，
java选择17，
packaging选择jar
```

step3.点击next

```bash

developer tools 选择lombok
web选择spring web
template engines 选择 thymeleaf
sql选择mybatis framework  和 mysql driver
```


step4.勾选完毕，选择create

step5.等待创建完毕。
step6.新建hellocontroller类 C:\Users\demo6\src\main\java\com\example\demo\HelloController.java

```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello Spring Boot!";
    }
}
```


step7.修改C:\Users\Administrator\IdeaProjects\demo5\src\main\resources\application.properties

```bash
# 数据库连接配置
spring.datasource.url=jdbc:mysql://localhost:3306/db_spring?useSSL=false
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# 服务端口配置
server.port=5200


```
step8.运行，打开浏览器，http://localhost:5200/hello
如果显示，Hello Spring Boot!   表示成功