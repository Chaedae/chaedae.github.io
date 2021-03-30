---
title: Properties 파일로 메시징 처리하기
author: ChaeDae
date: 2016-02-02 15:03:00 +0800
categories: [Java, Spring]
tags: [JAVA, Spring, Message]
---

프로젝트 진행중에 엑셀 다운로드를 하는 부분이 있었는데 컬럼명이 한글이다 보니 다국어 처리를 하게 되었다.

JAVA단에 한글 넣는것은 극도로 꺼리기 때문에 (주석도 최대한 영어로 적는다.)
<br/>
한글 처리를 위해 메시징 처리를 하였다.


따로 세팅을 잡아주는 사람도 없기 때문에 모든걸 직접 해야 하는 상황이다.
<br/>
(2년반된 개발자로써 좋은 기회라 생각한다. 이거저거 해보는게 좋다고 생각하기 때문에..)


이번에도 역시 구글신님의 도움을 받아서 
<br/>
`ReloadableResourceBundleMessageSource` 라는 스프링 클래스를 사용하여 처리를 하였다.
<br/>
기본적인 스프링 프레임워크의 라이브러리를 이용하기 때문에 pom.xml에 별도의 플러그인을 추가할 필요가 없을 것이다.

만약 필요하다면

## pom.xml
---

```xml
<dependency> 
    <groupId>org.springframework</groupId> 
    <artifactId>spring-core</artifactId> 
    <version>3.2.11.RELEASE</version> 
</dependency> 
<dependency> 
    <groupId>org.springframework</groupId> 
    <artifactId>spring-context</artifactId> 
    <version>3.2.11.RELEASE</version> 
</dependency> 
<dependency> 
    <groupId>org.springframework</groupId> 
    <artifactId>spring-context-support</artifactId> 
    <version>3.2.11.RELEASE</version> 
</dependency>
```

위의 내용을 pom.xml의 dependencies 안에 넣어주면 된다.

pom.xml을 확인하고 나서는 spring-context.xml 파일에 아래 내용을 추가해준다.

## spring-context.xml
---

```xml
<bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
    <property name="defaultEncoding" value="UTF-8"/> 
    <property name="basenames"> 
        <list> 
            <value>classpath:messages/msg</value> 
        </list> 
    </property> 
    
    <property name="fallbackToSystemLocale" value="false"/> 
    <property name="cacheSeconds" value="5"/> 
</bean>
```
 
인코딩은 UTF-8로 설정하고 5초에 한번씩 해당 프로퍼티를 다시 읽는 옵션을 주었다.(cacheSeconds)

`fallbackToSystemLocale` 옵션은 Locale 정보가 없을 경우 시스템의 Locale을 사용하는 옵션이다.

눈여겨 볼 곳은 `<value>classpath:messages/msg</value>` 항목이다.

messages 는 메시지 파일들이 있는 경로이고, msg 는 해당 프로퍼티 파일의 헤더이름이다.

사용하는 다국어에 따라 **msg_ko.properties** 나 **msg_en.properties** 등으로 만들어두면 
<br/>
서버의 Locale 정보에 따라 해당 메시지 파일을 읽어온다.
  
xml세팅은 여기까지다.

이제 실제 properties 파일을 만들어서 안에 다국어 처리할 메시지를 넣는다.

## msg_ko.properties
---

```properties
########## Message Properties ########## 
test.name=안녕하세요 
common.name=채대의 블로그 입니다. 
common.args={0} 많은 {1} 부탁 드립니다.
```

이제 JAVA단에서 사용하는 방법을 알아보겠다.

미리 xml에 세팅을 해놓았기 때문에 매우 심플하다.

## Java Source
---

```java
package com.chaedae.test.controller; 

import org.springframework.beans.factory.annotation.Autowired; 
import org.springframework.context.MessageSource; 
import org.springframework.context.i18n.LocaleContextHolder; 

public class TestController { 
    @Autowired private MessageSource msg; 
    
    public static void main(String[] args) { 
        String val = "일반 텍스트"; 
        String hi = msg.getMessage("test.name", null, LocaleContextHolder.getLocale()); 
        String blog = msg.getMessage("common.name", null, LocaleContextHolder.getLocale()); 
        String message = msg.getMessage("common.args", new String{"앞으로","관심"}, LocaleContextHolder.getLocale()); 
        
        system.out.println(val); 
        system.out.println(hi); 
        system.out.println(blog); 
        system.out.println(message); 
    } 
}
```
  
위와 같이 사용하면 된다. 

getMessage() 의 각 파라미터를 설명하자면..

 `첫번째 String` 은 해당 프로퍼티의 key 값이고,

 `두번째 String[]` 은 해당 메시지에 블라블라 이런식으로 index가 정해져 있을 때 해당 순서대로 값을 넣어주는 역할을 한다. (일반 text라면 null로 주면 된다.)

 세번째는 Locale 정보인데, 나는 따로 Locale설정은 따로 설정하지 않고 LocaleContextHolder를 이용해서 현재 접속된 Locale 정보를 넣어주었다.

결과값은 다음과 같이 출력 된다.

```
일반텍스트

안녕하세요.

채대의 블로그 입니다.

앞으로 많은 관심 부탁 드립니다.
```
