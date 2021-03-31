---
title: Java9 + Spring5 + Gradle + Hibernate 개발환경 만들기
author: ChaeDae
date: 2017-12-08 19:52:00 +0800
categories: [Java, Spring]
tags: [Java, Spring, Gradle, Hibernate, 환경설정]
---

_<<Tistory 블로그에서 작성했던 글>>_

**이 포스팅은 지난 포스팅과 연결된 포스팅 입니다.**

## 지난 포스팅
---

[1. [JAVA] Java9 + Spring5 + Gradle 개발환경 만들기](/posts/Java9+Spring+Gradle-개발환경-만들기){:target="_blank"}  

[2. \[JAVA\] Java9 + Spring5 + Gradle 개발환경 만들기 (2)](/posts/Java9+Spring+Gradle-개발환경-만들기(2)){:target="_blank"}  

---

## Hibernate
---

> 포스팅을 시작하기에 앞서 Hibernate에 대해....
> 
> JPA : [woniper님의 블로그 :: JPA란 무엇인가?](http://blog.woniper.net/255){:target="_blank"}  
>       [woniper님 슬라이드 :: JPA 잘 (하는 척) 하기](https://www.slideshare.net/ssusere4d67c/jpa-53004111){:target="_blank"}  
> 
> Hibernate : [Y2K님의 블로그 :: Hibernate](http://netframework.tistory.com/entry/10-Hibernate){:target="_blank"}  
> 
> JPA를 먼저 확인 하시고 Hibernate 링크를 보시기 바랍니다.

---

이번엔 Spring 프로젝트에 Hibernate를 적용하려 한다.  

포스팅을 준비하면서 자료 검색을 상당히 많이 해봤는데, 쉽게 볼 물건이 아니었다.  

완벽한 이해를 바탕으로 적용할 수 있어야 ORM을 사용한다고 할 수 있다고 한다..  
(그래서 다음 포스팅은 xBatis 설정법으로 해야겠다는 생각도....)  

여하튼 처음 목표로 잡았던대로 Hibernate를 적용해보자.

Hibernate만 쓸 수도 있지만 Spring 프로젝트이니 만큼 Spring Data JPA와 같이 사용하기로 했다.  

이유인 즉,  
 Hibernate만 사용할 경우 기본적인 CRUD에 대해서 직접 코드를 작성해야 하는 불편함이 있는데,  
 Spring Data JPA를 사용하게 되면 기본적인 CRUD 뿐 아니라, 메소드명을 이용한 쿼리 생성, 페이징, 정렬 등 여러 이점이 있다고 해서 적용했다.  

 ex) findOneBy컬럼명(String 파라미터);  -- By뒤의 컬럼명으로 데이터를 하나 가져오는 쿼리  
       SELECT * FROM TB_TEMP WHERE 컬럼명 = 파라미터

그리고 Connection Pool 은 퍼포먼스가 가장 잘 나온다는 HikariCP를 사용 하기로 했다.

우선 build.gradle 파일에 관련 디펜던시를 추가해줘야 한다.

Spring Data JPA와 Hibernate, HikariCP 를 추가 해주자.

```groovy
def ver = { 
            ..........., 
            springJPA: '2.0.2.RELEASE', 
            hibernate: '5.2.12.Final', 
            hikariCP : '3.1.0', 
            ........... 
} 

dependencies { 
               ........... 
               compile "org.springframework:spring-orm:${ver.springJPA}" 
               compile "org.springframework.data:spring-data-jpa:${ver.springJPA}" 
               compile "org.hibernate:hibernate-core:${ver.hibernate}" 
               compile "com.zaxxer:HikariCP:${ver.hikariCP}" 
               ........... 
}
```

ver 쪽에 **springJPA, hibernate, HikariCP 의 버전 정보**를 넣었고,  
dependencies 쪽에 **Spring-ORM 과 Spring-Data-Jpa, Hibernate, HikariCP 관련 디펜던시 정보**를 추가했다.  
 ps) ver 쪽에는 콤마(,) 가 필요하지만, dependencies 쪽에 콤마를 넣는다면 오류가 난다.


## Spring ORM ?
---

> - Hibernate, iBatis, Eclipse Link 등 JPA 구현체를 사용할 때 통합 클래스를 제공해서 Spring 구성에 맞게 사용할 수 있도록 해주는 모듈
>
> - Session, Transaction 관리 등
> 

## Spring Data JPA ?
---

> - Spring Data용 JPA.
>
> - DAO를 구현할 필요 없이 메소드명을 이용해서 쿼리를 생성하도록 도와주는 모듈
>
> - 네이티브 쿼리 작성(JPQL)을 할 수 있는 애노테이션 제공(Query, NamedQuery) 



위의 내용을 넣었다면 이제 Gradle Build를 해주자.

![Setting Image1](/assets/img/posts/tistory/20171208-1.png){:width="400px" height="376px"}  

![Setting Image2](/assets/img/posts/tistory/20171208-2.png){:width="400px" height="196px"}  

빌드가 정상적으로 완료가 되었다면 아래와 같이 관련 라이브러리가 추가된 것을 확인할 수 있다.

![Setting Image3](/assets/img/posts/tistory/20171208-3.png){:width="400px" height="546px"}  

여기까지 완료가 되었다면, 이제 하이버네이트를 사용하기 위한 설정을 해보자.

모든 설정을 JAVA로만 하려고 했는데,  
DB 접속 정보등의 유동적인 부분에 대해서는 자바 소스보다 properties를 사용해서 관리하는 것이 맞는것 같아서 properties를 사용하기로 했다.

많은 삽을 푸는 와중에 주말에만 잠깐 작업을 했던 관계로 실제 포스팅을 쓰는 시점에선 완료된 내용만 있어서,  
기억을 더듬어 의식의 흐름대로 글을 써내려가 보겠다..

## DB 접속정보에 대한 Properties 파일 생성
---

src/main/resources 디렉토리 하위에 properties 파일을 저장할 패키지를 추가해주고 properties 파일을 생성한다.

![Setting Image4](/assets/img/posts/tistory/20171208-4.png){:width="240px" height="80px"}  

db.properties 파일은 기본적인 내가 사용할 DB의 정보를 담는 properties 파일이다.

### db.properties
---

```properties
########################################### # DB Information ########################################### 
jdbc.driverClassName=org.h2.Driver 
jdbc.url=jdbc:h2:mem:testDB;MODE=MySQL;DB_CLOSE_DELAY=-1 
jdbc.username=sa 
jdbc.password= 

########################################### # HikariCP Information (option) ########################################### 
# 연결 Timeout 
hikaricp.connection_timeout=30000 
# 최소 연결 수 
hikaricp.minimum_idle=5 
# connection pool 최대 사이즈 
hikaricp.maximum_pool_size=20 
# 대기 시간 
hikaricp.idle_timeout=300000
```

일단 테스트를 위해 h2db를 사용 했고, HikariCP Connection Pool 에 대한 설정값도 정의했다.

### hibernate.properties
---

```properties
########################################### # Hibernate Information ########################################### 
hibernate.dialect=org.hibernate.dialect.H2Dialect 
# SQL 출력여부 
hibernate.show_sql=true 
# SQL 정렬해서 보기 
hibernate.format_sql=true 
# SQL 코멘트 보기 
hibernate.use_sql_comment=true 
# DDL 자동 생성 
hibernate.hbm2ddl.auto=none
```

## DB Connection 설정 파일 생성
---

DB 관련 설정 파일을 담아두기 위한 패키지를 생성 해주고,  
그 안에 DataSourceConfig 와 HibernateConfig 라는 자바 클래스 파일을 2개 추가해 준다.

![Setting Image5](/assets/img/posts/tistory/20171208-5.png){:width="460px" height="108px"}  

`DataSourceConfig` 파일은 DB Connection에 대한 부분들을 정의하는 클래스이고,  
`HibernateConfig` 파일은 Hibernate 설정등에 대한 부분들을 정의하는 클래스이다.

굳이 이렇게 나눈 이유는 나중에 myBatis로의 전환을 고려해서 분리를 했다.

그럼 이제 파일 내용을 살펴보자.

### DataSourceConfig.java
---

```java
package com.chaedae.db.config; 

import javax.sql.DataSource; 
import org.springframework.beans.factory.annotation.Autowired; 
import org.springframework.context.annotation.Bean; 
import org.springframework.context.annotation.Configuration; 
import org.springframework.context.annotation.Primary; 
import org.springframework.context.annotation.PropertySource; 
import org.springframework.core.env.Environment; 
import com.zaxxer.hikari.HikariConfig; 
import com.zaxxer.hikari.HikariDataSource; 

/** 
 * <h1>Datasource Configuration</h1> 
 * old : root-context.xml > dataSource 
 * @author ChaeDae 
 */ 
@Configuration 
@PropertySource("classpath:config/props/db.properties") 
public class DataSourceConfig { 
    
    @Autowired private Environment env; 
    
    @Primary 
    @Bean 
    public DataSource dataSource() { 
        HikariConfig config = new HikariConfig(); 
        // Set Database Info 
        config.setDriverClassName(env.getProperty("jdbc.driverClassName")); 
        config.setJdbcUrl(env.getProperty("jdbc.url")); 
        config.setUsername(env.getProperty("jdbc.user")); 
        config.setPassword(env.getProperty("jdbc.passwd")); 

        // Set HikariCP Configuration 
        config.setConnectionTimeout(Integer.parseInt(env.getProperty("hikaricp.connection_timeout"))); 
        config.setMinimumIdle(Integer.parseInt(env.getProperty("hikaricp.minimum_idle"))); 
        config.setMaximumPoolSize(Integer.parseInt(env.getProperty("hikaricp.maximum_pool_size"))); 
        config.setIdleTimeout(Integer.parseInt(env.getProperty("hikaricp.idle_timeout"))); 
        
        return new HikariDataSource(config); 
    } 
}
```

DataSourceConfig 파일에서는 db.properties 파일에 정의 한 정보로 DB 접속 정보를 설정 해준다.

### HIbernateConfig.java
---

```java
package com.chaedae.db.config; 

import java.util.Properties; 
import org.springframework.beans.factory.annotation.Autowired; 
import org.springframework.context.ApplicationContext; 
import org.springframework.context.annotation.Bean; 
import org.springframework.context.annotation.ComponentScan; 
import org.springframework.context.annotation.Configuration; 
import org.springframework.orm.hibernate5.HibernateTransactionManager; 
import org.springframework.orm.hibernate5.LocalSessionFactoryBean; 
import org.springframework.transaction.annotation.EnableTransactionManagement; 

@Configuration 
@EnableTransactionManagement 
@ComponentScan(basePackages = { "com.chaedae" }) 
public class HibernateConfig { 
    
    @Autowired private ApplicationContext applicationContext; 
    
    @Autowired private DataSourceConfig dataSourceConfig; 
    
    @Bean 
    public LocalSessionFactoryBean getSessionFactory() throws Exception { 
        LocalSessionFactoryBean factoryBean = new LocalSessionFactoryBean(); 
        factoryBean.setDataSource(dataSourceConfig.dataSource()); 
        factoryBean.setPackagesToScan(new String[] {"com.chaedae.model"}); 
        
        Properties props = new Properties(); 
        props.load(applicationContext.getResource("classpath:/config/props/hibernate.properties").getInputStream()); 
        factoryBean.setHibernateProperties(props); 
        
        return factoryBean; 
    } 
    
    @Bean public HibernateTransactionManager getTransactionManager() throws Exception { 
        HibernateTransactionManager transactionManager = new HibernateTransactionManager(); 
        transactionManager.setSessionFactory(getSessionFactory().getObject()); 
        
        return transactionManager; 
    } 
}
```

여기서는 `LocalSessionFactoryBean`을 이용한 SessionFactory 생성과 HibernateTransactionManager를 설정 해준다. 

그리고 나서 AppInitializer 클래스 파일의 getRootConfigclasses() 메소드에 앞에서 설정한 Config 파일을 return 시켜준다.

### AppInitailizer.java
---

```java
@Override protected Class<?>[] getRootConfigClasses() { 
    return new Class[] { 
        HibernateConfig.class
    }; 
}
```

나중에 MyBatis를 쓰게 된다면 MyBatisConfig 파일을 생성하고 위의 부분만 변경해주면 된다.

여기까지 왔으면 Hibernate를 사용할 준비는 끝났다.

이제 적용한 내용이 정상적으로 구동하는지 테스트를 해보자.

## 샘플 데이터
---

테스트를 위해 h2db를 사용 했고, 확인을 위한 샘플 테이블이나 데이터를 넣기 위해 별도의 sql파일을 생성했다.

우선 샘플 테이블과 데이터를 넣기 위해 아래와 같이 패키지와 sql 파일을 생성한다.

![Setting Image6](/assets/img/posts/tistory/20171208-6.png){:width="239px" height="130px"}  

### H2DB-schema.sql
---

```sql
--------------------------- -- Sample DDL History -- --------------------------- 
DROP ALL OBJECTS; 

-- User 
CREATE TABLE TB_USER ( 
    USERID VARCHAR2(20) NOT NULL, 
    PWD VARCHAR2(128) NOT NULL, 
    USERNM VARCHAR2(20) NOT NULL, 
    HPNO VARCHAR2(20), 
    EMAIL VARCHAR2(50), 
    REGDT DATE, 
    REGID VARCHAR2(20) 
);
```

### H2DB-schema.sql
---

```sql
--------------------------- -- Sample DDL History (Data) -- ---------------------------
-- User 
INSERT INTO TB_USER ( 
    USERID 
    , PWD 
    , USERNM 
    , HPNO 
    , EMAIL 
    , REGDT 
    , REGID 
) VALUES ( 
    'master' 
    , 'a1234' 
    , 'master' 
    , '01012345678' 
    , 'master@chaedae.com' 
    , SYSDATE 
    , 'master' 
); 

INSERT INTO TB_USER ( 
    USERID 
    , PWD 
    , USERNM 
    , HPNO 
    , EMAIL 
    , REGDT 
    , REGID 
) VALUES ( 
    'test01' 
    , 'a1234' 
    , 'tester01' 
    , '01011112222' 
    , 'test01@chaedae.com' 
    , SYSDATE 
    , 'master' 
); 

INSERT INTO TB_USER ( 
    USERID 
    , PWD 
    , USERNM 
    , HPNO 
    , EMAIL 
    , REGDT 
    , REGID
) VALUES ( 
    'test02' 
    , 'a1234' 
    , 'tester02' 
    , '01033334444' 
    , 'test02@chaedae.com' 
    , SYSDATE 
    , 'master' 
); 

INSERT INTO TB_USER ( 
    USERID 
    , PWD 
    , USERNM 
    , HPNO 
    , EMAIL 
    , REGDT 
    , REGID 
) VALUES ( 
    'test03' 
    , 'a1234'
    , 'tester03' 
    , '01055556666' 
    , 'test03@chaedae.com' 
    , SYSDATE 
    , 'master' 
); 

commit;
```

이제 서버가 구동 될 때 해당 sql 파일을 읽어서 쿼리를 실행 시키도록 처리를 해보자.

DataSourceConfig 클래스 파일에 메소드를 추가 해준다.

### DataSourceConfig.java
---

```java
/** 
 * Default Database Schema & data 
 * @return 
 */ 
@Bean public DataSourceInitializer dataSourceInitializer() { 
    ResourceDatabasePopulator resourceDatabasePopulator = new ResourceDatabasePopulator(); 
    resourceDatabasePopulator.addScript(new ClassPathResource("/config/sql/H2DB-schema.sql")); 
    resourceDatabasePopulator.addScript(new ClassPathResource("/config/sql/H2DB-data.sql")); 
    
    DataSourceInitializer dataSourceInitializer = new DataSourceInitializer(); 
    dataSourceInitializer.setDataSource(dataSource()); 
    dataSourceInitializer.setDatabasePopulator(resourceDatabasePopulator); 
    
    return dataSourceInitializer; 
}
```

DataSourceInitailizer를 이용해서 미리 작성해 놓은 sql 파일을 읽어서 서버 구동 시 실행을 해준다.

## MVC 구성
---

그럼 이제 TB_USER에 대한 MVC를 생성해보자.

![Setting Image7](/assets/img/posts/tistory/20171208-7.png){:width="203px" height="150px"}  

VO, Service, DAO를 생성 해주고 Controller는 그냥 WelcomeController를 사용했다.

### User.java
---

```java
package com.chaedae.model; 

import java.io.Serializable; 
import java.util.Date; 
import javax.persistence.Entity; 
import javax.persistence.Id; 
import javax.persistence.Table; 

@SuppressWarnings("serial") 
@Entity(name = "TB_USER") 
@Table(name = "TB_USER") 
public class User implements Serializable { 
    
    @Id 
    String userId; 
    String pwd; 
    String userNm; 
    String hpNo; 
    String email; 
    Date regDt; 
    String regId; 
    
    Getter / Setter

}
```

User 테이블의 Entity와 Table 어노테이션에 각각 JPA의 Entity 정보와 DB의 테이블을 매핑 시켜준다.

참고 : [https://stackoverflow.com/questions/18732646/name-attribute-in-entity-and-table](https://stackoverflow.com/questions/18732646/name-attribute-in-entity-and-table){:target="_blank"}

### UserService.java
---

```java
package com.chaedae.service; 

import java.util.List; 
import org.springframework.beans.factory.annotation.Autowired; 
import org.springframework.stereotype.Service; 
import com.chaedae.dao.UserDAO; 
import com.chaedae.model.User; 

@Service public class UserService { 
    @Autowired 
    private UserDAO userDAO; 
    
    public User selectByUserId(User vo) { 
        return userDAO.selectByUserId(vo); 
    } 
    
    public List<User> selectList(User vo) { 
        return userDAO.selectList(vo); 
    } 
}
```

### UserDAO.java
---

```java
package com.chaedae.dao; 

import java.util.List; 
import org.hibernate.SessionFactory; 
import org.springframework.beans.factory.annotation.Autowired; 
import org.springframework.stereotype.Repository; 
import org.springframework.transaction.annotation.Transactional; 
import com.chaedae.model.User; 

@Repository public class UserDAO { 
    @Autowired 
    private SessionFactory sessionFactory; 
    
    @Transactional 
    public User selectByUserId(User vo) { 
        return this.sessionFactory.getCurrentSession().get(User.class, vo.getUserId()); 
    } 
    
    @SuppressWarnings("unchecked") 
    @Transactional 
    public List<User> selectList(User vo) { 
        return this.sessionFactory.getCurrentSession().createQuery("FROM TB_USER").list(); 
    } 
}
```

### WelcomeController.java
---

```java
package com.chaedae.welcome; 

import java.text.SimpleDateFormat; 
import java.util.Date; 
import java.util.Locale; 
import org.springframework.beans.factory.annotation.Autowired; 
import org.springframework.stereotype.Controller; 
import org.springframework.ui.Model; 
import org.springframework.web.bind.annotation.GetMapping; 
import com.chaedae.model.User; 
import com.chaedae.service.UserService; 

@Controller 
public class WelcomeController { 
    @Autowired 
    private UserService userService; 
    
    @GetMapping("/") 
    public String welcome(Locale locale, Model model) { 
        Date date = new Date(); 
        String simpleDate = new SimpleDateFormat("YYYY년 MM월 dd일 HH:mm:ss").format(date); 
        model.addAttribute("time", simpleDate); 
        
        User user = new User(); 
        user.setUserId("master"); 
        model.addAttribute("user", this.userService.selectByUserId(user)); 
        model.addAttribute("userList", this.userService.selectList(user)); 
        
        return "welcome"; 
    } 
}
```

위 3개 클래스는 특별히 설명할 부분이 없을 것 같다.

이제 화면에서 나오는지 확인 해야할 차례다.



## 테스트
---

우선 화면에서 리스트를 정상적으로 받아왔는지 확인하기 위해 JSTL 라이브러리와 taglib 라이브러리를 추가해주자.

### build.gradle
---

```groovy
def ver = { 
    ..........., 
    taglib: '1.1.2', 
    jstl : '1.2.1', 
    ........... 
} 

dependencies { 
    ........... 
    compile "javax.servlet.jsp.jstl:javax.servlet.jsp.jstl-api:${ver.jstl}" 
    compile group: 'taglibs', name: 'standard', version: "${ver.taglib}" 
    ........... 
}
```

### welcome.jsp
---

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Welcome to the Daeyeong world</title>
    
    <style>
        th, td {
            border: 1px solid black;
        }
    </style>
</head>
<body>
    <div>
        <h1>
            Welcome Spring Gradle Project 
        </h1>
        <p>What time is it? ${time}.</p>
    </div>
    
    <div>
        <div>User ID : ${user.userId} / User Name : ${user.userNm}</div> 
        <div>
            <table class="table" style="border: 1px solid black;">
                <caption align="center"><h1>User List</h1></caption>
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Name</th>
                        <th>Phone</th>
                        <th>Email</th>
                        <th>Join</th>
                    </tr>
                </thead>
                <tbody>
                    <c:forEach var="item" items="${userList}">
                        <tr>
                            <td>${item.userId}</td>
                            <td>${item.userNm}</td>
                            <td>${item.hpNo}</td>
                            <td>${item.email}</td>
                            <td>${item.regDt}</td>
                        </tr>
                    </c:forEach>
                </tbody>
            </table>
        </div>
    </div>
    
</body>
</html>
```
이제 구동을 해보자.

### Console
---

![Setting Image8](/assets/img/posts/tistory/20171208-8.png){:width="593px" height="400px"}  

### localhost:8080
---

![Setting Image9](/assets/img/posts/tistory/20171208-9.png){:width="557px" height="350px"}  

위와 같이 로그와 화면이 출력 되는 것을 확인할 수 있다.

해당 내용은 git 에서 받을 수 있다.

---

:: 현재까지 완성된 프로젝트 Git Repo ::

[https://github.com/Chaedae/SpringGradleProject/tree/Spring-Hibernate](https://github.com/Chaedae/SpringGradleProject/tree/Spring-Hibernate){:target="_blank"}

---

