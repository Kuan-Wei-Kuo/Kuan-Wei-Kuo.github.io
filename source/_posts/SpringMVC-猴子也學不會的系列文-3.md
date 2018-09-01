---
title: SpringMVC. 猴子也學不會的系列文(3) - Aspect-Oriented Programming (AOP)
date: 2018-09-01 08:43:34
tags:
    - Spring
    - SpringMVC
---

# What is AOP ?
AOP只是一種概念，因此許多人會覺得聽起來非常抽象，很難理解它的意思，原先我們使用OOP時，會發現我們都是由上到下的完成一個流程，但AOP卻會Cross-cutting concern，將整個流程橫切，如安全 （Security）檢查、交易（Transaction）等系統層面的服務（Service），在一些應用程式之中常被見到安插至各個物件的處理流程之 中，這些動作在AOP的術語中被稱之為Cross-cutting concerns。

## AOP concepts(概念)

### Aspect
Aspect最大的作用即是收集各商務物件中的crosscutting concerns。在Spring中除了使用XML設定方式，也建議可以使用Annotation方式進行開發，會較於簡單。

### Joinpoint
Aspect在應用程式執行時加入商務流程的點或時機稱之為Joinpoint，被呼叫的時機有可能在某個方法的前面(@before)或後面(@after)，然而Joinpoint還可以讓我們取到傳入該流程的參數以及結果，也能夠攔截例外處理。

### Advice
顧名思義，Advice將會處理有使用“around”、“before”和“after”等不同類型的通知，這部分後面看程式碼會較於清楚。

### Pointcut
這部分可以看成，對於哪一個class的某個method進行關聯的動作，讓Advice可以知道我們要從哪個method進行通知處理。

### Introduction
這不要翻譯成介紹，稱之為引入較為適當，在Java很不幸的我們無法動態的修改已編譯過的類別，但是Introduction卻可以讓你動態為這個以編譯過的類別進行新增功能等動作，確實是滿驚人的，不過我沒有試過，有興趣的大大歡迎試試看。

### Target object
一個Advice被應用的對象或目標物件，如果直接使用Target Object進行撰寫橫切點等程式碼，會發現使用這種方式進行的crosscutting concerns都是需要自己設定ProxyBean，這實在是非常麻煩，但若使用Aspect則可以達到自動代理的機制。

