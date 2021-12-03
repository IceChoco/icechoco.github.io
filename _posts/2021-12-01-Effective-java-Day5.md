---
title: '[Effective Java] Day 5 - Item 2 :: 생성자에 매개변수가 많다면 빌더를 고려하라 (2), Item 3 :: Private 생성자나 열거 타입으로 싱글턴임을 보증하라'
layout: post
categories: java
tags: java
comments: true
---

Day 5 기록 시작!

## Item 2 :: 생성자에 매개변수가 많다면 빌더를 고려하라 (2)
### 3. 빌더 패턴(JavaBeans pattern) (2)

소스코드는 Day4 게시글 참고

Pizza.Build 추상 클래스는 `재귀적 타입 한정`을 이용하고 selft라는 메소드를 사용해서 클래스에서는 형변환 하지 않고도 메서드 연쇄를 지원할 수 있다. `selft type`이 없는 자바를 위한 우회 방법을 `시물레이트한 셀프 타입` 관용구라 한다.
   
하위 클래스의 메서드인 NyPizza.Builder가 상위 클래스의 메서드가 정의한 반환 타입이 아닌(이 예제에서는 상위 클래스의 메서드가 추상 메서드라 반환 타입이 구현되어 있지 않긴함), 그 하위타입인 NyPizza를 반환하는 기능을 `공변환 타이핑(convariant return typing)`이라 한다. 이 기능을 이용하면 클라이언트 코드가 타입 캐스팅을 할 필요가 없어진다.

```java
NyPizza nyPizza = new NyPizza.Builder(NyPizza.Size.SMALL) //1
    .addTopping(Pizza.Topping.SAUSAGE)                    //2
    .addTopping(Pizza.Topping.ONION)                      //3
    .build();                                             //4

Calzone calzone = new Calzone.Builder()
    .addTopping(Pizza.Topping.HAM)
    .sauceInde()
    .build();
```

#### 위 nyPizza 소스코드가 작동하는 순서
1. NyPizza.Builder 클래스 수행  
    `NyPizza.size = MEDIUM`
   → NyPizza.Builder 클래스 반환
2. NyPizza는 Pizza 클래스를 상속받고 있으므로 바로 Pizza.Builder 클래스 내 addToping 수행.
   피자의 빌더가 자신의 하위타입을 받을 수 있으므로 NyPizza.Builder 클래스를 타입으로 받을 수 있음.
   `Pizza.Builder.toppings = HAM`
   self() 리턴 → NyPizza 내 self() 수행 → NyPizza.Builder 클래스 반환
3. Pizza.Builder 클래스 내 addToping 수행.
   `Pizza.Builder.toppings = HAM, ONION`
   self() 리턴 → NyPizza 내 self() 수행 → NyPizza.Builder 클래스 반환
4. NyPizza.Builder.build 수행 → NyPizza 메소드 수행 → super(builder) 수행 → Pizza class의 Builder를 매개변수로 받는 Pizza 메소드 수행 → 빌더가 가지고 있는 토핑을 Pizza.toppings로 바꿔줌 → NyPizza로 돌아와 builder가 가지고 있는 사이즈를 NyPizza의 사이즈에 셋팅 → NyPizza 객체 리턴됨
 
#### 빌더패턴의 장점
1. 빌더를 이용하면 **가변인수 매개변수를 여러개 사용**할 수 있다.
    -  생성자나 팩토리는 가변인자를 맨 마지막에 매개변수에 한번 밖에 못씀
2. 토핑 예제에서 본 것 처럼 메서드를 여러 번 호출하도록 하고 각 호출 때 **넘겨진 매개변수들을 하나의 필드로** 모을 수 있다.  
3. 빌더는 상당히 유연해서 빌더 하나로 여러 객체를 순회하며 만들 수도 있다.
4. 빌더에 넘기는 매개변수에 따라 다른 객체를 만들어 줄 수 있다.
5. 객체마다 부여되는 일련번호와 같은 **특정 필드는 빌더가 알아서 채우도록** 할 수도 있다.

#### 빌더패턴의 단점
1. 객체 생성 전 빌더를 먼저 만들어야하므로 성능에 민감한 상황에서는 문제가 될 수 있다.
    - 하지만, 성능 차이가 아주 미미하므로 성능때문에 빌더를 안쓴다는건... 크게 고려하지 않아도 된다!
