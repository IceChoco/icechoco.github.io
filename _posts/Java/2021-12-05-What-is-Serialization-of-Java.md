---
title: '[Java] 직렬화(Serialization)와 마커 인터페이스(Marker Interface)'
layout: post
categories: java
tags: java
comments: true
---

**이 글의 목적**
- 직렬화(Serialization)와 마커 인터페이스(Marker Interface)가 무엇인지 이해하고 직렬화(Serialization)를 사용할 수 있다.

## 직렬화(Serialization)란?
- 객체의 상태를 영속화하는 메커니즘
- 객체를 파일, 메모리, DB와 같은 다른 환경에 저장했다가 나중에 다시 불러와서 재구성 할 수 있게 만드는 일련의 과정

## 자바 직렬화란?
- 1997년에 처음 도입되었으며 쉬운 방법으로 분산객체를 생성하기 위해 도입된 기술
- <span style="background-color: #fff5b1">객체의 상태를 Byte 배열과 같은 ByteStream으로</span> 만드는 것
- **Deserialization**은 반대로 <span style="background-color: #fff5b1">Byte Stream을 객체로</span> 변환하는 것

![serialization](/assets\img/serialization.png)
<div style="text-align: center; color:grey">Serialization</div>  
<br/>
![Deserialization](/assets\img/Deserialization.png)
<div style="text-align: center; color:grey">Serialization and Deserialization</div>

## 언제 쓸 수 있는가?
객체의 상태를 어딘가에 저장하여 영속해야 할 필요가 있을 때
  - 파일 또는 **Database에 저장하거나** 캐시와 같은 메모리의 형태가 될 수 있음
  - **Network를 통해 전송할 수 있도록** 변환하는 것  
    - 다른 VM에게 객체의 정보를 전송해야 할 때, ByteStream으로 변환해서 전송할 때 사용

## 어떻게 쓸 수 있는가?
- **java.io.Serializable**

```java
package java.io;

public interface Serializable {
}
```
```java
public class Singleton2 implements Serializable {

    private static final Singleton2 instance = new Singleton2();

    private Singleton2(){
    }

    public static Singleton2 getInstance(){
        return instance;
    }

}
```
Serialization 할 수 있는 Class가 되려면 <span style="background-color: #fff5b1">Serializable 인터페이스를 클래스에 implements</span>해야 한다. Serializable는 공개API가 없는 단순한 마커인터페이스이다. 
### 마커 인터페이스(Marker Interface)
- 일반적인 인터페이스와 동일하지만 사실상 아무 메서드도 선언하지 않은 인터페이스  
예를 들어 아래와 같다.
```java
public interface SomeObject {
}
```
얼핏 보기엔 난해한 코드이다. 인터페이스만 있고 메서드가 없다. 이런 대표적인 마커 인터페이스로는 `Serializable`,`Cloneable`와 Spring에서 event 리스너를 사용한다면 종종 보이는 `EventListner`가 있다. 참고로 Spring의 `ApplicationListner` 인터페이스가 상속받고 있다.

#### 어떻게 무엇을 위해 만들어졌나요?
뭔가 대단한 것처럼 보일수도 있지만 실질적으로는 간단하다. 대부분의 경우에는 단순한 타입 체크라고 할 수 있다. 자바의 대표적인 마커 인터페이스인 Serializable를 살펴보자.
```java
public interface Serializable {
}
```
메소드가 한개도 선언되어 있지 않다. Serializable 인터페이스 같은 경우에는 직렬화를 할 수 있다는 뜻이다. 즉, 이 인터페이스를 구현하지 않은 클래스의 경우에는 직렬화를 하지 못한다. 아래는 간단한 예제이다.
- **java.io.ObjectOutputStream**
  - writeObject(Object obj)

```java
public class SomeObject{

    private String name;
    private String email;

    public SomeObject(String name, String email){
        this.name = name;
        this.email = email;
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        byte[] serializedObject;

        //1. 파일 출력
        File f= new File("a.txt");
        ObjectOutputStream fileOutputStream = new ObjectOutputStream(new FileOutputStream(f));
        fileOutputStream.writeObject(new SomeObject("IceChoco", "test@test.com"));

        //2. byteArray로 출력
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream);
        objectOutputStream.writeObject(new SomeObject("IceChoco", "test@test.com"));
        serializedObject = byteArrayOutputStream.toByteArray();

        System.out.println(serializedObject);
    }
}
```
ObjectOutputStream의 writeObject을 이용하여 객체를 직렬화한다. 위 소스코드에서는 직렬화된 바이트 스트림을 2가지 방법으로 획득했다.  
1. FileOutputStream을 활용하여 파일로 출력
2. ByteArrayOutputStream의 toByteArray 메서드를 이용하여 byte 배열로 출력

하지만 위 코드를 실행하면 `java.io.NotSerializableException: item03.SomeObject` 에러가 발생한다.  
이는 직렬화를 할 수 있는 Serializable을 구현하지 않았기 때문이다. 그렇다면 직렬화를 할 수 있도록 Serializable 인터페이스를 구현해보자.

