---
title: Apache Tiles 적용
author: ChaeDae
date: 2018-09-04 19:52:00 +0800
categories: [Java, Spring]
tags: [Java, Spring, Tiles]
---

_<<Tistory 블로그에서 작성했던 글>>_

지난 프로젝트에서 적용했던 [Apache Tiles](https://tiles.apache.org){:target="_blank"} 에 대해 정리해본다.

진행했던 프로젝트의 화면 구성이 헤더, 좌측 메뉴, footer, 그리고 본문.

이런식으로 구성이 되어 있다보니 반복되는 부분들이 많아서 

Apache Tiles를 적용해서 Layout 관리를 했다.

대충 그림으로 보자면 아래와 같다.

<table class="txc-table" style="border: none; border-collapse: collapse; width: 402px;" border="0" width="402" cellspacing="0" cellpadding="0"><tbody><tr><td style="width: 401px; height: 41px; border: 1px solid #cccccc;" colspan="2" rowspan="1"><p style="text-align: center;">Header</p></td></tr><tr><td style="width: 97px; height: 294px; border-bottom: 1px solid #cccccc; border-right: 1px solid #cccccc; border-left: 1px solid #cccccc;"><p style="text-align: center;">Menu</p></td><td style="width: 304px; height: 294px; border-bottom: 1px solid #cccccc; border-right: 1px solid #cccccc;"><p style="text-align: center;">Contents&nbsp;</p></td></tr><tr><td style="width: 401px; height: 39px; border-bottom: 1px solid #cccccc; border-right: 1px solid #cccccc; border-left: 1px solid #cccccc;" colspan="2" rowspan="1"><p style="text-align: center;">Footer&nbsp;</p></td></tr></tbody></table>


## 라이브러리 추가
---

우선 Tiles를 사용하기 위해 Dependency를 추가해준다.

### pom.xml
---

```xml
<dependencies> 
    .......... 
    <!-- Apache Tiles --> 
    <dependency> 
        <groupId>org.apache.tiles</groupId> 
        <artifactId>tiles-extras</artifactId> 
        <version>3.0.8</version> 
    </dependency> 
    .......... 
</dependencies>
```

### gradle.build
---

```groovy
dependencies { 
               .......... 
               compile group: 'org.apache.tiles', name: 'tiles-extras', version: '3.0.8' 
               .......... 
}
```

Dependency 추가가 완료 되었다면, Bean 등록을 진행한다.

XML 설정과 JAVA 설정 둘 다 정리해둔다.

## 1. XML Configuration
---

### servlet-context.xml
---

```xml
<!-- Tiles Configuration --> 
<beans:bean id="tilesViewResolver" class="org.springframework.web.servlet.view.tiles3.TilesViewResolver"> 
    <beans:property name="order" value="1" /> 
</beans:bean> 

<beans:bean id="tilesConfigurer" class="org.springframework.web.servlet.view.tiles3.TilesConfigurer"> 
    <beans:property name="definitions"> 
        <beans:list> 
            <beans:value>/WEB-INF/tiles/tiles.xml</beans:value> 
        </beans:list> 
    </beans:property> 
</beans:bean>
```

Spring에서 Tiles에 대한 `TilesViewResolver`와 `TilesConfigurer` 클래스를 제공해주고 있어서 이것을 사용했다.

주의할 점은 TilesViewResolver Bean 하위에 order property 항목이 있는데,  
별도로 선언되어 있는 ViewResolver(아마 InternalResourceViewResolver) 보다 낮게 설정해야 한다.

**TilesViewResolver가 먼저 로드 된 후 ViewResolver가 로딩 되어야 한다.**

TileConfigurer 의 list에는 Tiles의 설정파일 경로를 넣어준다.

### tiles.xml
---

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE tiles-definitions PUBLIC "-//Apache Software Foundation//DTD Tiles Configuration 3.0//EN" "http://tiles.apache.org/dtds/tiles-config\_3\_0.dtd"> 

<tiles-definitions> 
    <definition name="sampleTiles" template="/WEB-INF/tiles/layouts/layout.jsp"> 
        <put-attribute name="header" value="/WEB-INF/tiles/layouts/header.jsp"/> 
        <put-attribute name="menu" value="/WEB-INF/tiles/layouts/menu.jsp"/> 
        <put-attribute name="footer" value="/WEB-INF/tiles/layouts/footer.jsp"/> 
    </definition> 
    
    <!-- 로그인 화면 --> 
    <definition name="/chaedae/login/*" template="/WEB-INF/tiles/layouts/empty.jsp"> 
        <put-attribute name="body" value="/WEB-INF/views/chaedae/login/.jsp" /> 
    </definition> 
    
    <!-- 메인 화면 --> 
    <definition name="/chaedae/*/*" extends="sampleTiles"> 
        <put-attribute name="body" value="/WEB-INF/views/chaedae/{1}/.jsp" /> 
    </definition> 
</tiles-definitions>
```

최상단 sampleTiles를 선언하면서 헤더와 메뉴 푸터를 분리 해준 뒤 메인 화면쪽에서 상속받아 사용하도록 처리했다.

와일드 카드를 이용해서 경로 정보를 설정해줬고, (/chaedae/*/* ==> /chaedae/main/Main.jsp)  
로그인 화면은 별도의 레이아웃이 필요없는 관계로 빈화면을 넣었다.

## 2. JAVA Configuration
---

### ServletConfig.java
---

```java
package com.chaedae.config;

import org.springframework.context.MessageSource; 
import org.springframework.context.annotation.Bean; 
import org.springframework.context.annotation.ComponentScan; 
import org.springframework.context.annotation.Configuration; 
import org.springframework.context.support.ResourceBundleMessageSource; 
import org.springframework.web.servlet.config.annotation.EnableWebMvc; 
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer; 
import org.springframework.web.servlet.view.InternalResourceViewResolver; 
import org.springframework.web.servlet.view.JstlView; 
import org.springframework.web.servlet.view.tiles3.TilesConfigurer; 
import org.springframework.web.servlet.view.tiles3.TilesView; 
import org.springframework.web.servlet.view.tiles3.TilesViewResolver; 
import com.chaedae.tiles.config.TilesDefinitionsConfig; 

@Configuration 
@EnableWebMvc 
@ComponentScan(basePackages = {"com.chaedae"}) 
public class ServletConfig implements WebMvcConfigurer { 
    /** 
     * Apache-Tiles Configuration 
     * @return tilesViewResolver 
     */ 
    @Bean public TilesViewResolver viewResolver() { 
        TilesViewResolver viewResolver = new TilesViewResolver(); 
        viewResolver.setViewClass(TilesView.class); 
        viewResolver.setOrder(1); 
        
        return viewResolver; 
    } 
    
    /** 
     * Apache-Tiles Init 
     * @return tilesConfigurer 
     */ 
    @Bean 
    public TilesConfigurer getTilesConfigurer() { 
        TilesConfigurer tilesConfigurer = new TilesConfigurer(); 
        tilesConfigurer.setCheckRefresh(true); 
        tilesConfigurer.setDefinitionsFactoryClass(TilesDefinitionsConfig.class); 
        TilesDefinitionsConfig.addDefinitions(); 
        
        return tilesConfigurer; 
    } 
    
    /** 
     * ViewResolver Configuration 
     * @return viewResolver 
     */ 
    @Bean public InternalResourceViewResolver resolver() { 
        InternalResourceViewResolver resolver = new InternalResourceViewResolver(); 
        resolver.setViewClass(JstlView.class); 
        resolver.setPrefix("/WEB-INF/views/"); 
        resolver.setSuffix(".jsp"); 
        
        return resolver; 
    } 
}
```

XML 설정을 Java에 그대로 녹였다고 보면 된다.

tiles.xml 파일 대신 TilesDefinitionsConfig 클래스를 만들어서 Definitions를 지정해줬다.

### TilesDefinitionsConfig.java
---

```java
package com.chaedae.tiles.config; 

import java.util.HashMap; 
import java.util.Map; 
import org.apache.commons.lang.StringUtils; 
import org.apache.tiles.Attribute; 
import org.apache.tiles.Definition; 
import org.apache.tiles.definition.DefinitionsFactory; 
import org.apache.tiles.request.Request; 

/** 
 * <h1>Apache-Tiles Definitions Config</h1> 
 * xml : tiles.xml 
 * @author ChaeDae 
 * 
 */ 
public class TilesDefinitionsConfig implements DefinitionsFactory { 
    private static final Map<String, Definition> TILES_DEFINITIONS = new HashMap<>(); 
    
    @SuppressWarnings("serial") 
    private static final Map<String, String> TILES_LAYOUTS = new HashMap<>() { 
        { 
            this.put("TILES_EMPTY_LAYOUT", "/WEB-INF/tiles/layouts/empty.jsp"); 
            this.put("TILES_LAYOUT", "/WEB-INF/tiles/layouts/layout.jsp"); 
        } 
    }; 
    
    private static String TILES_HEADER = "/WEB-INF/tiles/layouts/header.jsp"; 
    private static String TILES_MENU = "/WEB-INF/tiles/layouts/menu.jsp"; 
    private static String TILES_FOOTER = "/WEB-INF/tiles/layouts/footer.jsp"; 
    
    @Override public Definition getDefinition(String name, Request tilesContext) { 
        return TILES_DEFINITIONS.get(name); 
    } 
    
    /** 
     * Init 
     */ 
    public static void addDefinitions() { 
        // Login Layout 
        setDefinitions("/chaedae/login/Login", "/WEB-INF/views/chaedae/login/Login.jsp", "TILES_EMPTY_LAYOUT"); 
        // default Layout 
        setDefinitions("/chaedae/main/Main", "/WEB-INF/views/chaedae/main/Main.jsp", "TILES_LAYOUT"); 
    } 
        
    /** 
     * Set Definitions 
     * @param name definition name 
     * @param bodyName 
     * @param layouts layout name 
     */ 
    public static void setDefinitions(String name, String bodyName, String layout) { 
        Map<String, Attribute> attributes = new HashMap<>(); 
        
        if (!StringUtils.equals(layout, "TILES_EMPTY_LAYOUT")) { 
            attributes.put("header", new Attribute(TILES_HEADER)); 
            attributes.put("menu", new Attribute(TILES_MENU)); 
            attributes.put("footer", new Attribute(TILES_FOOTER)); 
        } 
        attributes.put("body", new Attribute(bodyName)); 
        
        Attribute baseTemplate = new Attribute(TILES_LAYOUTS.get(layout)); 
        TILES_DEFINITIONS.put(name, new Definition(name, baseTemplate, attributes)); 
    } 
}
```

원래는 위의 XML 설정과 동일하게 와일드카드를 이용해서 매핑을 하려 했는데, 소스가 너무 복잡해질 것 같아서 간단하게 특정 페이지들을 매핑 하였다.

페이지가 몇 개 없고 직관적으로 관리하겠다 한다면 사용할 수야 있겠지만,... 나중을 생각하면 그냥 XML 로 설정하는게 더 나은 선택일 것 같다.

XML파일을 참조하는 자바 설정은 ServletConfig에서 Definitions관련 설정을 변경 해주면 된다.

### ServletConfig.java (with tiles.xml)
---

```java
package com.chaedae.config; 

import org.springframework.context.MessageSource; 
import org.springframework.context.annotation.Bean; 
import org.springframework.context.annotation.ComponentScan; 
import org.springframework.context.annotation.Configuration; 
import org.springframework.context.support.ResourceBundleMessageSource; 
import org.springframework.web.servlet.config.annotation.EnableWebMvc; 
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer; 
import org.springframework.web.servlet.view.InternalResourceViewResolver; 
import org.springframework.web.servlet.view.JstlView; 
import org.springframework.web.servlet.view.tiles3.TilesConfigurer; 
import org.springframework.web.servlet.view.tiles3.TilesView; 
import org.springframework.web.servlet.view.tiles3.TilesViewResolver; 
import com.chaedae.tiles.config.TilesDefinitionsConfig; 

@Configuration 
@EnableWebMvc 
@ComponentScan(basePackages = {"com.chaedae"}) 
public class ServletConfig implements WebMvcConfigurer { 
    /** 
     * Apache-Tiles Configuration 
     * @return tilesViewResolver 
     */ 
    @Bean public TilesViewResolver viewResolver() { 
        TilesViewResolver viewResolver = new TilesViewResolver(); 
        viewResolver.setViewClass(TilesView.class); 
        viewResolver.setOrder(1); 
        
        return viewResolver;
    } 
    
    /** 
     * Apache-Tiles Init 
     * @return tilesConfigurer 
     */ 
    @Bean public TilesConfigurer getTilesConfigurer() { 
        TilesConfigurer tilesConfigurer = new TilesConfigurer(); 
        tilesConfigurer.setDefinitions(new String[]{ "/WEB-INF/tiles/layouts/tiles.xml" }); 
        tilesConfigurer.setCheckRefresh(true); 
        
        return tilesConfigurer; 
    } 
    
    /** 
     * ViewResolver Configuration 
     * @return viewResolver 
     */ 
    @Bean public InternalResourceViewResolver resolver() { 
        InternalResourceViewResolver resolver = new InternalResourceViewResolver(); 
        resolver.setViewClass(JstlView.class); 
        resolver.setPrefix("/WEB-INF/views/"); 
        resolver.setSuffix(".jsp"); 
        
        return resolver; 
    } 
}
```

## 화면 레이아웃 설정
---

### empty.jsp
---

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%> 
<%@ taglib prefix="tiles" uri="http://tiles.apache.org/tags-tiles" %> 
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %> 

<html> 
    <body> 
        <div> 
            <tiles:insertAttribute name="body" /> 
        </div> 
    <body> 
</html>
```

별다른 내용 없이 tiles 태그와 jstl 태그만 추가해주고  
**&lt;tiles:insertAttribute&gt;** 태그로 메인영역을 잡아주었다.  
(tiles.xml의 put-attribute태그의 name과 동일해야 한다.)

### layout.jsp
---

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%> 
<%@ taglib prefix="tiles" uri="http://tiles.apache.org/tags-tiles" %> 
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %> 

<html> 
    <head> 
        <style> 
            div { 
                  border: 1px solid; 
                  text-align: center; 
            } 
        </style> 
    </head> 
    
    <body> 
        <div id="top" style="height:100px;"> 
            <tiles:insertAttribute name="header" /> 
        </div> 
        
        <div id="body"> 
            <div id="lnb" style="position:absolute; height:300px; width: 350px"> 
                <tiles:insertAttribute name="menu" /> 
            </div> 
            
            <div id="contents" style="margin-left:351px; height:300px;"> 
                <tiles:insertAttribute name="body" /> 
            </div> 
        </div> 
        
        <div id="footer" style="height:100px;"> 
            <tiles:insertAttribute name="footer" /> 
        </div> 
    </body> 
</html>
```

여기서는 실제 구상했던 레이아웃대로 화면을 나누었다.

### header.jsp
---

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%> 

<div> 
    <span style="text-align: center;">
        <h1>Header</h1>
    </span> 
</div>
```

### menu.jsp
---

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%> 

<div> 
    <ul> 
        <li><a href="#">Menu1</a></li> 
        <li><a href="#">Menu2</a></li> 
        <li><a href="#">Menu3</a></li> 
        <li><a href="#">Menu4</a></li> 
    </ul> 
</div>
```

### footer.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%> 

<div> 
    <span style="text-align: center;">
        <h1>Footer</h1>
    </span> 
</div>
```

header, menu, footer에는 그냥 영역 표시만 해두었다.


## 테스트 화면 작성
---

### Login.jsp
---

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %> 

<style> 
    table, td {
                margin: auto; 
                text-align: center; 
    } 
</style> 

<script src="https://code.jquery.com/jquery-3.2.1.min.js"></script> 

<script type="text/javascript"> 
    function fnLogin() { 
        $.ajax({ 
            type : "POST", 
            url : "/chaedae/login/loginProc.json", 
            data : $('#loginForm').serialize(), 
            success : function (response) { 
                if (response == "SUCCESS") { 
                    location.href = "/chaedae/main/main.cd"; 
                } else { 
                    alert(response); 
                } 
            }, 
            error : function (response) { 
                alert(response.responseText); 
            } 
        }); 
    } 
</script> 

<form id="loginForm"> 
    <table> 
        <tr> 
            <td> 
                <h1> Welcome to Chaedae World </h1> 
            </td> 
        </tr> 
        <tr> 
            <td> 
                <label for="userId">ID</label> 
                <input type="text" id="userId" name="userId" title="ID" /> 
            </td> 
        </tr> 
        <tr> 
            <td> 
                <label for="pwd">PW</label> 
                <input type="password" id="pwd" name="pwd" title="패스워드" /> 
            </td> 
        </tr> 
        <tr> 
            <td> 
                <button type="button" style="height: 50px; width:100px" onclick="fnLogin();">Login</button> 
            </td> 
        </tr> 
    </table> 
</form>
```

### Main.jsp
---

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%> 
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %> 

<span>메인 화면 입니다.</span>
```

로그인 화면과 메인화면도 별다른 내용은 없다.. 그저 확인용..

확인을 위한 화면 request를 받아줄 컨트롤러도 생성 해준다.

### LoginController.java
---

```java
package com.chaedae.login; 

import javax.servlet.http.HttpServletResponse; 
import org.springframework.beans.factory.annotation.Autowired; 
import org.springframework.stereotype.Controller; 
import org.springframework.web.bind.annotation.PostMapping; 
import org.springframework.web.bind.annotation.RequestMapping; 
import org.springframework.web.bind.annotation.ResponseBody; 
import com.chaedae.model.User; 
import com.chaedae.service.UserService; 

@Controller 
public class LoginController { 
    @Autowired 
    private UserService userService; 
    
    @RequestMapping(value = "/chaedae/login/login.cd") 
    public String login() { 
        return "/chaedae/login/Login"; 
    } 
    
    @PostMapping(value = "/chaedae/login/loginProc.json") 
    @ResponseBody 
    public void loginProc(User vo, HttpServletResponse response) throws Exception { 
        String rtn = "FAILURE"; 
        
        if (vo != null && vo.getUserId() != null) { 
            User user = userService.selectByUserId(vo); 
            
            if (user != null && user.getPwd().equals(vo.getPwd())) { 
                rtn = "SUCCESS"; 
            } 
        } 
        
        response.getWriter().print(rtn);
    } 
}
```

### MainController.java
---

```java
package com.chaedae.main; 

import org.springframework.stereotype.Controller; 
import org.springframework.web.bind.annotation.GetMapping; 

@Controller 
public class MainController { 
    @GetMapping(value = "/chaedae/main/main.cd") 
    public String main() { 
        return "/chaedae/main/Main"; 
    } 
}
```

여기까지 완료 되었다면 프로젝트 구조는 대략 아래와 같아진다.

![Setting Image1](/assets/img/posts/tistory/20180904-1.png){:width="500px" height="884px"}  

이제 서버를 구동한 뒤 localhost:8080/chaedae/login/Login.cd 로 접속을 해서 확인을 해보자.

![Setting Image2](/assets/img/posts/tistory/20180904-2.png){:width="500px" height="220px"}  

대략 위처럼 아주 못생긴 로그인 화면이 나오고...

로그인을 해보면.. (제 프로젝트로 따라오셨다면 master/a1234 로 접속)

![Setting Image3](/assets/img/posts/tistory/20180904-3.png){:width="500px" height="289px"}  

더 못생긴 위와 같은 화면이 나온다... (다음 포스팅 때는 디자인도 신경을 좀 써야 할 듯..)

---

:: Git Repo ::  
[https://github.com/Chaedae/SpringMVC-tiles](https://github.com/Chaedae/SpringMVC-tiles){:target="_blank"}

---