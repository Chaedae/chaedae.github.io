---
title: RestAPI에 Swagger 적용하기
author: ChaeDae
date: 2019-01-18 19:52:00 +0800
categories: [Java, Api]
tags: [Java, Spring, Swagger, rest, Api]
---

_<<Tistory 블로그에서 작성했던 글>>_

[이전에 만든 Rest 연습 프로젝트](/posts/SpringBoot+H2+RestAPI-연습){:target="_blank"}에 Swagger2를 적용해보았다.

**Swagger** 는 프로젝트에 정의되어 있는 URL 매핑 정보를 브라우저로 한눈에 볼 수 있게 해주는 자동화 라이브러리이다.  

또한, Postman 처럼 URL 호출 테스트도 지원한다.

그럼 이제 Swagger 적용을 해보자.

## 라이브러리 추가
--- 

우선 Swagger Dependency를 추가한다.

```xml
<dependency> 
    <groupId>io.springfox</groupId> 
    <artifactId>springfox-swagger2</artifactId> 
    <version>2.8.0</version> 
</dependency> 

<dependency> 
    <groupId>io.springfox</groupId> 
    <artifactId>springfox-swagger-ui</artifactId> 
    <version>2.8.0</version> 
</dependency>
```

**Swagger-ui** 는 브라우저에서 깔끔한 포맷으로 확인할 수 있는 Html 페이지를 제공해주는 라이브러리이다.

(Swagger-ui 없이 http://localhost:8080/v2/api-docs 를 호출해서 매핑된 url 정보만을 얻을수도 있다.)

이제 추가한 라이브러리를 써먹어보자.

## 환경설정
---

### SwaggerConfig.java
---

```java
package com.chaedae.restdemo.config; 

import org.springframework.context.annotation.Bean; 
import org.springframework.context.annotation.Configuration; 
import springfox.documentation.builders.ApiInfoBuilder; 
import springfox.documentation.builders.PathSelectors; 
import springfox.documentation.builders.RequestHandlerSelectors; 
import springfox.documentation.service.ApiInfo; 
import springfox.documentation.service.Contact; 
import springfox.documentation.spi.DocumentationType; 
import springfox.documentation.spring.web.plugins.Docket; 
import springfox.documentation.swagger2.annotations.EnableSwagger2; 

@Configuration 
@EnableSwagger2 
public class SwaggerConfig { 
    @Bean 
    public Docket api() { 
        return new Docket(DocumentationType.SWAGGER_2) 
                       .apiInfo(apiInfo()) 
                       .select() 
                       .apis(RequestHandlerSelectors.basePackage("com.chaedae.restdemo.controller")) 
                       .paths(PathSelectors.any()) 
                       .build(); 
    } 
    
    private ApiInfo apiInfo() { 
        return new ApiInfoBuilder() 
                       .title("Welcome, Chaedae's Rest Api Docs") 
                       .description("Thank you for visiting my blog") 
                       .contact(new Contact("ChaeDae", "http://chaedae.tistory.com", "chaedae0@gmail.com")) 
                       .version("1.0.0") 
                       .build(); 
    } 
}
```

`@EnableSwagger2` 어노테이션으로 Swagger2를 사용하는 클래스임을 명시해준다.

api() 메소드를 통해 Docket을 생성해준다.

 - apiInfo : 페이지의 title, 설명, contact 등의 정보를 설정한다.  
 - apis : Swagger를 적용할 패키지 경로  
            (PathsSelectors.any() 를 사용해서 모든 패키지를 스캔할 수도 있지만 다른 라이브러리 내에 있는 api도 노출이 되기 때문에 스캔하고자 하는 패키지를 명시해주는게 좋다.)  
 - paths : "/api/**" 식으로 특정 url만 설정하는 것도 가능하다. 여기선 Controller 패키지를 명시했기때문에 모든 url을 추출하도록 설정했다.

## 테스트
---

설정이 끝났으니 테스트를 해보자.

### http://localhost:8080/swagger-ui.html 

![Setting Image1](/assets/img/posts/tistory/20190118-1.png){:width="800px" height="455px"}  

ApiInfo에 적은 내용도 정상적으로 나오고 API 정보가 정의된 PostController와 그에 대한 Models 가 보인다.

### Tabs
---
Models 탭에서는 어떤 파라미터들이 있는지와 그 타입에 대한 정보가 나온다.

![Setting Image2](/assets/img/posts/tistory/20190118-2.png){:width="800px" height="186px"}  

탭들을 열어보면 아래와 같이 나온다.

![Setting Image3](/assets/img/posts/tistory/20190118-3.png){:width="800px" height="873px"}  


### CREATE
---

이제 Swagger에서 테스트를 하기 위해 PostController > POST /api/post createPost 를 클릭해보자.

![Setting Image4](/assets/img/posts/tistory/20190118-4.png){:width="800px" height="884px"}  

여기서 Try it out을 누르면 필요한 파라미터들과 타입이 정의되어 있는데 여기에 입력할 값들을 세팅해주고 Execute를 누르면 API를 호출하게 된다.

![Setting Image5](/assets/img/posts/tistory/20190118-5.png){:width="800px" height="643px"}  

Execute를 클릭해서 호출을 하면 하단에 내가 요청한 URL 정보와 리턴값들이 노출된다.

![Setting Image6](/assets/img/posts/tistory/20190118-6.png){:width="800px" height="518px"}  

### GET
---

이제 GET 호출을 통해 입력한 값이 정상적으로 들어갔는지 확인해보자.

(POST 테스트할 때  postno를 0으로 넣어버렸는데.. Generate 설정을 했기 때문에 넣지 않으면 자동으로 1번부터 생성된다..)

![Setting Image7](/assets/img/posts/tistory/20190118-7.png){:width="800px" height="554px"}  

---

:: Git Repo ::  
[https://github.com/Chaedae/restdemo](https://github.com/Chaedae/restdemo){:target="_blank"}

---