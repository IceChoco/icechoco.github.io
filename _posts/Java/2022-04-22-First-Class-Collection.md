---
title: '[Java] 일급 컬렉션 (First Class Collection)이란?'
layout: post
categories: java
tags: java
comments: true
---

우아한 테크캠프 Pro 프리코스 2주차 과제에서는 1주차 과제에서 추가로 **일급 콜렉션**을 사용하라는 프로그래밍 요구사항이 추가되었다.
일급 컬렉션이란 무엇인지, 어떤 장점들이 있기에 객체지향적이고 리팩토링하기 쉬운 코드를 짜기위해 필요한지 알아보려한다.

- <span style="color:grey">*향로님이 올려주신 [일급 컬렉션 (First Class Collection)의 소개와 써야할 이유](https://jojoldu.tistory.com/412)를 보고 정리한 글입니다.* </span>

우선 일급 컬렉션이라는 단어는 소트웍스 앤솔로지의 객체지향 생활체조 파트에서 언급이 되었다.
>규칙 8: 일급 콜렉션 사용
이 규칙의 적용은 간단하다.
콜렉션을 포함한 클래스는 반드시 다른 멤버 변수가 없어야 한다.
각 콜렉션은 그 자체로 포장돼 있으므로 이제 콜렉션과 관련된 동작은 근거지가 마련된셈이다.
필터가 이 새 클래스의 일부가 됨을 알 수 있다.
필터는 또한 스스로 함수 객체가 될 수 있다.
또한 새 클래스는 두 그룹을 같이 묶는다든가 그룹의 각 원소에 규칙을 적용하는 등의 동작을 처리할 수 있다.
이는 인스턴스 변수에 대한 규칙의 확실한 확장이지만 그 자체를 위해서도 중요하다.
콜렉션은 실로 매우 유용한 원시 타입이다.
많은 동작이 있지만 후임 프로그래머나 유지보수 담당자에 의미적 의도나 단초는 거의 없다. - 소트웍스 앤솔로지 객체지향 생활체조편
>

설명을 보기 전까지는 무슨 의미인지 알겠다. 그런데 소스코드를 봐야 좀 더 확실하게 알 수 있을듯하다. 아래의 코드를 보자.
```
Map<String, String> map = new HashMap<>();
map.put("1", "A");
map.put("2", "B");
map.put("3", "C");
```
위와 같은 코드를 아래와 같이 **Wrapping**할 수 있다.

```java
public class GameRanking {

    private Map<String, String> ranks;
    
    public GameRanking(Map<String, String> ranks) {
        this.ranks = ranks;
    }
}
```

**Collection을 Wrapping**하면서, **그 외 다른 멤버 변수가 없는 상태**를 일급 컬렉션이라 한다.
Wrapping하면서 얻게되는 이점들은 크게 4가지가 있다.

1. **비즈니스**에 종속적인 **자료구조**
2. Collection의 **불변성**을 보장
3. 상태와 행위를 한 곳에서 관리
4. 이름이 있는 컬렉션

## 1. 비즈니스에 종속적인 자료구조
예를 들어 다음과 같은 조건으로 **로또 복권 게임**을 만든다고 가정해보자.
로또복권은 다음과 같은 조건이 있다.
- 6개의 번호가 존재
- 보너스 번호는 이번 예제에서 제외
- 6개의 번호를 서로 중복되지 않아야 함

일반적으로 이런 일은 **서비스 메소드**에서 진행한다.
그래서 구현을 해보면 아래처럼 된다.

```java
public class LottoService {
    private static final int LOTTO_NUMBERS_SIZE = 6;
    
    public void createLottoNumber(){
        List<Long> lottotNumbers = createNonDuplicateNumbers();
        validateSize(lottotNumbers);
        validateDuplicate(lottotNumbers);
        
        //이후 로직 쭉쭉 실행
    }
    
    private List<Long> createNonDuplicateNumbers() {
     return Arrays.asList(new Long(1));
    }
    
    private void validateDuplicate(List<Long> lottotNumbers) {
        if(lottotNumbers.size() != LOTTO_NUMBERS_SIZE){
           throw new IllegalArgumentException("로또 번호는 6개만 가능합니다.");
        }
    }
    
    private void validateSize(List<Long> lottotNumbers) {
        Set<Long> nonDuplicateNumbers = new HashSet<>(lottotNumbers);
        if(nonDuplicateNumbers.size()!=LOTTO_NUMBERS_SIZE){
          throw new IllegalArgumentException("로또 번호는 중복될 수 없습니다.");
        }
    }
}
```

서비스 메소드에서 비즈니스 로직을 처리했다. 이럴경우 큰 문제가 있다.
**로또 번호가 필요한 모든 장소에선 검증로직**이 들어가야만 한다.
- `List<Long>`으로 된 데이터는 모두 검증로직이 필요할까 ?
- 신규 입사자들의 경우 이 검증 로직이 필요한지 알 수 있을까?

등등 **모든 코드와 도메인 로직**을 알고 있지 않다면 언제든 문제가 발생할 소지가 있다.
그렇다면 이 문제를 해결하기 위해서는

- 6개의 숫자로만 이루어져야하고
- 6개의 숫자는 서로 중복되지 않아야만 하는

이런 자료구조가 없을까? 없으니 만들면된다!
아래와 같이 **해당 조건으로만 생성 할 수 있는 자료구조**를 만들면 위에서 언급한 문제들이 모두 해결된다.
그리고 이런 클래스를 **일급 컬렉션**이라고 부른다.

```java
public class LottoTicket {
    private static final int LOTTO_NUMBERS_SIZE = 6;
    
    private final List<Long> lottoNumbers;
    
    //6개 / 중복되지 않은 숫자들만 가능한 자료구조
    public LottoTicket(List<Long> lottoNumbers) {
        validateSize(lottoNumbers);
        validateDuplicate(lottoNumbers);
        this.lottoNumbers = lottoNumbers;
    }
    
    private void validateDuplicate(List<Long> lottotNumbers) {
        if(lottotNumbers.size() != LOTTO_NUMBERS_SIZE){
            throw new IllegalArgumentException("로또 번호는 6개만 가능합니다.");
        }
    }
    
    private void validateSize(List<Long> lottotNumbers) {
        Set<Long> nonDuplicateNumbers = new HashSet<>(lottotNumbers);
        if(nonDuplicateNumbers.size()!=LOTTO_NUMBERS_SIZE){
            throw new IllegalArgumentException("로또 번호는 중복될 수 없습니다.");
        }
    }
}
```

이제 로또 번호가 필요한 모든 로즉인 이 일급 컬렉션만 있으면 된다.

```java
public class LottoService2 {
    public void createLottoNumber(){
        LottoTicket lottoTicket = new LottoTicket(createNonDuplicateNumbers());
    
    //이후 로직 쭉쭉 실행
    }

    private List<Long> createNonDuplicateNumbers() {
        return Arrays.asList(new Long(1));//로직 구현하기
    }
}
```

**비즈니스에 종속적인 자료구조**가 만들어져, 이후 발생할 문제가 최소화 되었다.

## 2. 불변
일급 컬렉션은 **컬렉션의 불변을 보장**한다.
불변을 보장하기 위해 final을 사용하면 되는거 아니냐 생각 할 수 있지만 아니다.
자바의 final은 정확히는 불변을 만들어주는게 아니라, **재할당만 금지**한다.

아래의 테스트 코드를 참고해보자.

```java
public class finalTest {
    @Test
    public void final도_값변경이_가능하다(){
        final Map<String, Boolean> collection = new HashMap<>();
        
        collection.put("1",true);
        collection.put("2",true);
        collection.put("3",true);
        collection.put("4",true);
        
        assertThat(collection.size()).isEqualTo(4);
    }
}
```

이를 실행해보면 collection에 **값이 추가**되는걸 확인할 수 있다.
이미 collection은 **비어 있는 HashMap**으로 선언되었음에도 값이 변경될 수 있다는 것이다.
추가로 테스트해보자.

```java
public class finalTest {
    @Test
    public void final은_재할당이_불가능하다(){
        final Map<String, Boolean> collection = new HashMap<>();
        
        collection = new HashMap<>();
        
        assertThat(collection.size()).isEqualTo(4);
    }
}
```

이 코드는 `collection = new HashMap<>();`에서 바로 컴파일 에러가 발생한다. **final로 할당된 코드에 재할당 될 수 없기 때문이다.**

위와 같이 Java의 final은 **재할당만 금지**한다.
이외에도 `member.setAge(10)`와 같은 코드 역시 작동해버리니 반쪽자리라 할 수 있다.

요즘과 같이 SW 규모가 커지고 있는 상황에서 **불변 객체**는 아주 중요하다.
각각의 객체들이 **절대 값이 바뀔 일이 없다**는게 보장되면 그만큼 코드를 이해하고 수정하는 **사이드 이펙트가 최소화**되기 때문이다.

Java에서는 final로 그 문제를 해결할 수 없기 때문에 **일급 컬렉션**(First Class Collection)과 **래퍼 클래스**(Wrapper class) 등의 방법으로 해결해야만 한다.

그래서 아래와 같이 **컬렉션의 값을 변경할 수 있는 메소드가 없는 컬렉션**을 만들면 **불변 컬렉션**이 된다.

```java
public class Orders {
    private final List<Order> orders;
    
    public Orders(List<Order> orders) {
        this.orders = orders;
    }
    
    public long getAmountSum(){
        return orders.stream()
        .mapToLong(Order::getAmount)
        .sum();
    }
}
```

이 클래스는 **생성자와 getAmountSum()**외에 다른 메소드가 없다.
즉, 이 클래스의 사용법은 **새로 만들거나 값을 가져오는 것**뿐이다.
List라는 컬렉션에 접근할 수 있는 방법이 없기 때문에 **값을 변경/추가가 안된다.**

이렇게 일급 컬렉션을 사용하면, 불변 컬렉션을 만들 수 있다.

## 3. 상태와 행위를 한곳에서 관리
일급 컬렉션의 세번쨰 장점은 **값과 로직이 함께 존재**한다는 것이다.
> 이 부분은 Enum의 장점과도 일맥상통한다.

예를들어 여러 Pay들이 모여있고, 이 중 **NaverPay 금액의 합**이 필요하다고 가정해보겠다.
일반적으로는 아래와 같이 작성한다.

```java
public class PayTest {
  @Test
  public void 로직이_밖에_있는_상태(){
    //값은 여기 있는데
    List<Pay> pays = Arrays.asList(
            new Pay(NAVER_PAY, 1000),
            new Pay(NAVER_PAY, 1500),
            new Pay(KAKAO_PAY, 2000),
            new Pay(TOSS, 3000L)
    );

    //계산은 여기서
    Long naverPaySum = pays.stream()
            .filter(pay -> pay.getPayType().equals(NAVER_PAY))
            .mapToLong(Pay::getAmount)
            .sum();

    assertThat(naverPaySum).isEqualTo(2500);
  }
}
```
- List에 데이터를 담고
- Service 혹은 Util 클래스에서 필요한 로직 수행

이 상황에서는 문제가 있다.  
결국 `pays`라는 컬렉션과 계산 로직은 **서로 관계**가 있는데, 이를 **코드로 표현이 안된다.**  

Pay 타입의 **상태에 따라** 지정된 메소드에서만 계산되길 월하는데, 현재 상태로는 강제할 수 있는 수단이 없다.  
지금은 **Pay 타입의 List라면** 사용될 수 있기 때문에 히스토리를 모른다면 실수할 여지가 많다.
- 똑같은 기능을 하는 **메소드를 중복 생성**할 수 있다.
  - 히스토리가 관리 안된 상태에서 신규 화면이 추가되어야 할 경우 계산 메소드가 있다는 것을 몰라 다시 만드는 경우가 빈번하다.
  - 만약 기존 화면의 계싼 로직이 변경될 경우, 신규 인력은 2개의 메소드 로직을 다 변경해야하는지, 해당 화면만 변경해야 하는지 알 수 없다.
  - 관리 포인트가 증가할 확률이 매우 높다.
- 계산 메소드를 누락할 수 있다.
  - 리턴 받고자 하는 것이 Long 타입이기 때문에 **꼭 이 계산식을 써야한다고 강제할 수 없다.** 

결국에 **네이버페이 총 금액을 뽑을려면 이렇게 해야한다는 계산식을 컬렉션과 함께 두어야** 한다.  
만약 네이버페이 외에 카카오 페이의 총금액도 필요하다면 더더욱 **코드가 흩어질 확률이 높다.**

그래서 이 문제를 일급 컬렉션으로 해결할 수 있다.

```java
public class PayGroup {
    private List<Pay> pays;

    public PayGroup(List<Pay> pays) {
        this.pays = pays;
    }

    public Long getNaverPaySum(){
        return pays.stream()
                .filter(pay -> PayType.isNaverPay(pay.getPayType()))
                .mapToLong(Pay::getAmount)
                .sum();
    }
}
```

만약 **다른 결제 수단들의 합이 필요**하다면 아래와 같이 **람다식으로 리팩토링** 가능하다.

```java
public class PayGroups {
    private List<Pay> pays;

    public PayGroups(List<Pay> pays) {
        this.pays = pays;
    }

    public Long getNaverPaySum(){
        return getFilteredPays(pay -> PayType.isNaverPay(pay.getPayType()));
    }

    public Long getKakaoPaySum(){
        return getFilteredPays(pay -> PayType.isKakaoPay(pay.getPayType()));
    }

    private Long getFilteredPays(Predicate<Pay> predicate){
        return pays.stream()
                .filter(pay -> PayType.isNaverPay(pay.getPayType()))
                .mapToLong(Pay::getAmount)
                .sum();
    }
}
```

이렇게 PayGroup라는 일급 컬렉션이 생김으로 **상태와 로직이 한곳에서 관리**된다.

```java
public class PayTest {
    @Test
    public void 로직이_밖에_있는_상태(){
        //값은 여기 있는데
        List<Pay> pays = Arrays.asList(
                new Pay(NAVER_PAY, 1000),
                new Pay(NAVER_PAY, 1500),
                new Pay(KAKAO_PAY, 2000),
                new Pay(TOSS, 3000L)
        );

        //상태와 로직이 한곳에!
        PayGroups payGroups = new PayGroups(pays);
        Long naverPaySum = payGroups.getNaverPaySum();

        assertThat(naverPaySum).isEqualTo(2500);
    }
}
```

## 4. 이름이 있는 컬렉션
마지막 장점은 **컬렉션에 이름을 붙일 수 있다**는 것이다.  
**같은 Pay들의 모임이지만 네이버페이의 List와 카카오페이의 List는 다르다.**  
그렇다면 이 둘을 구분하려면 어떻게 해야할까?  
가장 흔한 방법은 변수명을 다르게 하는 것이다.

```java
@Test
public void 컬렉션을_변수명으로(){
    List<Pay> naverPays = createNaverPays();
    List<Pay> kakakoPays = createKakaoPays();
}
```

위와 같이 변수명으로 선언했을 떄의 단점은 뭘까?  
- 검색이 어렵다
  - 네이버 페이 그룹이 어떻게 사용되는지 검색 시 **변수명으로만 검색할 수 있다.**
  - 이 상황에서 검색은 거의 불가능하다.
  - **네이버 페이 그룹**이라는 뜻은 **개발자마다 다르게 지을 수 있기 때문**이다.
  - 네이버 페이 그룹은 어떤 검색어로 검색이 가능할까....?
- 명확한 표현이 불가능하다.
  - 변수명에 불과하기 때문에 **의미를 부여하기 어렵다.**
  - 이는 개발팀/운영팀간에 의사소통 시 보편적인 언어로 사용하기가 어려움을 이야기한다.
  - 중요한 값임에도 **이를 표현할 명확한 단어가 없는 것**이다.

위 문제 역시 일급 컬렉션으로 쉽게 해결할 수 있다.  
네이버 페이 그룹과 카카오 페이 그룹 각각의 일급 컬렉션을 만들면 **이 컬렉션 기반으로 용어사용과 검색**을 하면 된다.

```java
    @Test
    public void 컬렉션을_변수명으로(){
        List<Pay> naverPays = createNaverPays();
        List<Pay> kakakoPays = createKakaoPays();
    }
```

개발팀/운영팀 내에서 사용될 표현은 이제 이 컬렉션에 맞추면 된다.  
검색 역시 이 컬렉션 클래스를 검색하면 모든 사용 코드를 찾아낼 수 있다.

## 마무리
처음에 소트웍스 앤솔로지의 객체지향 생활체조 파트에서 언급된 일급 컬렉션에 대한 설명은 잘 이해가 안갔다.  
코드를 직접 보지 않으면 이해가 잘 안가는 스타일이라 그런가보다.  
그런데 다행이도 이렇게 너무 자세하게 남겨주신 향로님께 감사의 말씀을 드리고 싶다.  
올려주신 글을 보며 같이 타이핑해가면서, 소스 코드를 구현하면서 보니 이해가 더 잘된다.  
이제는 왜 일급 컬렉션을 왜 써야하는지 공감이 간다! :D

### 참고
- [소트웍스 앤솔러지 - 규칙 8: 일급 콜렉션 사용](https://developerfarm.wordpress.com/2012/02/01/object_calisthenics_/)
- [일급 컬렉션 (First Class Collection)의 소개와 써야할 이유](https://jojoldu.tistory.com/412)