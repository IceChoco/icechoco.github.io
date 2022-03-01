---
title: '[Effective Java] Day 32 - Item 86 :: Serialize를 구현할지는 신중히 결정하라'
layout: post
categories: java
tags: java
comments: true
---

어떤 클래스의 인스턴스를 직렬화할 수 있게 하려면? → Class 선언에 implements Serializable 추가
구현한다고 선언하기는 아주 쉽지만, 그렇다고 프로그래머가 따로 신경 쓸 일이 없다고 생각하는 것은 오해이다.

## Serailizable의 문제점
### 1. Serailizable을 구현하면 릴리스한 뒤에는 수정하기 어렵다.
#### 클래스가 Serializable을 구현하면 직렬화된 Byte Stream 인코딩도 하나의 공개 API가 됨
따라서 이 클래스가 널리 사용될 경우, 다른 공개 API처럼 직렬화 형태를 영원히 지원해야 함

#### 커스텀 직렬화 형태로 설계하지 않고 자바의 기본 방식대로 직렬화한다면, 직렬화 형태는 최초 적용 당시 클래스의 내부 구현 방식에 묶임
즉, 기본 직렬화 형태에서는 클래스의 private, package-private 인스턴스 필드마저 API로 공개됨 → <span style="background-color: #fff5b1">캡슐화가 깨진다.</span>
Item 15에서 필드로의 접근을 최대한 막아 정보를 은닉하라 조언했지만,,,, 무력화된다.  
※ Item 15: 클래스와 멤버의 접근 권한을 최소화하라

뒤늦게 클래스 내부 구현을 고치면 원래의 직렬화 형태랑 달라진다.  
ex) 한 쪽은 구버전 인스턴스를 직렬화하고, 다른 쪽은 신버전 클래스로 역직렬화한다면 실패될 것

물론 `ObjectOutputStream.PutField`와 `ObjectInputStream.GetField`를 사용하여 원래의 직렬화 형태를 유지하면서 내부 표현을 바꿀 수 있지만, 소스코드가 지저분하다.
그러므로 직렬화 가능 클래스를 만들고자 한다면, 좀 더 길게 보고 고품질의 직렬화 형태도 주의해서 함께 설계해야 한다(Item 87,90)
```java
public class Person implements Serializable {
  //...
  private void writeObject(ObjectOutputStream oos) throws IOException{
    ObjectOutputStream.PutField putField = oos.putFields();
    putField.put("name", name);
    putField.put("age", age);
    oos.writeFields();
  }
  
  private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException {
    ObjectInputStream.GetField readField = ois.readFields();
    name = (String) readField.get("name","(defaultName)");//the default value to use if name does not have a value
    age = (int) readField.get("age",0);
  }
}
```

#### 직렬화가 클래스 개선을 방해하는 예 - Serial Version UID
- 스트림 고유 식별자, Serail Version ID(직렬 버전 UID)
- **JVM**은 모든 Serializable Class에 각각 serialVersionUID라 불리는 <span style="background-color: #fff5b1">**Version Number를 매칭 한다.**</span>
- static final long filed

보통 IDE에서는 Class명 기준으로 자동으로 생성해주는 경우가 많다.
SerialVersionUID를 따로 설정해주지 않더라도 JVM에서 RunTime 시 암호 해시 함수(SHA-1)를 적용해 자동으로 생성해준다.

- 이 경우는 클래스 이름, 구현한 인터페이스들, 컴파일러가 자동으로 생성해 넣은 것을 포함한 대부분의 클래스 멤버들이 고려된다.
- 나중에 메서드를 추가하는 식으로 이들 중 하나라도 수정한다면 Serial Version UID 값도 변한다.

Serialization, Deserialization 할 때 Serial Version UID를 확인한 후 처리하며, 자동 생성되는 값에 의존하면 이 값이 맞지 않아 호환성이 깨져버려 Runtime에 <span style="background-color: #fff5b1">**InvalidClassException**</span>이 발생한다.

### 2. 버그와 보안 구멍이 생길 위험이 높아진다.
- 기본 역직렬화를 사용하면 생기는 문제
  - (1) 불변식 깨짐
  - (2) 허가되지 않은 접근에 쉽게 노출

직렬화는 언어의 기본 메커니즘인 '객체는 생성자를 사용하여 만든다'를 우회하는 객체 생성 기법이다.  
역직렬화는 일반 생성자의 문제가 그대로 적용되는 숨은 생성자다. 하지만 일반 생성자처럼 소스 전면에 드러나지 않는다.  
그러다보니 '생성자에서 구축한 불변식을 모두 보장해야 한다, 생성 도중 공격자가 객체 내부를 들여다 볼 수 없도록 해야한다'는 사실을 떠올리기 어렵다.