### AOP proxy
AOP框架創建的對象，用來實現Aspect。然而Spring AOP是使用動態代理的方式進行。然後代理的理解可以看看這個從[代理機制初探 AOP](https://openhome.cc/Gossip/SpringGossip/FromProxyToAOP.html)，這邊有個簡單的小程式讓大快速理解代理機制。

### Weaving
Advice被應用到物件上的過程稱為Weaving，在AOP中縫合的方式有幾個時間點：編譯時期（Compile time）、類別載入時期（Classload time）、執行時期（Runtime）。(取自於:[AOP 觀念與術語](https://openhome.cc/Gossip/SpringGossip/AOPConcept.html))

## Advice 通知的類型

### @Before advice
在某連接點之前執行的通知，但不能令Join point停止執行，除非發生異常。

### @After return advice
在某個接點執行成功，並且返回值得時候觸發。

### @After throwing advice
在方法拋出異常退出時執行的通知。

### @After(final) advice
某個接點退出的時候執行，不管接點是否異常。

### @Around advice
這個功能算是滿常用的，上述的通知類型都會執行。

# Get Started

## 目錄
```
├─build.gradle
└─src
    ├─main
    │  ├─java
    │  │  └─com
    │  │      └─spring
    │  │          └─example
    │  │              ├─aspect
    │  │              │  └─UserAspect.java
    │  │              ├─config
    │  │              │  ├─AspectConfig.java
    │  │              │  ├─PersistenceJpaConfig.java
    │  │              │  ├─WebConfiguration.java
    │  │              │  └─WebServletInitializer.java
    │  │              ├─controller
    │  │              │  ├─HomeController.java
    │  │              │  └─UserController.java
    │  │              ├─entity
    │  │              │  └─User.java    
    │  │              ├─repository
    │  │              │  └─UserRepository.java
    │  │              └─service
    │  │                 ├─impl
    │  │                 │  └─UserServiceImpl.java
    │  │                 └─UserService.java
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
    compile group: 'org.springframework', name: 'spring-orm', version: "${springVersion}"
    compile group: 'org.springframework', name: 'spring-aspects', version: "${springVersion}"
 
    compile group: 'javax.servlet', name: 'jstl', version: '1.2'
    providedCompile 'javax.servlet:servlet-api:2.5'
    
    def springData = '2.0.0.RELEASE'
    compile group: 'org.springframework.data', name: 'spring-data-jpa', version: "${springData}"

    //DBCP (Connection Pool)
    compile group: 'org.apache.commons', name: 'commons-dbcp2', version: "2.0"

    //Hibernate
    def hibernateVersion = '5.2.10.Final'
    compile group: 'org.hibernate', name: 'hibernate-core', version: "${hibernateVersion}"

    def sqlVersion = '2.2.0'
    compile group: 'org.mariadb.jdbc', name: 'mariadb-java-client', version: "${sqlVersion}"

    def jacksonCoreVersion = '2.9.4'
    compile group: 'com.fasterxml.jackson.core', name: 'jackson-core', version: "${jacksonCoreVersion}"
    compile group: 'com.fasterxml.jackson.core', name: 'jackson-databind', version: "${jacksonCoreVersion}"
    
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

## Java

### UserService.java
```java
public interface UserService {
    public List<User> getUsers();
}
```

### UserServiceImpl.java
```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public List<User> getUsers() {
        return userRepository.findAll();
    }
	
}
```

### UserAspect.java
今天的主角出現了，文章開頭就介紹許多關於AOP的術語與觀念，這邊有看到@Before、@After，這兩個就是Advice通知類型，在這個Advice後面大家可以看到execution以及指定類別、方法，這邊就是Pointcut。
```java
@Aspect
public class UserAspect {
	
	private static Logger logger = LogManager.getLogger(UserAspect.class);
	
	@Before("execution(* com.spring.example.service.UserService.getUsers(..))")
	public void logBeforeV1(JoinPoint joinPoint) {
		logger.info("UserAspect.logBeforeV1() : " + joinPoint.getSignature().getName());
	}

	@Before("execution(* com.spring.example.service.UserService.*(..))")
	public void logBeforeV2(JoinPoint joinPoint) {
		logger.info("UserAspect.logBeforeV2() : " + joinPoint.getSignature().getName());
	}

	@After("execution(* com.spring.example.service.UserService.getUsers(..))")
	public void logAfterV1(JoinPoint joinPoint) {
		logger.info("UserAspect.logAfterV1() : " + joinPoint.getSignature().getName());
	}

	@After("execution(* com.spring.example.service.UserService.*(..))")
	public void logAfterV2(JoinPoint joinPoint) {
		logger.info("UserAspect.logAfterV2() : " + joinPoint.getSignature().getName());
	}

}
```

### UserController.java
```java
@RestController
@RequestMapping("/users")
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping(method = RequestMethod.GET)
    public List<User> getUsers() {
        return userService.getUsers();
    }

}
```

### AspectConfig.java
1. @Configuration 代表這是一個SpringConfig，在運行時會自動載入。
2. @EnableAspectJAutoProxy 開啟讓Aspect自動代理。
```java
@Configuration
@EnableAspectJAutoProxy
public class AspectConfig {

    @Bean
    public UserAspect userAspect() {
        return new UserAspect();
    }
	
}
```

## Console View
接下來我們於瀏覽器中輸入 http://localhost:8080/spring.example/users 即可於Console看到下面的畫面
<img src="/2018/09/01/SpringMVC-猴子也學不會的系列文-3/console.view.png" width="600">

# Reference
## [AOP 觀念與術語](https://openhome.cc/Gossip/SpringGossip/AOPConcept.html)   
## [Aspect Oriented Programming with Spring](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html)
## [Spring 实践：AOP](http://www.importnew.com/19041.html)