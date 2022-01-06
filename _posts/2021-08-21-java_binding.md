---
layout : posts
comments : true
title: "[JAVA] Binding"

categories:
  - JAVA
tags:
  - Binding
  - Late binding
  - Early binding
  - Dynamic binding
  - Polymorphism
---

## Binding이란

Method invocation과 method definition을 연관짓는 process.

## Binding 종류

- Early binding : 컴파일 타임에 binding이 일어남.
- Late binding : Run Time에 binding이  일어남.

## How late binding works?

### Note

설명하기 쉽도록 가정을 해보도록 하자. 우선 Figure라는 부모 클래스가 존재하고 자식클래스로는 Circle, Square, Triangle.. 여러 도형이 있을 수 있다고 가정한다. 그리고 Figure 클래스에는 center라고 하는 메서드 하나를 가지는데 화면상에서 좌표를 바꾸어 도형을 다시 그려주는 기능을 한다. 자식클래스에서는 draw메서드가 존재하는데 현재 가진 도형정보를 토대로 화면상에 그려주는 기능을 한다.

### 시나리오

`Rectangle` 그리고 `Circle`과 같은 클래스에서 상속받은 메서드 `center()`를 사용한다고 생각해볼 때 다형성에 대해 자세히 모른다면 의아해 할 수 있는 상황이 발생한다.

부모클래스에서 정의된 메서드 `center()`는 내부적으로 `draw()` 를 사용한다. 그리고 `draw()`는 자식 클래스에 따라 코드가 전부 다르다. 만약 JAVA에서 late binding을 사용하지 않고 **early binding**을 사용한다면 어떻게 될까?

우선 `center()`은 부모클래스에 정의 되어있다. 이것이 의미하는 바는 `draw()` code를 작성하기 전에 `center()`는 이미 컴파일이 되어있다고 봐도 무방하다. 문제가 되는 부분은 `draw()`일 것이다. 위에서 설명했듯이 자식 클래스 별로 `draw()`는 코드가 모두 다르기 때문이다.

early binding이 적용되면 현재 이용가능한 `draw()`의 정의로 고정될 수 밖에 없다. 따라서 모든 자식클래스의 `draw()`는 컴파일시에 이용된 method definition으로 고정되어 버리기 때문에 같은 동작을 수행할 수 밖에 없다. 반면 **lateing binding**을 사용한다면 함수 호출이 발생할 때 binding이 이루어 지기 때문에 각 클래스에 따른 적합한 method definition이 연관되어 올바른 연산이 가능해진다.

## 덧붙이자면...

***Polymorphism***과 *late binding*은 근본적으로 같은 개념을 나타내는 용어이다. *Polymorphism*은 *late binding*을 사용하여 하나의 메소드 이름에 다양한 의미를 부여하는 process를 지칭한다.
