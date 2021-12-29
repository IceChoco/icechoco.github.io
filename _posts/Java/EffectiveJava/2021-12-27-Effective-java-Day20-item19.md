---
title: '[Effective Java] Day 20 - Item 19 :: 상속을 고려한 설계 주의 사항과 문서화 (2)'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava
comments: true
---

Day20에서는 item 19에 대한 내용을 다룬다.

## Item 19 :: 상속을 고려한 설계 주의 사항과 문서화
### 결론
상속을 고려해 설계하기 전에 아래 checklist를 검토해보라
- **<span style="color:red"> ★ 부모 클래스의 개발자가 세웠던 가정이나 추론 과정을 정확하게 이해하고 있는가</span>**
- 클래스 내부에서 스스로를 어떻게 사용하는지 적절하게 문서화했는가
- protected hook을 결정하기 위해 하위 클래스를 구현해 테스트 해보았는가
- 자식 클래스의 인스턴스가 초기화되기 전에 부모 클래스에서 overridable method를 호출해 자식 클래스 필드에 접근하지 않는가

이런 고려사항에도 불구하고 상속을 꼭 써야만 하겠다면  
method를 override 하더라도 다른 메소드 동작에 영향을 주지 않게 **class 내부에서는 overridable method를 호출하지 않게 만들고 이 사실을 문서화**하길 권한다.  
상속을 써야 한다면 템플릿 메서드 패턴을 쓰는게 좋을 듯 하다.

- SolrRequest example
    - SolrRequest란 엘라스틱서치(Elasticsearch)와 같이 Apache Lucene을 기반으로 serach 리퀘스트를 할 수 있는 검색 엔진
      - Elasticsearch: Apache 재단이 만든 Lucene 기반의 Java 오픈소스 분산 NoSQL 검색엔진
    - SolrRequest의 `process()`는 final로 선언하여 override를 금지하고
      `createResponse(SolrClient)` hook을 `protected abstract`로 선언하여 다형성을 구현하고 있다.
    - **Q. 결론의 4번째 항목인 `자식 클래스의 인스턴스가 초기화되기 전에 부모 클래스에서 overridable method를 호출해 자식 클래스 필드에 접근하지 않는가`라는 항목과 상충되지 않는가?**
      - 부모 클래스 개발자의 의도를 파악하는 것이 중요하기 때문에, 파악이 잘 되었다면 예외가 될 수도 있을 것 같다.
        아래의 예제에서도 createResponse 메서드 내 리턴값이 new로 새로운 인스턴스를 생성하여 반환한다. 따라서 자식 클래스 필드가 초기화 되지 않았더라도 에러가 발생하지 않는다.

```java
public abstract class SolrRequest<T extends SolrResponse> implements Serializable {
    ...
    protected abstract T createResponse(SolrClient var1);

    public final T process(SolrClient client, String collection) throws SolrServerException, IOException {
        long startTime = TimeUnit.MILLISECONDS.convert(System.nanoTime(), TimeUnit.NANOSECONDS);
        T res = this.createResponse(client);
        res.setResponse(client.request(this, collection));
        long endTime = TimeUnit.MILLISECONDS.convert(System.nanoTime(), TimeUnit.NANOSECONDS);
        res.setElapsedTime(endTime - startTime);
        return res;
    }

    ...
}
```

```java
public class QueryRequest extends SolrRequest<QueryResponse> {
    ...
    protected QueryResponse createResponse(SolrClient client) {
        return new QueryResponse(client);
    }
    ...
}
```
```java
public class UpdateRequest extends SolrRequest<QueryResponse> {
    ...
    protected UpdateResponse createResponse(SolrClient client) {
        return new UpdateResponse();
    }
    ...
}
```