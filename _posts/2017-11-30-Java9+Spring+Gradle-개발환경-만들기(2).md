---
title: Java9 + Spring5 + Gradle 개발환경 만들기 (2)
author: ChaeDae
date: 2017-11-30 19:52:00 +0800
categories: [Java, Spring]
tags: [Java, Spring, Gradle, 환경설정]
---

_<<Tistory 블로그에서 작성했던 글>>_

이전 포스팅에서 만든 프로젝트를 Spring 프로젝트로 바꾸는 작업을 해보자.

[[JAVA] Java9 + Spring5 + Gradle 개발환경 만들기](/posts/Java9+Spring+Gradle-개발환경-만들기){:target="_blank"}  

일단 내가 알던 Spring 환경설정법 하고는 확연히 바뀐 점이 있다.

내가 알던 스프링은 별도의 xml을 이용해서 설정들을 관리 했었는데,  
이게 Servlet 3.0 버전부터 JAVA로 만들 수 있게 되었다고 한다.

(자세한 설명 >>> [[스프링 3.1] web.xml이 없는 자바 웹 애플리케이션](http://whiteship.me/?p=13397){:target="_blank"}  

또 버전별로 달라진 부분들도 제법 많았다.

## 버전별 특징 
---

> Java 7 -> Java 8 : [Java 8의 특징](https://prezi.com/evgeuth9nzy-/java-8/){:target="_blank"}  
> 
> Java 8 -> Java 9 : [자바9 특징](https://www.slideshare.net/changhwanhan1/9-java9-features){:target="_blank"}  
> 
> Spring 3 -> Spring 4 : [SPRING FRAMEWORK 4.0 무엇이 달라졌나?](https://stargatex.wordpress.com/2014/11/19/spring-framework-4-0-%EB%AC%B4%EC%97%87%EC%9D%B4-%EB%8B%AC%EB%9D%BC%EC%A1%8C%EB%82%98/comment-page-1/){:target="_blank"}  
> 
> Spring 4 -> Spring 5 : [Springframework 5에서 바뀌는 것들에 대한 간단 정리 및 생각](https://roadmichi.blogspot.kr/2017/07/springframework-5.html){:target="_blank"}  
>    
---

이제 본론으로 넘어가서 위에 설명을 했던대로 Java Config 설정부터 해보자.

우선 기본으로 생성 되었던 패키지를 제거하고, 내 프로젝트에 맞는 패키지를 생성했다.

![Setting Image1](/assets/img/posts/tistory/20171130-1.png){:width="500px" height="116px"}

설정과 관련된 클래스가 들어갈 config 패키지와 welcome 화면을 장식하기 위한 welcome 패키지를 만들었다.

그후에 Config 클래스들을 생성해준다.

```
 o Config.java
 o Initializer.java
```

![Setting Image2](/assets/img/posts/tistory/20171130-2.png){:width="500px" height="150px"}

AppInitaillizer.java 파일을 먼저 살펴보자.

## AppInitaillizer.java
---

```java
package com.chaedae.config; 
import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer; 

/** 
 * <h1>Deployment Descriptor</h1> 
 * old : web.xml 
 * @author ChaeDae 
 */ 
 public class AppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer{ 
     /** 
      * Database Configuration 
      * old : root-context.xml 
      */ 
     @Override protected Class<?>[] getRootConfigClasses() { 
         return null; 
     } 
        
     /** 
      * Servlet Configuration 
      * old : servlet-context.xml 
      */ 
     @Override protected Class<?>[] getServletConfigClasses() { 
         return new Class[] { ServletConfig.class }; 
     } 
    
     /** 
      * Servlet Mapping
      */ 
     @Override protected String[] getServletMappings() { 
         return new String[] {"/"}; 
     } 
}
```

`AbstractAnnotationConfigDispatcherServletInitializer` 라는 추상 클래스를 상속 받으면,  
getRootConfigClasses, getServletConfigClasses, getServletMappings 클래스들이 오버라이드 된다.

각각의 기능을 살펴보자.


| 메소드명              | 역할                                 | XML                      |  
|:--------------------|:-----------------------------------|:--------------------------|  
| getRootConfigClasses | Bean정보, DB 정보, 트랜잭션 등의 기본환경 설정 | &lt;bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource"/&gt;<br>&nbsp;&nbsp;&nbsp;&nbsp; &lt;property name="driverClass" value="$"/&gt; <br> &nbsp;&nbsp;&nbsp;&nbsp;&lt;property name="url" value="$"/&gt; <br>&nbsp;&nbsp;&nbsp;&nbsp; &lt;property name="username" value="$"/&gt; <br>&nbsp;&nbsp;&nbsp;&nbsp; &lt;property name="password" value="$"/&gt; <br> &lt;/bean&gt; |  
| getServletConfigClasses | ViewResolver, Message 등의 Servlet Context 설정 | &lt;bean class=" org.springframework.web.servlet.view.InternalResourceViewResolver"&gt; <br>&nbsp;&nbsp;&nbsp;&nbsp; &lt;property name="prefix"&gt; <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;value&gt;/WEB-INF/views/&lt;/value&gt; <br>&nbsp;&nbsp;&nbsp;&nbsp; &lt;/property&gt; <br>&nbsp;&nbsp;&nbsp;&nbsp; &lt;property name="suffix"&gt; <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;value&gt;.jsp&lt;/value&gt; <br>&nbsp;&nbsp;&nbsp;&nbsp; &lt;/property&gt; <br> &lt;/bean&gt; |  
| getServletMappings | ServletMapping 정보 설정 |&lt;servlet-mapping&gt; <br>&nbsp;&nbsp;&nbsp;&nbsp; &lt;servlet-name&gt;dispatcher&lt;/servlet-name&gt; <br>&nbsp;&nbsp;&nbsp;&nbsp; &lt;url-pattern&gt;/&lt;/url-pattern&gt; <br> &lt;/servlet-mapping&gt; |  

기존 XML 방식을 사용하셨던 분이라면 대강 어떤 내용인지는 아실 것이라 생각한다.

(혹시 몰라 남겨놓는 XML방식 링크 -> [카루시에라 님 블로그](http://addio3305.tistory.com/39){:target="_blank"}

이제 하나씩 설정을 해보자.

일단 **getRootConfigClasses**의 경우 다음 포스팅에서 다룰 hibernate 설정 편에서 넣을 것이기 때문에 현재 상태를 유지한다.

**getServletConfigClasses** 메소드의 리턴 클래스는 앞서 생성한 ServletConfig.java 클래스 이다.

여기서 Servlet에 관한 설정을 하게 된다.

## ServletConfig.java
---

```java
package com.chaedae.config; 

import org.springframework.context.annotation.Bean; 
import org.springframework.context.annotation.ComponentScan; 
import org.springframework.context.annotation.Configuration; 
import org.springframework.web.servlet.config.annotation.EnableWebMvc; 
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer; 
import org.springframework.web.servlet.view.InternalResourceViewResolver; 
import org.springframework.web.servlet.view.JstlView; 

/** 
 * Spring WebMVC Configuration 
 * @author ChaeDae 
 */ 
@Configuration 
@EnableWebMvc 
@ComponentScan(basePackages = {"com.chaedae"}) 
public class ServletConfig implements WebMvcConfigurer { 
    
    /** 
     * ViewResolver Configuration 
     * @return viewResolver 
     */ 
    @Bean public InternalResourceViewResolver resolver() { 
        InternalResourceViewResolver resolver = new InternalResourceViewResolver(); 
        resolver.setViewClass(JstlView.class); 
        resolver.setPrefix("/webapp/views/"); 
        resolver.setSuffix(".jsp"); 
        
        return resolver; 
    } 
}

```
Serlvet 설정을 위해서는 Spring의 WebMvcConfigurer 인터페이스를 상속받아서 구현하게 된다.

그리고 몇가지 어노테이션들도 추가 했는데 해당 어노테이션들의 역할은 아래와 같다.

| 어노테이션        | 역할                                   |
|:---------------|:--------------------------------------|
| Configuration  | Spring 환경설정과 관련된 Class 라는 것을 정의 |
| EnableWebMvc   | Spring WebMVC를 사용하겠다는 것을 정의.     |
| ComponentScan(basePackages = {}) | @component 어노테이션이 등록된 Bean 클래스들을 찾는 패키지 위치를 지정 |

---
> 
> 포스팅을 위해 검색을 하던 중에 XML -> Java Config 로 변경하는것에 대해 정말 자세히 잘 포스팅 해주신 분의 블로그를 발견했다..
> 
> 세세한 설정이 왜 이렇게 되는지에 대해 알고 싶으신분들은 [메이킹러브 님의 블로그](http://zgundam.tistory.com/){:target="_blank"} 에서
> 
> [프로그래밍 > Spring > 전자정부프레임워크를 JavaConfig로 설정해보자](http://zgundam.tistory.com/category/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/Spring){:target="_blank"} 의 모든 글들을 읽어보는 것을 추천 드린다.
> 
---

## ViewResolver 설정
---

우선 간단하게 ViewResolver를 설정 해보았다. 

/webapp/views/ 경로의 모든 jsp 파일로 설정을 해놓았다. 

해당 경로를 추가해주자.

우선 main/webapp/WEB-INF 폴더를 추가 해주고

![Setting Image3](/assets/img/posts/tistory/20171130-3.png){:width="500px" height="410px"}

![Setting Image4](/assets/img/posts/tistory/20171130-4.png){:width="500px" height="551px"}

welcome.jsp 파일을 추가해준다.

![Setting Image5](/assets/img/posts/tistory/20171130-5.png){:width="500px" height="411px"}

![Setting Image6](/assets/img/posts/tistory/20171130-6.png){:width="333px" height="315px"}

![Setting Image7](/assets/img/posts/tistory/20171130-7.png){:width="333px" height="370px"}


생성된 welcome.jsp 파일에는 아래와 같이 내용을 넣어준다.

## welcome.jsp
---

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%> 
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd"> 
<head> 
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"> 
    <title>Insert title here</title> 
</head> 

<body>
    <h1> Welcome Spring Gradle Project </h1> 
    <p>What time is it? $.</p> 
</body> 
</html>
```

그리고 이제 이 파일로 넘겨줄 Controller를 생성해보자.

![Setting Image8](/assets/img/posts/tistory/20171130-8.png){:width="500px" height="173px"}

![Setting Image9](/assets/img/posts/tistory/20171130-9.png){:width="500px" height="602px"}

## WelcomeController.java
---

```java
package com.chaedae.welcome; 

import java.text.SimpleDateFormat; 
import java.util.Date; 
import java.util.Locale; 
import org.springframework.stereotype.Controller; 
import org.springframework.ui.Model; 
import org.springframework.web.bind.annotation.GetMapping; 

@Controller 
public class WelcomeController { 
    
    @GetMapping("/") 
    public String welcome(Locale locale, Model model) { 
        Date date = new Date(); 
        String simpleDate = new SimpleDateFormat("YYYY년 MM월 DD일 HH:mm:ss").format(date); 
        model.addAttribute("time", simpleDate); 
        
        return "welcome"; 
    } 
}
```

간단하게 현재 시간을 반환해주는 Controller이다.

이제 구현한 내용을 확인 해보자.

## 서버 구동
---

기본으로 제공해주는 Pivotal을 이용해서 서버를 구동 시켜보자.

Server 탭 > Pivotal tc..... 를 더블클릭하면 아래와 같은 화면이 나온다.

![Setting Image10](/assets/img/posts/tistory/20171130-10.png){:width="500px" height="78px"}

하단의 Modules 탭을 클릭하자.

![Setting Image11](/assets/img/posts/tistory/20171130-11.png){:width="1000px" height="540px"}

우측의 Add Web Module.. 클릭 후 현재 프로젝트를 추가 시켜준다.

![Setting Image12](/assets/img/posts/tistory/20171130-12.png){:width="500px" height="138px"}

![Setting Image13](/assets/img/posts/tistory/20171130-13.png){:width="500px" height="428px"}  


확인을 누른 뒤 프로젝트 선택 후 Edit 클릭 후 나오는 팝업에서 Path 경로에 '/' 를 제외하고 지워준다.

(위에서 등록 시 되면 좋은데 이클립스 버그인지 추가로 작업을 해줘야 한다)

![Setting Image14](/assets/img/posts/tistory/20171130-14.png){:width="500px" height="136px"}  

![Setting Image15](/assets/img/posts/tistory/20171130-15.png){:width="500px" height="279px"}  

저장 한 후 서버를 실행 시켜보자.

![Setting Image16](/assets/img/posts/tistory/20171130-16.png){:width="500px" height="319px"}  


### 정상 구동 되었을 때의 콘솔 내용
---

![Setting Image17](/assets/img/posts/tistory/20171130-17.png){:width="500px" height="135px"}  

정상적으로 구동이 되었다면 웹에서 확인을 해보자.

확인경로는 http://localhost:8080

![Setting Image18](/assets/img/posts/tistory/20171130-18.png){:width="500px" height="141px"}  

여기까지 기본적인 Spring 설정은 완료 되었다.

해당 내용은 git 에서 받을 수 있다.

---

:: 현재까지 완성된 프로젝트 Git Repo ::

 [https://github.com/Chaedae/SpringGradleProject/tree/SpringMVC](https://github.com/Chaedae/SpringGradleProject/tree/SpringMVC)

---

다음 포스팅은 AppInitailizer에서 뒤로 밀어놨던 Hibernate 설정에 대해 써볼까 한다.