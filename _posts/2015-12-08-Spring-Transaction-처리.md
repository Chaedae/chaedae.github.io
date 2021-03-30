---
title: Spring Transaction 처리
author: ChaeDae
date: 2015-12-02 15:03:00 +0800
categories: [Java, Spring]
tags: [JAVA, Spring, Transaction]
---

SPRING + MAVEN + MyBatis로 되어 있는 프레임워크로 프로젝트를 진행하던 중 트랜잭션 관련된 이슈가 생겨서 글로 남겨둔다.


스프링자체에 `@Transactional` 이라는 어노테이션이 있길래 당연히 이걸 쓰면 되는줄 알고 썼다가.. 큰일날뻔 했다..

**요놈이 글쎄 트랜잭션 처리를 안해줌..!!!**

로직 자체가 한 메소드에서 여러 테이블에 저장을 해야 하는 로직인데,
autoCommit이 되버려서 데이터가 꼬여버리는 것이었다..

결국 구글신님께 도움을 요청. 몇가지 원인이 될만한 것들을 찾았다.

## 원인
---
1. 프로젝트에서 쓰는 MySQL DB내의 테이블 타입이 InnoDB가 아니면 문제가 될 수 있다.<br/>
 -> 타입은 정확히 InnoDB로 되어 있었다..
  
2. Database-config.xml 에서 트랜잭션 옵션 설정(defaultAutoCommit)이 true로 되어 있을 것이다.<br/>
 -> 일단 내가 config쪽을 손댈 수도 없는 입장이라 이부분은 패스했다.. 프로젝트 전반에 걸쳐 설정을 해놓은것을 한두개의 모듈만 만드는 내가 감놔라 배놔라 할 위치는 아니기때문..<br/>
_(그리고 건의는 해보았으나 연락이 없어서 따로 찾는게 더 빠를것 같았다..)_

찾다 찾다 도저히 못찾겠어서 MyBatis고 나발이고 JDBC를 써뻐릴까 하다가.. 결국 찾아냈다. (역시 의지의 한국인)
  
DataSourceTransactionManager 이라는 녀석을 사용하면 트랜잭션 처리가 가능하단다. 

소스로 만나보자..
 
## JAVA Source
---

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.datasource.DataSourceTransactionManager; 
import org.springframework.stereotype.Service; 
import org.springframework.transaction.TransactionDefinition; 
import org.springframework.transaction.TransactionStatus; 
import org.springframework.transaction.support.DefaultTransactionDefinition; 

@Service public class TranSaveService() { 
    @Autowired private DataSourceTransactionManager transactionManager; 
    
    public boolean tranSave(String param) { 
        // 파라미터값이 없을 경우 리턴 
        if (param == null || param.trim.length() == 0) return false; 
        
        // 하나의 트랜잭션으로 DB 저장 처리를 하기 위해 트랜잭션 선언 
        DefaultTransactionDefinition def = new DefaultTransactionDefinition(); 
        def.setName("example-transaction"); 
        def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED); 
        
        TransactionStatus status = transactionManager.getTransaction(def); 
        
        boolean chk = false; 
        
        try { 
            // 1. 테이블 조회 
            Sample sample = sampleDao.getSample(param); 
            
            // 2. Sample 테이블 정보를 이용해서 데이터 저장 
            sampleDao.addTemp1(sample); 
            
            // 3. 데이터 저장2 
            sampleDao.addTemp2(sample); 
            
            // 4. 데이터 저장3 
            String temp = sampleDao.addTemp3(sample); 
            
            // 5. 샘플 데이터 업데이트 
            sampleDao.updateSample(temp); 
            
            // 오류가 없었을 경우 Commit 처리 
            transactionManager.commit(status); 
            
            chk = true; 
        } catch(Exception e) { 
            e.printStackTrace(); 
            // Exception 발생 시 롤백 
            transactionManager.rollback(status); 
        } 
        return chk; 
    } 
}
```
  

일단 전역변수로 `DataSourceTransactionManager` 를 선언하고

사용할 메소드내에서 `DefaultTransactionDefinition`과 `TransactionStatus`를 이용해서 트랜잭션 처리를 한다.

이렇게 사용하면 기존 JDBC처럼 내가 원하는 부분에서 별도로 commit이나 rollback을 수행할 수 있다.


좋은점은 내부적으로 자원해제 인터셉터를 사용하기 때문에 따로 close()를 안해줘도 된다는 점이다.
  

혹여 나처럼 @Transactional에 데인 분이 있다면 위 소스를 참조해서 사용하면 될 것이다.