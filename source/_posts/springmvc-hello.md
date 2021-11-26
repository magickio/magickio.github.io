---

title: springmvc-hello
date: 2021-07-14 22:02:32
tags: springmvc
---

### 配置tomcat，修改idea控制台乱码

1. `首先要分清是tomcat日志编码，与idea的日志显示控制台编码`
2. `tomcat日志编码：cmd内 "cd /d tomcat根目录" "bin\catalina.bat run" 运行，"chcp65001"切换cmd为utf8，"chcp 936"切换cmd为gbk，确定tomcat日志编码，一般因为tomcat/conf/logging.properties java.util.logging.ConsoleHandler.encoding = UTF-8已设置为utf8`
3. `idea显示编码：windows默认用gbk所以idea显示默认为gbk编码，【一定】在 Help-- custom vm options 添加-Dfile.encoding=UTF-8，强制为utf8编码显示，不要自己改.vmoptions可能位置不对，idea会在用户目录复制一个`
4. `【切忌】自己改tomcat的logging.properties 为GBk 会导致调试时get/post参数乱码`

### web.xml的配置

![](https://magickio.oss-cn-shenzhen.aliyuncs.com/img/20210714220033.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
<!--            springmvc配置文件的属性-->
            <param-name>contextConfigLocation</param-name>
<!--            指定springmvc配置文件位置-->
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
<!--        服务器启动时创建对象，值越小优先级越高-->
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```

![image-20210717113528425](https://magickio.oss-cn-shenzhen.aliyuncs.com/img/20210717113535.png)

### springmvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

<!--    包扫描路径-->
    <context:component-scan base-package="com.yts.controller"/>

<!--    视图解析器，拼装链接的前后-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>

</beans>
```

```java
package com.yts.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class MyFirstController {

    @RequestMapping(value = "/hello")
    public String myfirstrequst() {
        return "success";
    }

}
```