2. 생성자를 사용하는 것보다 코드가 장황하다

### 결론
- **생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는게 좋다.**
    - 매개변수 중 다수가 선택 값이거나, 데이터 타입이 같다면 더더욱!
- 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 **간결하고**, 자바빈즈보다 훨씬 **안전하다.**
- 빌더 패턴은 매개변수가 많거나(한 4개 이상) 앞으로 늘어날 가능성이 있는 경우에 사용하는 것이 좋다.
    - API는 시간이 지날수록 매개변수가 많아지는 경향이 있으니 애초에 빌더로 시작하는 것을 추천

### 참고: Lombok annotation을 추가하여 빌더패턴 사용하기
```java
package item02;

import lombok.Builder;
import lombok.Singular;

import java.util.List;

@Builder
public class NutritionFactsLombok {
	@Builder.Default private int servingSize = 10;
	private int sodium;
	private int carbohydrate;
	private int servings;
	@Singular private List<String> names;

	public static void main(String[] args) {
		NutritionFactsLombok nutritionFactsLombok = NutritionFactsLombok.builder()
				.servings(10)
				.carbohydrate(100)
				.name("ara")
				.name("minwoo")
				.clearNames()
				.build();

//		NutritionFactsLombok facts = new NutritionFactsLombok();//기본 생성자가 안만들어짐. @NoArgsConstructor 사용해야함
	}
	
}
```

## Item 3 :: Private 생성자나 열거 타입으로 싱글턴임을 보증하라
- **싱글턴**: 객체의 인스턴스가 오직 1개만 생성되는 패턴을 의미  
ex) 함수와 같은 <u>stateless Object(무상태 객체)</u>, 설계상 유일해야 하는 시스템 컴포넌트  
    - **stateless Object(무상태 객체)**: 인스턴스 변수가 없는 객체
      ```java
        public class Car {
            void Car() {
                System.out.println("I'm a car!");
            }
        }
      ```
      아래와 같이 컴파일 타임에 정의되어 있고 변경되지 않는 상수도 stateless Object이다.
      ```java
      public class Car {
            static final String MESSAGE = "I'm a car!";
           
            void Car() {
                System.out.println(MESSAGE);
            }
        }
      ```
      stateless Object와 연관된 헷갈릴 수 있는 개념으로 Immutable Object가 있다.
      객체 지향 프로그래밍에서 Immutable Object란 상태를 바꿀 수 없는 객체이다.
      ```java
      public class Car {
            private static String message;
           
            void Car(String message) {
                this.message = message;
            }
           
            public void printMessage() {
                System.out.println(message);
            }
        }
      ```
      이와 같이 불변 객체는 상태가 한번 지정되면 바뀔 수 없지만 컴파일 시점에 값이 정의되는 것이 아니기때문에 stateless Object는 아니다.
 
클래스를 싱글턴으로 만들면 클라이언트 코드를 **테스트하기가 어렵다**. 싱글톤이 인터페이스를 구현한게 아니라면 싱글턴 인스턴스를 mock 객체 구현으로 대체할 수 없기 때문이다.  
- **Mock 객체**: 실제 객체를 다양한 조건으로 인해 제대로 구현하기 어려울 경우 **가짜 객체를 만들어 사용**하는데, 이를 Mock 객체라 한다.  
  Mock 객체가 필요한 경우,
    - 테스트 작성을 위한 환경 구축이 어려운 경우
    - 테스트가 특정 경우나 순간에 의존적인 경우
    - 시간이 걸리는 경우

싱글턴을 만드는 방식은 두가지가 있으며 **공통점**이 있다.
- 생성자는 private으로 감춰둔다.
- 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 마련해둔다.

### 1. Fianal 필드 방식
### 2. static 팩토리 메소드

### 참고
- [[이팩티브 자바] #2 생성자 매개변수가 너무 많아? 빌더 패턴을 써 봐](https://www.youtube.com/watch?v=OwkXMxCqWHM&t=4s)
- [[java] Stateless Object](https://kyeoneee.tistory.com/54)
- [ITEM 3: Singleton](https://dahye-jeong.gitbook.io/java/java/effective_java/2021-01-14-singleton)