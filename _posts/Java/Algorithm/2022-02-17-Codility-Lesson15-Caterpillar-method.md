---
title: '[Algorithm] Codility Lesson 15 - Caterpillar method'
layout: post
categories: java
tags: java
comments: true
---

무턱대고 A4용지 이면지에 영어 해석하면서 정리했었는데... 오늘 멘토링을 통해 우연히 알게된 개발자 분의 블로그를 보게 되었다. 블로그에 이것저것 많이 올리시고, git 잔디까지 빼곡한걸 보면서 정말 대단하다 생각했다. 그리고 나도 블로그에 정리한 내용을 올리면 보기 쉽겠다 생각해서 만든 Algorithm 카테고리의 첫 포스팅!  

알고리즘 소스는 gitHub에 올리지만, Codility 내 각 Lesson마다 있는 교재를 읽고 정리한 내용은 L자 폴더 내 이면지로 쌓여있어 원활히 찾기가 쉽지 않았다...^^
번역이 올바르지 않을 수 있으므로 양해 부탁드립니다. 틀린 부분 있으면 코멘트 남겨주세요 :D

------

caterpillar method, 이름하여 애벌레 메소드!  
- 주된 아이디어: 애벌레의 움직임을 연상시키는 방식으로 요소를 확인하는 방식

## 15.1 사용예제
시퀀스 a0, a1, . . . , an−1 (1 <= ai <= 10^9)는 **요소의 합이 S인** 연속 부분수열을 포함한다.
 - S=12인 시퀀스 찾기  
![algorithm_lessson15_array](/assets\img/algorithm_lessson15_array.PNG)

애벌레의 각 위치는 원소의 합계가 s보다 크지 않은 다른 인접 시퀀스를 나타냄.  
처음으로 첫번째 원소에 애벌레를 설정하고 다음으로 아래 단계를 수행.
- (Case 1) 가능하면 오른쪽 끝(애벌레의 머리-front)을 앞으로(오른쪽으로) 이동하고 애벌레의 사이즈를 늘린다.
- (Case 2) 그렇지 않으면 왼쪽 끝(back)을 앞으로 이동하고 애벌레의 크기를 줄인다.

이런식으로 왼쪽 끝의 모든 위치에 대해(for문 반복) 합계가 s보다 크지 않은 원소를 덮는 가장 긴 애벌레를 알 수 있다.

| 횟수  | Total | elements                               |
|-----|-------|----------------------------------------|
| 1   | 6     | 6                                      |
| 2   | 8     | 6,2 → 7더하면 15이므로 15>12, case(2)에 해당    |
| 3   | 2     | 2                                      |
| 4   | 9     | 2,7 → 9+4(next) = 13 > 12, case(2)에 해당 |                                
| 5   | 7     | 7                                      |  
| 6   | 11    | 7, 4                                   |  
| 7   | 12    | 7, 4, 1                                |  

원소의 총계가 s인 수열이 있다면, 애벌레가 모든 원소를 도는 순간이 Certainly 있다.

### O(N) 시간복잡도의 Caterpillar 알고리즘
```java
public class MainIdea {
    private boolean caterpillarMethod(int[] A, int S){
        int n = A.length, front=0, total = 0;
        for(int back=0; back < n; back++){
            while(front < n && total + A[front] <=S){
                total++;
                front++;
            }
            if(total == S) return true;
            total -= A[back];
        }
        return false;
    }
}
```
#### 시간복잡도 추론  
이중 loop이니까 O(n^2) 아니야? 라고 생각했지만 땡! 모든 단계에서 애벌레의 front나 back을 이동하며, 그들의 위치는 n을 초과하지 않는다. 따라서 실제로 O(n)의 솔루션을 얻게된다. 이 추정 방법은 상각된 Cost에 기초하고 있는 추정 방법이다.

## 15.2 Excercise
### <문제>
n개의 스틱들이 있다. 길이는 a0, a1, . . . , an−1 (1 <= ai <= 10^9)  
목표: 이 stick을 사용하여 구성할 수 있는 삼각형의 수 세기  
x<y<z, ax+ay>az를 만족하는 수 3개가 나올 수 있는 경우의 수를 구하라.

### <풀이>
모든 x,y 쌍에 대해 가장 큰 Z stick을 알 수 있다. 왜? ax + ay > az를 만족하기 때문이다.  
y<k<=z를 만족하는 모든 stick K도 사용할 수 있다. 왜냐하면 az + ay > ak 조건을 여전히 만족하기 때문이다.  
우리는 이 삼각형을 한 번에 모두 합칠 수 있다. 단, z값이 시작부터 모든 경우의 수를 찾는다면 시간 복잡도는 O(N^3)가 된다.  
하지만 이 때, 앞서 배운 애벌레 솔루션을 사용할 수 있다! y의 값을 증가시킬 때 z의 값을 가능한 한 늘릴 수 있다.

### O(N^2)의 삼각형들의 수를 구하는 알고리즘
```java
public class MainIdea {
    private boolean triangles(int[] A){
        int n = A.length, result=0;
        for(int x=0; x < n; x++){
            int z = x+2; //x+y>z를 만족하는 z의 최소값은 x+2이다. +1을 하면 같아져서 >를 만족하지 못한다.
            for(int y =x+1;y<n;y++){
                while(z<n && A[x]+A[y]>A[z])
                    z++;
                result += z-y-1;//ax+ay>az이므로 z와 y 사이의 값이 위 식을 만족하는 갯수임. y<k<=z인 k의 수를 모두 구하기 위함.
            }
        }
        return false;
    }    
}
```
위 알고리즘의 시간복잡도는 O(n)이다. 이유는 모든 스틱x에 대해 y와 z의 값이 O(n)번 증가하기 때문이다.
