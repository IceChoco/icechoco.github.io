---
title: '[우아한 테크캠프 Pro] 우아한 테크캠프 Pro 프리코스 1주차 - TDD로 숫자 야구게임 구현'
layout: post
categories: java
tags: java
comments: true
---

우아한 테크캠프 Pro 프리코스 1주차 과제 완료 후 2차 과제와 함께 1주차 과제인 숫자 야구 게임을 TDD로 구현하는 방법에 대한 동영상 강의가 제공되었다.
이 포스팅에서는 해당 강의에 대한 내용을 정리하고자 한다.

## Test Driven Development(테스트 주도 개발, TDD)
### 용어 정리
- Prodiction code: 실제 배포하는 서비스 코드
- Test code: production code에 대한 테스트를 구현하는 코드

### TDD란?
- TDD = TFD(Test First Development) + 리팩토링
- 단위테스트와 TDD를 같다고 생각하는 사람이 있으나, Nope! 이 둘은 엄연히 다르다.
- 단위테스트 : Production Code 구현 → 단위 Test Code 구현
- TDD: Production Code를 추가하기 전 단위 Test Code를 먼저 추가하는 것
- 그럼 그냥 TFD(Test First Development)로 말하면되는데 왜 TDD라고 할까?
- 이유: TDD의 과정에 리팩토링이 포함되어 있기 때문
- TDD는 리팩토링을 한 번에 하는 것이 아닌 작은 단위로 쪼개서 하는 것이다.
- ex) 기능이 추가 될 때 마다 설계를 계속 변경하는 것
- **리팩토링은 TDD 사이클에서 상당히 중요하다.**
> **리팩토링이란?**
> 리팩토링은 설계의 활동이다.
> 기능에 대한 변경은 없으면서 클래스의 구조, 메서드 분리

- TDD란 프로그래밍 의사결정과 피드백 사이의 간극을 의식하고 이를 제어하는 기술이다. - 켄트백, Test Driven Development by Example 중
- TDD의 아이러니 중 하나는 **테스트** 기술이 아니라는 점이다. TDD는 **분석** 기술이며, **설계** 기술이기도 하다. - 켄트벡, Test Driven Development by Example 중

TDD를 잘하기 위해서는 요구사항 분석을 잘하여 To Do List를 작성해놔야 한다.

### TDD를 하는 이유
- 디버깅 시간을 줄여준다.
- 동작하는 문서 역할을 한다.
- 변화에 대한 두려움을 줄여준다.

### TDD 사이클
아래 싸이클을 유지하면서 프로그래밍 하는 것을 TDD 사이클이라 한다.
1. Test fails: 실패하는 Test를 구현한다.
2. Test passes: 테스트가 성공하도록 Production Code를 구현한다. 즉, 실패하는 테스트를 최대한 빠르게 패스한다.
3. Refactoring: Production Code와 Test Code를 리팩토링 한다.

### TDD 원칙
1. 실패하는 단위 테스트를 작성할 때까지 Production code를 작성하지 않는다.
2. 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위테스트를 작성한다.
3. 현재 실패하는 Test를 통과할 정도로만 실제 코드를 작성한다.

즉, 너무 많은 부분을 예측해서 개발하지 말고 현재 TestCase를 만족할 수 있는 수준으로만 Proction code를 작성해라.

## TDD로 숫자 야구게임 구현
### 기능 요구사항
- 기본적으로 1부터 9까지 서로 다른 수로 이루어진 3자리의 수를 맞추는 게임이다.
- 스트라이크: 같은 수가 같은 자리에 있는 경우
- 볼: 같은 수가 다른 자리에 있는 경우
- 낫싱: 같은 수가 전혀 없음

### 1단계 - Util 성격의 기능이 TDD로 도전하기 좋음
 - 사용자로부터 입력받는 3개의 숫자 예외처리
   - 1~9의 숫자인가?
   - 중복 값이 있는가?
   - 3자리인가?  
   
1.Test fails: 실패하는 Test를 구현한다.

```java
public class ValidationUtilsTest {
    @Test
    @DisplayName("야구 숫자 1부터 9까지 검증")
    void Baseball_Number_1_9_Verification() {
        boolean result = VlidationUtils.validNo(9);
        assertThat(result).isTrue();
    }
}
```
2.Test passes: 테스트가 성공하도록 Production Code를 구현한다. 즉, 실패하는 테스트를 최대한 빠르게 패스한다.

```java
public class ValidationUtilsTest {
    @Test
    @DisplayName("야구 숫자 1부터 9까지 검증")
    void Baseball_Number_1_9_Verification() {
        boolean result = VlidationUtils.validNo(9);
        assertThat(result).isTrue();
    }
}

public class VlidationUtils {
    public static boolean validNo(int i) {
        if(i<=9){
            return true;
        }
        return false;
    }
}
```
3.Refactoring: Production Code와 Test Code를 리팩토링 한다.
- 1차 리펙토링

```java
public class ValidationUtilsTest {
    @Test
    @DisplayName("야구 숫자 1부터 9까지 검증")
    void Baseball_Number_1_9_Verification() {
        assertThat(VlidationUtils.validNo(9)).isTrue();
    }
}

public class VlidationUtils {
    public static boolean validNo(int no) {
        if(no<=9){
            return true;
        }
        return false;
    }
}
```
- 2차 리펙토링

```java
public class ValidationUtilsTest {
    @Test
    @DisplayName("야구 숫자 1부터 9까지 검증")
    void Baseball_Number_1_9_Verification() {
        assertThat(VlidationUtils.validNo(9)).isTrue();
        assertThat(VlidationUtils.validNo(1)).isTrue();
        assertThat(VlidationUtils.validNo(0)).isFalse();
        assertThat(VlidationUtils.validNo(10)).isFalse();
    }
}

public class VlidationUtils {
    public static final int MIN_NO = 1;
    public static final int MAX_NO = 9;

    public static boolean validNo(int no) {
        return no >= MIN_NO && no<= MAX_NO;
    }
}
```

### 2단계 - 테스트 가능한 부분에 대해 TDD로 도전
- 스트라이크: 같은 수가 같은 자리에 있는 경우
- 볼: 같은 수가 다른 자리에 있는 경우
- 낫싱: 같은 수가 전혀 없음

테스팅이 가능한 부분 → 랜덤, UI, 날짜 데이터가 있으면 Test가 하기 힘든데 그런것들이 없는 것을 말함.

#### 첫번쨰 방법
---
com / user  
123, 456 → nothing  
123, 245 → 1 ball  
123, 145 → 1 strike  

PlayResult result = play(Arrays.asList(1,2,3), Arrays.asList(4,5,6))

---
위치값을 모르고 각 수만 아는 경우 ball인지 strike인지 알 수가 없다. 따라서 위치값을 넣어줘야한다.
아래에서는 '위치값, 숫자'와 같은 형태로 표현했다. TDD를 할 때는 위와 같은 형태보다 작은 형태로 쪼갠 아래와 같은 형태가 더 쉽다.

com / user  
1 1 , 1 1 → 1 strike  
1 4 , 2 4 → 1 ball  
1 4 , 2 5 → nothing  

<span style="color:red">문제를 작은 단위로 쪼갠 뒤 TDD 구현을 도전하라</span>

#### 난이도를 높여서, 두 번쨰 방법
com / user  
123 / 1 4 → nothing  
123 / 1 2 → ball  
123 / 1 1 → strike  

도메인 로직에서 객체에 접근해 Getter를 이용하여 데이터를 가져와 비교하지 말고, <span style="color:red">객체 내에 판단 로직을 만들어 객체가 일하게 만들어라.</span>