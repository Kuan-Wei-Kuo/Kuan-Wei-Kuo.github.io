---
title: SpringMVC. 猴子也學不會的系列文(2) - Hibernate with Spring Data JPA
date: 2018-08-29 20:54:52
tags: 
    - Spring
    - SpringMVC
---

# What is Hibernate ?
Hibernate擁有高效能與物件關聯的持久化的一款ORM，使用Java的人應該對於這款ORM一點都不陌生，在業界也是很常使用的。

# What is ORM ?
ORM是什麼? 其實就是Object-Relational Mapping(ORM)，主要功能為轉換程式語言中的物件與資料庫的關聯關係。

使用ORM還是比傳統JDBC方便的，如下:
1. 隱藏SQL邏輯，可以防止SQL-Injection。
2. 無須自己實現數據庫的操作。
3. 可以快速開發。
4. 構造數據化結構非常方便。

但使用ORM卻會犧牲效能，它會自動管理與映射這些物件以及實現複雜查詢的困難等等，因此還是要看實務上來進行使用。

# What is JPA
JPA是J2EE ORM的標準之一，JPA主要負責物件與資料表的映射，在JPA還沒出來前，聽說是很亂的，各種ORM各種自己的實現方式(小的我沒經歷過)。

# What is Spring Data JPA
Spring Data JPA主要基於Hibernate所開發的一款框架，可以讓我們更為方便的使用Hibernate與JPA。

# Get Started
在上一篇文章我們展示了簡單的SpringMVC範例，接下來我們將使用該範例進行一系列的修改。

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
    │  │              │  ├─PersistenceJpaConfig.java
    │  │              │  ├─WebConfiguration.java
    │  │              │  └─WebServletInitializer.java
    │  │              ├─controller
    │  │              │  ├─HomeController.java
    │  │              │  └─UserController.java
    │  │              ├─entity
    │  │              │  └─User.java    
    │  │              └─repository
    │  │                 └─UserRepository.java
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
這次新增了PersistenceJpaConfig、Entity、Repository等三個部份的程式碼。

### PersistenceJpaConfig.java
```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(basePackages = {"com.spring.example.repository"})
public class PersistenceJpaConfig {
	
    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
        LocalContainerEntityManagerFactoryBean entityManagerFactory = new LocalContainerEntityManagerFactoryBean();
        entityManagerFactory.setDataSource(dataSource());
        entityManagerFactory.setPackagesToScan("com.spring.example.entity");
        
        JpaVendorAdapter jpaVendorAdapter = new HibernateJpaVendorAdapter();
        entityManagerFactory.setJpaVendorAdapter(jpaVendorAdapter);
        
        Properties properties = new Properties();
        properties.setProperty("hibernate.dialect", "org.hibernate.dialect.MariaDBDialect");
        properties.setProperty("hibernate.show_sql", "true");
        entityManagerFactory.setJpaProperties(properties);
        
        return entityManagerFactory;
    }

    @Bean
    public BasicDataSource dataSource() {
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setDriverClassName("org.mariadb.jdbc.Driver");
        dataSource.setUrl("jdbc:mariadb://192.168.180.3:3306/test_db");
        dataSource.setUsername( "andy" );
        dataSource.setPassword( "123456" );
            
        return dataSource;
    }

    @Bean
    public JpaTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {
        JpaTransactionManager transactionManager = new JpaTransactionManager();
        transactionManager.setEntityManagerFactory(entityManagerFactory);
        return transactionManager;
    }

    @Bean
    public PersistenceExceptionTranslationPostProcessor exceptionTranslation() {
        return new PersistenceExceptionTranslationPostProcessor();
    }
	
}
```

### User.java
這些@Entity、@Table等等，都是JPA本身的標準，用來對應資料表的關係。
```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @Column(name = "id")
    private int id;

    @Column(name = "account")
    private String account;

    @Column(name = "nickname")
    private String nickname;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getAccount() {
        return account;
    }

    public void setAccount(String account) {
        this.account = account;
    }

    public String getNickname() {
        return nickname;
    }

    public void setNickname(String nickname) {
        this.nickname = nickname;
    }

    @Override
    public String toString() {
        return "User [id=" + id + ", account=" + account + ", nickname=" + nickname + "]";
    }

}
```

### UserRepository.java
JpaRepository是Spring Data JPA幫我們實現的功能，只要這樣做，基礎的CRUD就有了。
```java
@Repository
public interface UserRepository extends JpaRepository<User, Integer> {

}
```

### UserController.java
接下來展示如何使用Repository取得資料，並且輸出成Restful API。
```java
@RestController
@RequestMapping("/users")
public class UserController {

    @Autowired
    private UserRepository userRepository;

    @RequestMapping(method = RequestMethod.GET)
    public List<User> getUsers() {
        return userRepository.findAll();
    }

}
```

## Browser View
接下來我們於瀏覽器中輸入 http://localhost:8080/spring.example/users 即可看到下面的畫面
<img src="/2018/08/29/SpringMVC-猴子也學不會的系列文-2/bowser.view.png" width="600">

# Reference
## [Hibernate Tutorial](https://www.tutorialspoint.com/hibernate/orm_overview.htm)
## [Spring Data JPA介紹與使用](https://www.jianshu.com/p/633922bb189f)