### 3. 해당 클래스의 신버전을 릴리스할 때 테스트할 것이 늘어난다.
- 직렬화 가능 클래스가 수정될 때 검사해야할 것들
  - 양방향 직렬화/역직렬화 모두 성공하는가?
    - 신버전 인스턴스를 직렬화 → 구버전으로 역직렬화
    - 구버전 인스턴스 직렬화 → 신버전으로 역직렬화
  - 원래의 객체를 충실히 복원하는가?
- 테스트 해야 할 양이 테스트 가능 클래스의 수 & 릴리스 횟수에 비례하여 ↑
- 클래스를 처음 제작할 떄 커스텀 직렬화 형태를 잘 설계해놨다면 테스트 부담↓ 가능(iTEM 87, 90)

## Serialize 구현 여부는 가볍게 결정할 사안이 아니다.
아래와 같은 경우라면 선택의 여지가 없긴하다.
- 객체를 전송하거나 저장할 때 자바 직렬화를 이용하는 프레임 워크용으로 만든 클래스
- Serializable을 반드시 구현해야 하는 다른 클래스의 컴포넌트로 쓰일 클래스

Serialize 구현에 따르는 비용이 적지 않으니, 클래스를 설계할 때 마다 비용 & 이득을 잘 판단해라.
대개 아래와 같은 기준으로 구현을 결정한다.
- BigInteger, Instant와 같은 **값** 클래스, **Collection** 클래스
  → Serailizable을 구현
- 스레드 풀처럼 **동작**하는 객체를 표현하는 클래스
  → Serailizable을 구현하지 않음
  
## 상속용으로 설계된 클래스(Item 19)는 대부분 Serializable을 구현하면 안되며, 인터페이스도 대부분 Serialize를 확장해서는 안된다.
상속용으로 설계된 클래스 중 Serializable을 구현한 예
1. Throwable :서버가 RMI를 통해 클라이언트로 예외를 보내기 위해 Serialzable을 구현
   - RMI(Remote Method Invocation, 원격 메소드 호출): 분산되어 존재하는 객체간의 메시지 전송(메소드를 호출하는 것을 포함)을 가능하게 하는 프로토콜
2. component: GUI를 전송, 저장, 복원하기 위해 Serializable을 구현했으나 Swing과 AWT가 널리 쓰이던 시절에도 현업에서 이런 용도로 거의 쓰이지 않았음

### 내가 작성하는 클래스의 인스턴스 필드가 직렬화와 확장이 가능할 때, 주의해야할 점
#### 1. finalize 메서드를 재정의하지 못하게 하라
인스턴스 필드 값 중 불변식을 보장해야 할 게 있는 경우, 반드시 하위 클래스에서 finalize 메서드를 재정의하지 못하게 해야한다.  
즉, finalize 메서드를 내가 재정의하고 final로 선언해라. → 이렇게 안하면 finalizer 공격(item 8, p42)을 당할 수 있다.

#### 2. 인스턴스 필드 중 기본값으로 초기화되면 위배가 되는 불변식이 있다면, readObejctNoData 메서드를 반드시 추가해라
- **슈퍼클래스 = 상속가능 + 직렬화 + 디폴트 값을 가지는 메서드** 인 경우
- 자바 4에 추가됨. 기존의 직렬화 가능 클래스에 직렬화 가능 상위 클래스를 추가하는 경우를 위한 메서드
```java
private void readObjectNoData() throws InvalidObjectException{
    throw new InvalidObjectException("스트림 데이터가 필요합니다.");
}
```

### Serailizable을 구현하지 않기로 할 때, 주의해야할 점 
상속용 클래스인데 직렬화를 지원하지 않는 경우, 하위 클래스가 직렬화를 지원하려 할 때 부담이 늘어난다.  
이런 클래스를 역직렬화 하기 위해서는 상위 클래스는 **매개변수가 없는 생성자를 제공**해야 한다.  
→ 제공하지 않은 경우, 하위 클래스는 직렬화 프록시 패턴(Item 90)을 사용해야 함

### 내부 클래스(item 24)는 직렬화를 구현하지 말아야 한다.
내부 클래스에는 바깥 인스턴스의 참조, 유효 범위 안의 지역변수 값을 저장하기 위해 컴파일러가 생성한 필드들이 자동으로 추가된다.  
내부 클래스에 대한 기본 직렬화 형태는 분명하지가 않다.  
단, static 멤버 클래스는 Serializable을 구현해도 된다.

## 결론
- Serializable을 구현하기 위해 선언하는 것은 쉽지만, 신경 써줘야할 부분이 많다.
- 한 클래스의 여러 버전이 상호작용할 일이 없고, 서버가 신뢰할 수 없는 데이터에 노출될 가능성이 없는 등, 보호된 환경에서만 쓰일 클래스가 아니라면 Serializable 구현은 아주 신중하게 해라
- 직렬화 + 상속 가능 클래스라면 주의사항이 더욱 많아짐을 유의하라.

### 참고
- [Interface Serializable docs](https://docs.oracle.com/javase/8/docs/api/java/io/Serializable.html)
- [직렬화(Serialization)](https://brunch.co.kr/@oemilk/179)