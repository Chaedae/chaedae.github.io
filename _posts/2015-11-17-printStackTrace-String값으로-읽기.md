---
title: printStackTrace String값으로 읽기
author: ChaeDae
date: 2015-11-17 19:52:00 +0800
categories: [Java, Log]
tags: [Java, Log, Exception]
---

로그 파일을 분리해서 로그를 남기기는 했는데..

Exception에서 제공해주는 getMessage()를 사용해서 로그를 남겼더니 정보가 너무 부족했다..

printStackTrace를 그대로 뿌려주면 안되나 했는데 될리가 있나..

방법이 없어 구글링 시작..  
  

그래서 찾은 해법..

일단 지난번 했던 [로그파일분리](/posts/Log4j-로그파일-분리){:target="_blank"} 에서 사용했던 메소드를 재활용해보자..

```java
// Properties load 
public void setProperties() {
    PropertyConfigurator.configure(this.getClass().getResource("/").getPath() + logPath); 

    // Logger Set 
    debug_logger = Logger.getLogger("변수명1"); 
    error_logger = Logger.getLogger("변수명2"); 

    // Message Set 
    String file_path = this.getClass().getResource("/").getPath() + msgPath;
    FileInputStream fis = null;
    prop = new Properties(); 

    // get properties content 
    try { 
        fis = new FileInputStream(new File(file_path)); 
        prop.load(new InputStreamReader(fis)); 
    } catch(IOException e) { 
        //error_logger.error("Properties Load fail"); 
        error_logger.error(this.getPrintStackTrace(e)); 
    } finally { 
        try { 
            fis.close(); 
        } catch (IOException e) {
        } 
    } 
} 

/** 
 * Exception PrintStackTrace to String 
 * @param e Exception 
 * @return printStackTrace(String) 
 */ 
public static String getPrintStackTrace(Exception e) { 
    ByteArrayOutputStream out = new ByteArrayOutputStream(); 
    PrintStream stream = new PrintStream(out); 
    e.printStackTrace(stream); 

    return out.toString(); 
}
```

```java
 //error_logger.error("Properties Load fail");
 error_logger.error(this.getPrintStackTrace(e));
```

기존에는 그냥 하드코딩으로 메시지를 박아넣었다면,  
이번에는 printstackTrace에 있는 내용을 그대로 뿌려주었다.  
  
  
익셉션에서 출력되는 printstackTrace의 내용들을 ByteArrayOutputStream과 PrintStream을 이용해서 byte단위로 쪼개 String으로 리턴해준다.  
  
  
막연한 로그보다는 이렇게 에러난 부분이 어딘질 정확히 명시하는 로그가 사후관리에 더욱 효과적이다.  
  
  
특히 개발자 입장에선 차라리 이런 에러가 소스코드 찾기도 편리하다