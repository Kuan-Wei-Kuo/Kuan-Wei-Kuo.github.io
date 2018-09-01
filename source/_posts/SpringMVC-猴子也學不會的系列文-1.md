---
title: SpringMVC. 猴子也學不會的系列文(1) - First SpringMVC
date: 2018-08-21 20:27:25
tags: 
    - Spring
    - SpringMVC
---

# What is Spring ?
Spring是一個lightweight(輕量級)的framework(框架)，它可以想像是各種框架的集成，因為它可以支援許多種框架，例如Struts, Hibernate, Tapestry, EJB, JSF等等，它包含幾種模組，例如IOC, AOP, DAO, Context, ORM, WEB MVC等等，不過我們第一步需要先了解IOC(控制反轉)以及Dependency Injection(依賴注入)。

# What is IOC、Dependency Injection ?
Spring容器實現了IOC，也是這整個框架的核心所在，容器將會創建、管理物件直到銷毀，並使用Dependency Injection(依賴注入)來管理這些物件(Bean)，然而Spring提供這兩種IOC容器，分別為BeanFactory、ApplicationContext，這兩部分的差別，後將依序簡單的探討。

## BeanFactory
這是Spring提供最簡單的IOC容器，是無法使用大部分模組的，例如AOP、Integration、Web等等，因此僅適合在有限資源的環境下使用，通常用於測試與非生產環境。

## ApplicationContext
大神也推薦使用這個IOC容器，該容器繼承BeanFactory並提供了更多的功能，因此我們可以方便的使用各種模組，來幫助開發。

# First SpringMVC
進入正題囉!! 首先帶大家看看SpringMVC應有的檔案架構。

## 目錄
```
├─build.gradle
└─src
    ├─main
    │  ├─java
    │  │  └─com
    │  │      └─spring
    │  │          └─example
    │  │              ├─config
    │  │              │  ├─WebConfiguration.java
    │  │              │  └─WebServletInitializer.java
    │  │              └─controller
    │  │                 └─HomeController.java
    │  ├─resources
    │  │  └─log4j2.xml
    |  └─webapp
    |     └─WEB-INF
    |          └─views
    |             └─home.jsp
```

## Gradle
```groovy
apply plugin: 'java'
apply plugin: 'war'
apply from: 'https://raw.github.com/akhikhl/gretty/master/pluginScripts/gretty.plugin'

repositories {
    mavenCentral()
}

dependencies {
    // Spring
    def springVersion = '5.0.1.RELEASE'
    compile group: 'org.springframework', name: 'spring-webmvc', version: "${springVersion}"
 
    compile group: 'javax.servlet', name: 'jstl', version: '1.2'
    providedCompile 'javax.servlet:servlet-api:2.5'
    
    // Log4j2
    def log4jVersion = '2.8.2'
    compile group: 'org.apache.logging.log4j', name: 'log4j-core', version: "${log4jVersion}"
    compile group: 'org.apache.logging.log4j', name: 'log4j-api', version: "${log4jVersion}"
    compile group: 'org.apache.logging.log4j', name: 'log4j-jcl', version: "${log4jVersion}"
    
    testImplementation 'junit:junit:4.12'
}

gretty {
    httpPort = 8080
    contextPath = 'spring.example'
}
```

## XML
### log4j2.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration status="OFF">
    <appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n" />
        </Console>
    </appenders>
    <loggers>
        <root level="trace">
            <appender-ref ref="Console" />
        </root>
    </loggers>
</configuration>
```

## Java
### WebConfiguration.java
在該部份我們需要配置靜態資源位置以及新增一個ViewResolver實例，為何需要配置ViewResolver，主因當Controller返回ModelAndView後，DispatcherServlet會交由ViewResolver物件來作View層的相關解析，所以需要設置ViewResolver實例。
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.view.InternalResourceViewResolver;
import org.springframework.web.servlet.view.JstlView;

@Configuration
@EnableWebMvc
@ComponentScan("com.spring.example.**")
public class WebConfiguration implements WebMvcConfigurer {

    @Bean
    public InternalResourceViewResolver viewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setViewClass(JstlView.class);
        viewResolver.setPrefix("/WEB-INF/views/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }

    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**").addResourceLocations("/resources/"); 
    }
	
}
```

### WebServletInitializer.java
AbstractAnnotationConfigDispatcherServletInitializer負責以下工作
1. 配置DispatcherServlet
2. 初始化Spring MVC容器
3. Spring容器

```java
import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

public class WebServletInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[] { WebConfiguration.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }

}
```

### HomeController.java
當為主頁的時候輸入home.jsp
```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class HomeController {
	
    @RequestMapping("/")
    public String home(ModelMap model) {
        // 傳送messages至JSP
        model.addAttribute("messages", "Spring MVC Example");
        return "home";
    }
	
}
```

## JSP

### home.jsp
由上方HomeController可以看出，我們經由Model傳送messages訊息給JSP，因此我們可以於JSP檔案中將該字串取得出來。
```jsp
<%@ page language="java" contentType="text/html; charset=ISO-8859-1" pageEncoding="ISO-8859-1"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
        <title>Welcome</title>
    </head>
    <body>
        <h1>${messages}</h1> <!-- 取得messages -->
    </body>
</html>
```

## Browser View
接下來我們於瀏覽器中輸入 http://localhost:8080/spring.example 即可看到下面的畫面
<img src="/2018/08/21/SpringMVC-猴子也學不會的系列文-1/bowser.veiw.png" width="600">

# Reference
## [Spring Tutorial](https://www.javatpoint.com/spring-tutorial)
## [開源框架: Spring Gossip](https://openhome.cc/Gossip/SpringGossip/)
## [极客学院 - IoC 容器](http://wiki.jikexueyuan.com/project/spring/ioc-containers.html)