```java
public class SomeObject implements Serializable {

    private String name;
    private String email;
    //...
}
```
간단하게 구현할 수 있다. 인터페이스의 메소드도 없으니 구현할 메소드도 필요 없다. 그냥 선언만 해주면 된다. 그럼 위에서 사용했던 `writeObject()` 메소드 안을 들여다 보자. `writeObject()` 메소드 안에는 `writeObject0()`가 존재한다. 이 메소드 맨 아래에 보면 다음과 같은 코드가 있다.
```java
  //... 
  if (obj instanceof String) {
    writeString((String) obj, unshared);
  } else if (cl.isArray()) {
    writeArray(obj, desc, unshared);
  } else if (obj instanceof Enum) {
    writeEnum((Enum<?>) obj, desc, unshared);
  } else if (obj instanceof Serializable) {
    writeOrdinaryObject(obj, desc, unshared);
  } else {
    if (extendedDebugInfo) {
      throw new NotSerializableException(
        cl.getName() + "\n" + debugInfoStack.toString());
    } else {
      throw new NotSerializableException(cl.getName());
    }
  }
  //...
```
이 코드에는 if문이 꽤 있다. 보면 String은 Serializable을 구현했으니 되고 배열도 Serializable 할 수 있고 Enum도 Serializable를 구현했으니 되고 다음으로는 Serializable가 되어있는지 체크하는 부분이다. 만약 Serializable가 없다면 에러로 처리한다. 위에서 보았듯이 간단하게 Serializable가 선언되었는지 안되어 있는지 **체크 정도만**한다. 실질적으로 뭘 하는 건 아니다. 그래서 **마커 인터페이스**라고 부른다.
  
마커 인터페이스는 어노테이션으로도 대체 가능하다. 만약 @SomeAnnotation이라는 어노테이션이 있다면 아래와 같이 가져와서 체크하면 된다.
```java
final SomeAnnotation someAnnotation = someObject.getClass().getAnnotation(SomeAnnotation.class);
```

- **java.io.ObjectInputStream**
  - readObject
```java
    static void serializableTest() throws IOException, ClassNotFoundException {
        ByteArrayInputStream byteArrayOutputStream = new ByteArrayInputStream(serializedObject);
        ObjectInputStream objectOutputStream = new ObjectInputStream(byteArrayOutputStream);
        SomeObject someObject = (SomeObject) objectOutputStream.readObject();

        System.out.println(someObject.name+" "+someObject.email);
    }
```  
직렬화가 `ObjectOutputStream`을 이용했다면 역직렬화는 `ObjectInputStream`을 이용한다.

#### 인터페이스와 마커 어노테이션의 차이
마커 인터페이스 같은 경우에는 컴파일 시점에 발견할 수 있다는 큰 장점이 있다. 그리고 또한 적용 범위를 좀 더 세밀하게 지정할 수 있다.

만약 어노테이션 자료형을 선언할 때 target에 ElementType.TYPE라고 지정해서 사용한다고 하면 ElementType.TYPE은 클래스 뿐만 아니라 인터페이스에도 적용 가능하다. 그런데 특정한 인터페이스를 구현한 클래스에만 적용할 수 있어야 하는 마커가 필요하다고 가정해보자.

마커 인터페이스를 쓴다면 그 특정 인터페이스를 상속하도록 선언만 하면 된다. 그럼 마커를 상속한 모든 자료형은 자동으로 그 특정 인터페이스의 하위 자료형이 된다.

##### 마커 어노테이션의 장점
마커 어노테이션은 유연하게 확장이 가능하다. 어노테이션을 만들어 사용한 뒤에도 계속적으로 더 많은 정보를 추가할 수 있는 것이 큰 장점이다.

예를 들어, 어떤 어노테이션을 만들고 배포를 한 뒤에 뭔가 더 정보를 추가하고 싶다면 새로 추가된 요소들에 대해 default 값을 갖게 하면 하위 호환성도 지킬 수 있으며 처음에는 마커 어노테이션으로 시작하여 쓰다가 나중에는 기능이 많은 어노테이션으로 진화 가능하다.

하지만 인터페이스 경우에는 메소드를 만드는 순간 하위 호환성이 깨지므로 마커 어노테이션처럼 지속적인 진화는 불가능하다.

마커 어노테이션과 마커 인터페이스 중 둘 중 어느게 낫다고 할 수 없다. 각각의 쓰임새가 다르기 때문이다. 위에서 언급했듯이 새로운 메소드가 없이 자료형을 정의하고 싶다면 마커 인터페이스를 이용해야 하고 클래스나 인터페이스 이외의 마커를 달아야 하고 앞으로도 더 많은 추가 정보가 있다고 생각하면 마커 어노테이션을 사용하면 된다.

### 참고
- [[10분 테코톡] 🍄비밥의 자바 직렬화](https://www.youtube.com/watch?v=3iypR-1Glm0)
- [자바의 마커 인터페이스](http://wonwoo.ml/index.php/post/1389)
- [Serialization](https://developnote-blog.tistory.com/entry/Serialization)
- [객체 직렬화(Object Serialization)](https://linuxism.ustd.ip.or.kr/1433)