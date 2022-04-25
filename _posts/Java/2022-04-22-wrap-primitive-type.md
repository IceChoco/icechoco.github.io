---
title: '[Java] 원시값과 문자열의 포장'
layout: post
categories: java
tags: java OOP OBJECT-CALISTHENIC
comments: true
---

우아한 테크캠프 Pro 프리코스 2주차 과제에서는 1주차 과제에서 추가로 **모든 원시값과 문자열을 포장하라**라는 프로그래밍 요구사항이 추가되었다.
- <span style="color:grey">*2기_오렌지님이 올려주신 [원시 타입을 포장해야 하는 이유](https://tecoble.techcourse.co.kr/post/2020-05-29-wrap-primitive-type/)를 보고 정리한 글입니다.* </span>

우선 소트웍스 앤솔로지의 객체지향 생활체조 파트에서 언급된 내용은 아래와 같다.
>규칙 3: 원시값과 문자열의 포장
>int 값 하나 자체는 그냥 아무 의미 없는 스칼라 값일 뿐이다.  
>어떤 메서드가 int 값을 매개변수로 받는다면 그 메서드 이름은 해당 매개변수의 의도를 나타내기 위해 모든 수단과 방법을 가리지 않아야 한다.
>만약 똑같은 메서드가 시간을 매개변수로 받을 경우, 무슨 일이 생기는지는 훨씬 자명하다.  
>이런 작은 객체가 프로그램의 유지보수성을 높일 수 있는 것은 시간 값을 매개변수로 받는 메서드에게 연도 값을 넘길 수는 없기 때문이다.  
>원시형 변수로는 컴파일러가 의미적으로 맞는 프로그램 작성을 안내할 수 없다.  
>객체로라면 아주 사소하더라도 컴파일러와 프로그래머에게 그 값이 어떤 값이며, 왜 쓰고 있는지에 대한 정보를 전하는 셈이다.
>
>또한 시간이나 돈과 같은 작은 객체는 행위를 놓을 분명한 곳을 마련해 줘서, 그렇지 않았다면 다른 클래스의 주위를 겉돌았을지도 모르는 사태를 방지한다.
> 이는 특히 게터와 세터에 관련된 규칙을 적용하고 그런 작은 객체만이 값에 접근할 수 있을 때 그렇다.

변수를 선언하는 방법에는 두 가지가 있다.
 - 원시타입의 변수를 선언하는 방법
 - 원시 타입의 변수를 객체로 포장한 변수를 선언하는 방법
   (Collection으로 선언한 변수도 포장한다. 이를 일급 컬렉션이라 하며 본 블로그의 `[Java] 일급 컬렉션 (First Class Collection)이란?`글을 참고하길 바란다.)
```java
int age = 20;
Age age = new Age(20);
```

이번 글에서는 객체지향 생활 체조에서도 언급된 **원시 타입의 값을 객체로 포장하면 얻을 수 있는 이점**들에 대해 보겠다.

## 자신의 상태 객체를 스스로 관리할 수 있다.
User라는 클래스에서, 사용자의 나이를 가지고 있다고 가정해보자.
```java
public class User {
    private int age;

    public User(int age) {
        this.age = age;
    }
}
```

위의 형태처럼 원시타입인 int로 나이를 가지고 있으면 어떻게 될까? 쉽게 생각해보면 우선, 아래의 소스코드 처럼 나이에 관한 유효성 검사를 User 클래스에서 하게 된다.

```java
public class User {
    private int age;

    public User(String input) {
        int age = Integer.parseInt(input);
        if(age<0){
            throw new RuntimeException("나이는 0살부터 시작합니다.");
        }
        this.age = age;
    }
}
```

위 예시는 User 클래스의 멤버변수가 나이밖에 없어 문제를 크게 못느낄 수도 있다. 사용자의 이름, 이메일 등 추가적인 값들을 관리하게 된다면 문제가 생길 수 밖에 없다.  
두 글자 이상의 이름만을 지원한다고 가정하고, 이름 변수를 추가해보자 !

```java
public class User {
    private String name;
    private int age;

    public User(String nameValue, String ageValue) {
        int age = Integer.parseInt(ageValue);
        validateAge(age);
        validateName(nameValue);
        this.name = nameValue;
        this.age = age;
    }

    private void validateName(String name){
        if(name.length()<2){
            throw new RuntimeException("이름은 두 글자 이상이어야 합니다.");
        }
    }

    private void validateAge(int age){
        if(age<0){
            throw new RuntimeException("나이는 0살부터 시작합니다.");
        }
    }
}
```

두개의 멤버변수를 선언했을 뿐인데 User 클래스가 할 일이 늘어나 버렸다. **이름** 값에 대한 상태 관리, **나이** 값에 대한 상태 관리를 모두 해야한다.  
User 클래스는 분명히
> 아, 나는 사용자 그 자체 상태만 관리하고 싶은데 왜 자잘 자잘 한 것 까지 내가 관리해야돼? 이건 불합리해!

라고 생가하지 않을까?  
그럼, 원시타입 변수를 포장해보자

```java
public class User {
   private Name name;
   private Age age;

   public User(String nameValue, String ageValue) {
      this.name = new Name(nameValue);
      this.age = new Age(ageValue);
   }
}

public class Name {
   private String name;

   public Name(String nameValue) {
      if(name.length()<2){
         throw new RuntimeException("이름은 두 글자 이상이어야 합니다.");
      }
      this.name = name;
   }
}

public class Age {
    private int age;

    public Age(String ageValue) {
        int age = Integer.parseInt(ageValue);
        if(age<0){
            throw new RuntimeException("나이는 0살부터 시작합니다.");
        }
        this.age = age;
    }
}
```

User 클래스가 해방됐다.
> 와! 나 이제 예외 처리 안 해도 돼!

이름과 나이 값이 각각의 Name, Age가 담당하도록 바뀌었다.  
유효성 검증을 비롯한 이름, 나이 값에 대한 상태값을 User에 넘기지 않고 스스로 관리할 수 있게 되었다. 책임이 명확해졌다.

## 코드의 유지보수에 도움이 된다.
이번엔 다른 예시이다. 다음은 로또 미션의 LottoNumber, Lotto, WinningLotto 클래스의 일부이다.

```java
public class LottoNumber {
    private final static int MIN_LOTTO_NUMBER = 1;
    private final static int MAX_LOTTO_NUMBER = 45;
    private final static String OUT_OF_RANGE = "로또번호는 1~45의 범위입니다.";
    private final static Map<Integer, LottoNumber> NUMBERS = new HashMap<>();

    private int lottoNumber;

    static {
        for(int i=MIN_LOTTO_NUMBER; i<MIN_LOTTO_NUMBER +1;i++){
            NUMBERS.put(i, new LottoNumber(i));
        }
    }

    public LottoNumber(int lottoNumber) {
        this.lottoNumber = lottoNumber;
    }

    public static LottoNumber of(int number){
        LottoNumber lottoNumber = NUMBERS.get(number);
        if(lottoNumber == null){
            throw new IllegalArgumentException(OUT_OF_RANGE);
        }
        return lottoNumber;
    }
    //...
}
```
```java
public class Lotto {
    private List<LottoNumber> lottoNumbers;

    public Lotto(List<LottoNumber> lottoNumbers){
        validateDuplication(lottoNumbers);
        validateAmountOfNumbers(lottoNumbers);
        this.lottoNumbers = lottoNumbers;
    }

    private void validateAmountOfNumbers(List<LottoNumber> lottoNumbers) {
    }
    
    private void validateDuplication(List<LottoNumber> lottoNumbers) {
    }
}
```
```java
public class WinnigLotto {
    //...
    private static final String BONUS_CANNOT_BE_DUPLICATE_WITH_WINNING_NUMBER =
            "보너스 넘버는 위닝 넘버와 중복될 수 없습니다.";

    private Lotto winningLottoNumbers;
    private int bonusNumbers;

    public WinnigLotto(Lotto winningLottoNumbers, int bonusNumbers){
        this.winningLottoNumbers = winningLottoNumbers;
        if(isBonusNumberDuplicatedWithWinningNumber(winningLottoNumbers, bonusNumbers)){
            throw new IllegalArgumentException(BONUS_CANNOT_BE_DUPLICATE_WITH_WINNING_NUMBER);
        }
        if(bonusNumbers < 1 | bonusNumbers > 45){
            throw new RuntimeException();
        }
        this.bonusNumbers = bonusNumbers;
    }

    private boolean isBonusNumberDuplicatedWithWinningNumber(Lotto winningLottoNumbers, int bonusNumbers) {
        return true;
    }
    //...
}
```

위의 코드를 살펴보면 Lotto 클래스에서는 int 값인 로또 숫자 하나하나를 `LottoNumber`로 포장해서 사용하고 있는 것을 볼 수 있다.  
(`List<int>`가 아닌 `List<LottoNumber>` 사용)

물론 LottoNumber 대신에 Integer, int와 같은 자료형을 사용할 수도 있다.(아마 캐싱은 하지 않았거나, Lotto 클래스 내부에서 이루어졌을 것이다.)  
그렇게 되면 위에서 다루었듯이 **개별 로또 숫자**에 관한 관리가 **로또**에서 이루어져 로또가 수행하는 일이 늘어날 수 밖에 없어진다.  
자연히 Lotto 클래스의 크기도 커지게 될 것이다. 객체지향과도 작별 인사를 할 수 밖에 없어진다.

또 다른 문제도 있다. 현재는 로또 숫자의 범위가 1~45인데 혹여나 많은 사람들이 당첨되면 좋겠다는 생각을 하는 사람이 나타나서 나는 로또 숫자의 범위를  
1-10으로 할거야!라며 **조건을 변경시키고, 추가시킨다면?** 만약 Winning의 예시처럼 로또 숫자가 원시값이라면 같은 조건의 로또 숫자가 사용되는  
WinningLotto 클래스의 Lotto 클래스를 모두 고칠 수 밖에 없어진다.
> 정말 비효율적이다.

```java
public class WinnigLotto {
    //...
    private static final String BONUS_CANNOT_BE_DUPLICATE_WITH_WINNING_NUMBER =
            "보너스 넘버는 위닝 넘버와 중복될 수 없습니다.";

    private Lotto winningLottoNumbers;
    private LottoNumber bonusNumbers;

    public WinnigLotto(Lotto winningLottoNumbers, LottoNumber bonusNumbers){
        this.winningLottoNumbers = winningLottoNumbers;
        if(isBonusNumberDuplicatedWithWinningNumber(winningLottoNumbers, bonusNumbers)){
            throw new IllegalArgumentException(BONUS_CANNOT_BE_DUPLICATE_WITH_WINNING_NUMBER);
        }
        this.bonusNumbers = bonusNumbers;
    }

    private boolean isBonusNumberDuplicatedWithWinningNumber(Lotto winningLottoNumbers, int bonusNumbers) {
        return true;
    }
    //...
}
```

비효율성을 제거하기 위해 위처럼 원시값인 개별 로또 숫자를 LottoNumber로 포장만해 주면 로또 숫자의 확장, 변경에 대해 유연해진다.  
Lotto와 WinningLotto는 전혀 바꿀 필요가 없다. 로또 숫자를 포장한 LottoNumver만 수정해주면 되기 때문이다.

## 자료형에 구애받지 않는다.(여러 타입의 지원이 가능하다.)
점수라는 값을 포장한 Score 클래스가 있다. 현재 점수는 int 값이다.
```java
public class Scroe {
    private int score;

    public Scroe(int score) {
        validateScore(score);
        this.score = score;
    }

    private void validateScore(int score) {
        //...
    }
}
```

점수를 보여주는 역할만 했던 Score 객체에 연산 등의 기능이 추가되어 새로운 자료형의 지원이 필요해졌다면?  
기존의 Score 변수를 없앨 필요가 없다.

```java
public class Scroe {
   private int score;
   private double doubleScore;

   public Scroe(int score) {
      validateScore(score);
      this.score = score;
   }

   public Scroe(double doubleScore) {
      validateDobuleScore(doubleScore);
      this.doubleScore = doubleScore;
   }

   private void validateScore(int score) {
      //...
   }

   private void validateDobuleScore(double doubleScore) {
      //...
   }
}
```

앞서 말했듯이 유지, 보수에 도움이 되는 점을 이요하면 된다. 기존 Score 클래스를 활용하면 된다.  
doubleScore라는 멤버변수를 추가하고, 생성자 오버로딩을 통해 간단히 해결할 수 있다. String 값이 필요하다 해도 마찬가지로 해결이 가능하다.

## 마무리
원시 타입을 포장하게 되면, 그 변수가 의미하는 바를 정확하게 나타낼 수 있다.  
책임 관계 또한 명확해지고 코드의 유지, 보수에도 많은 도움이 된다.  
실제 프리코스 2주차 자동차 경주게임을 구현해보면서 그 장점을 느껴보고 싶다 !

### 참고
- [유원시 타입을 포장해야 하는 이유](https://tecoble.techcourse.co.kr/post/2020-05-29-wrap-primitive-type/)
- [소트웍스 앤솔러지 - 규칙 3: 원시값과 문자열의 포장](https://developerfarm.wordpress.com/2012/01/27/object_calisthenics_4/)