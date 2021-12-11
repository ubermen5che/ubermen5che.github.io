---
title: "[CS] Garbage Collection-1"

categories:
  - CS
tags:
  - Java
  - CS
  - Reference counting
  - Mark & sweep
  - Generative Collection
  - Copy Collection
---

## Garbage Collection이란?
GC는 1959년에 개발된 자동적 메모리 관리 기법이다. GC란 이름을 보면 유추할 수 있듯이 여기서 말하는 Garbage는 사용되지 않는 Object를 말한다. GC는 결국 이러한 Garbage가 메모리에 있는지 scan하고 만약 존재한다면 다시 회수하는 동작을 하는 기법이라고 말할 수 있다.

개념적으로는 정말 간단해 보이지만 구현이 정말 까다롭다. GC는 발전을 거듭해서 꽤나 괜찮은 성능을 보여주고 있는데 현재 Java JVM에서 사용되고있는 Generational Collection 방법까지 오면서 여러번 성능개선 작업이 이루어져왔다. 이번 포스팅에서는 발전순서대로 차근차근 알아볼 예정이다.

**발전 순서는 다음과 같다**

- Manual Reference Counting
- Mark and Sweep
- Copy Collection
- Generational Collection

## Manual Reference Counting & Automatic Reference Counting

우선 reference counter와 GC와의 관계에 대해 알 필요가 있을것같다. reference counter는 어떠한 객체가 있을 때 해당 객체를 참조하는 객체의 개수라고 생각하면 쉽다. 모든 객체는 자신의 reference counter를 저장하기위한 field가 존재한다 예제를 보자.

```java
A a = new A(); //reference counter = 1(생성시 자동으로 1로 초기화)
A b = a; //reference counter = 2;
```

위의 경우 a객체의 reference counter는 2가된다.

그렇다면 GC는 어떠한 객체를 garbage라고 판단하게 될까? 바로 reference counter가 0이된 객체를 garbage라고 여기고 reclaim할것이다.

위의 예제는 ARC(Automatic Reference Counting) 예시라고 볼 수 있다. 프로그래머가 따로 reference counter을 증가시키고, 감소시키는 코드가 보이지 않는다. 만약 MRC의 경우 어떻게될까? ARC가 해주는 자동 reference counting을 프로그래머가 직접 코드로 작성해주어야한다.

이는 code의 readability측면에서 매우 안좋은 영향을 주게되고 무엇보다...! 프로그래머가 매우 귀찮아지는일이 발생한다. 그래서 MRC 방식은 일부 legacy프로젝트 말고는 사용되는곳이 거의 없다고 봐도 될것같다. 그러나 ARC에는 **단점**이 존재하는데 Reference counting이 0이되는 object가 있는지 계속해서 확인하고 reclaim할 때 발생하는 오버헤드가 크다는점이다.

또한 아래 그림에서 볼 수 있듯 linkedList가 있다고 하고 프로그래머가 실수로 root node로부터 linkedList를 끊어버리는 경우가 있을 수 있는데, 이 경우 linked List에 node들은 연결된 상태가 여전히 유지되어있기 때문에 reference count가 0이 되지 않는 문제가 발생해서 reference counting방식으로는 안정적인 garbage collection을 보장해주지 못한다는 단점이 존재한다.

![GC structure.png](/assets/images/posts/2021-12-11/Untitled.png)

Reference Counting 방식의 장단점을 다시 정리해보자면 아래와 같다.

### 장점

- 구현이 쉬운편이다.
- 개념화하기 쉽다.

### 단점

- 모든 garbage objects에 대한 release를 보장해주지 못한다.
- 모든 오브젝트들에 대한 추가적인 오버헤드가 발생한다(ref_count)
- 오브젝트의 ref_count에 대한 동기화 처리가 필요하다
    - 다수의 thread가 하나의 오브젝트에 대해 접근할 때 lock을 걸어주어야하기 때문에 비용이 크다.

내용이 길어질 것 같아 여러 포스팅으로 나누어 mark & sweep, Copy collection, Generative Collection 개념을 차례대로 설명할 예정입니다!

> 해당 내용은 경북대학교 권영우교수님의 강의 내용을 정리한 내용임을 밝힙니다.
>
