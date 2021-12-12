---
title: '[Java] 패턴 매칭을 위한 유한 상태 머신(Finite state machine) 사용'
layout: post
categories: java
tags: java
comments: true
---

**이 글의 목적**
- FSM(Finite state machine)이 무엇인지, Pattern이 FSM이 되는 이유에 대해 알 수 있다.  
- <span style="color:grey">*번역이 매끄럽지 않을 수 있습니다. 문맥상 이해 부탁드립니다.* </span>

* * *  
유한 상태머신은 Pattern matching에 굉장히 유용한 툴이다. Pattern mathcing을 위해 어떻게 FSM을 사용하는지와 JAVA로 쓰여진 예시를 보겠다.

## Finite State Machine (유한상태머신)
- 유한상태머신은 `수학 방향 그래프`
- State는 여기서 정점을 뜻함
- 전이는 그래프 전이

## Sample Problem
String이 특정 패턴으로 끝나는지 확인하고 싶다고 가정해 보자. 주어진 String이 @로 시작해서 숫자, 마지막으로 #로 끝나는지 여부를 알고 싶다. 아래는 3가지 예시이다.
- 예시
 - @3#
 - @12344#
 - @0112#

여기서 우리는 String이 이런 패턴으로 끝나는지 알고 싶다. 우리는 이 패턴을 모델링하는 유한 상태 머신(FSM)을 만들 수 있다.

## Non-Finite State Machine
만약 FSM 솔루션을 쓰지 않는다면
- @와 #를 찾고
- 그 사이에 문자를 넣고 변환하여 성공적으로 변환되면 패턴을 가질 수 있는 코드를 작성할 수 있다.

성공적으로 변환되지 않으면 문자가 없을 수도 있다고 가정하면서 말이다. 하지만 일련의 숫자가 정말 큰 정수가 될 수 있다. 그러니 이건 쓰기 쉬운 알고리즘은 아니다.

![FSM](/assets\img/FSM.PNG)

FSM 접근 방식을 사용하여 패턴의 각 State에서 시작한다. **State 0에서 시작**하고 만약 다음 문자가 @이면 State 1로 이동한다. 따라서 전이선을 볼 때마다 전이선에 있는 문자가 전이 방법을 보여준다. 다음 문자를 읽은 후에 @이면 State 1로 전환한다.
 
State 1에서는 0부터 9까지의 숫자 중 하나를 얻으면 State 2로 전환된다. State 2에서 0부터 9까지의 숫자 중 하나를 얻으면 State 2로 유지되므로 두 자리 숫자 또는 세 자리 숫자 또는 100이 될 수 있다. State 2에서 다음 자리가 @이면, @가 패턴의 시작이므로 State 1로 돌아간다. 왜냐하면 2개의 @ 뒤에 숫자와 숫자 기호가 올 수 있기 때문이다.  

State 2에서 #가 보이면 State 3으로 이동한다. State 3에서 @ 기호를 보면 패턴을 시작하는 State 1로 돌아간다. 다음 문자가 **전이선에 없는 문자인지 확인되면 자동으로 0 상태로** 되돌아간다. 그래서 우리가 State 2에 있고 다음 문자가 문자 B이면, 문자 B에 대해 State 2를 떠나는 전이선이 없기 때문에 그것은 State 0으로 돌아간다는 것을 암시한다. 그럼 어떻게 위 로직이 구현되었는지 자바코드를 확인해보자.

## FSM 구현 코드
- Variables
 - a: `A1@312#` 값을 가진 String이 있고, 이 String은 패턴을 가지고 있는지 우리가 확인할 대상이다. 이 string은 @ 기호 뒤에 숫자, 그리고 끝에 #가 있다.
 - digits: 간단하게 숫자를 저장하는 변수
 - state: 초기값 0

왼쪽부터 오른쪽까지 문자열의 각 문자를 반복하는 for 루프가 있다. 그래프에서와 같이 state 0에 있고 다음 문자가 @ 기호이면 state 1로 이동한다. 그런 다음 for 내부 이후 소스들은 if/else 구문 이므로 건너뛴다.  
 
지금 state 1에 있으면 다음 문자를 얻는다. 다음 문자가 숫자라면 state 2로 이동한다. @ 기호인 경우 state 1로 유지되고 다른 값인 경우 state 0으로 돌아간다. state 2인 경우 다음 문자가 #이면 state 3로 이동, 숫자면 state 2에 그대로, @이면 state 1으로 이동하고, 다른 값인 경우 모두 state 0로 돌아간다. 마지막으로 state 3인 경우 다음 문자가 @이면 state 1로 이동, 다른 값인 경우 모두 state 0로 돌아간다.  

그리고 for문이 끝나고 state가 3에 있는 경우 스트링이 패턴과 매치된다는 걸 알 수 있다. state가 3이 아닌 경우 스트링이 패턴과 맞지 않음을 의미한다.

```java
public class FSMPatternExample {

    public static void main(String[] args) {
        String s = new String("A1@312#");
        String digits = new String("0123456789");
        int state = 0;

        for(int idx = 0; idx < s.length(); idx++){
            if(state == 0){
                if(s.charAt(idx) == '@')
                    state = 1;
            }else if(state == 1){
                if(digits.indexOf(s.charAt(idx)) != -1){
                    state = 2;
                }else if(s.charAt(idx) == '@'){
                    state = 1;
                }else{
                    state = 0;
                }
            }else if(state == 2){
                if(s.charAt(idx) == '#'){
                    state = 3;
                }else if(digits.indexOf(s.charAt(idx)) != -1){
                    state = 2;
                }else if(s.charAt(idx) == '@'){
                    state = 1;
                }else{
                    state = 0;
                }
            }else if(state == 3){
                if(s.charAt(idx) == '@'){
                    state = 1;
                }else{
                    state = 0;
                }
            }
        }

        if(state == 3){
            System.out.println("It matches");
        }else{
            System.out.println("It does not match");
        }
    }
}
```

위 소스코드를 수행하면 `It matches`가 출력된다. String을 바꿔가며 수행해보면 각 String 별 수행결과가 달라진다.
- A1@3555555555512#      → `It matches`
- A13555555555512        → `It does not match`
- A135555@r55555512#     → `It does not match`
- A135555@@@@@55555512#  → `It matches`

위 케이스 외에도 다양한 문자를 통해 테스트 수행이 가능하다. 이것이 String 패턴 일치 여부 검사에 FSM을 사용하는 방법이며 코드를 작성할 때 이러한 유형의 문제에 대해 매우 간단하고 매우 멋지다.

### 참고
- [Using Finite State Machines for Pattern Matching in Java](https://www.youtube.com/watch?v=ZfW7FwuBd90)