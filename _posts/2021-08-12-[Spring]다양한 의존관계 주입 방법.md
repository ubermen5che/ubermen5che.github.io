---
title: "[Spring] 다양한 의존관계 주입 방법"

categories:
  - Spring
tags:
  - Spring
  - DI
---

## 의존관계 주입은 크게 4가지 방법이 있다

- 생성자 주입
- 수정자 주입(setter 주입)
- 필드 주입
- 일반 메서드 주입

## 생성자 주입

- 생성자를 통해서 의존 관계를 주입 받는 방법이다.
- 특징
    - 생성자 호출시점에서 딱 1번만 호출되는 것이 보장된다.
    - 불편, 필수 의존관계에 사용
        - ex) MVC패턴에서 Repository, Service등과 같이 필수적으로 사용되는 객체들
        - 거의 대부분의 서비스에서 생성자 주입방법을 사용한다.
        - 왜냐하면 웹 서비스를 공연으로 비유할 수 있는데, 공연이 진행되는 도중에 배우를 바꾸는 경우는 잘 없기 때문이다.

```java
@Component
public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy rateDiscountPolicy){
        this.memberRepository = memberRepository;
        this.discountPolicy = rateDiscountPolicy;
    }
}
```

⭐️ **생성자가 딱 1개만 있으면 @Autowired를 생략해도 자동 주입된다.(물론 스프링 빈에만 해당)**

## 수정자 주입(setter 주입_

> setter라 불리는 필드의 값을 변경하는 수정자 메서드를 통해서 의존관계를 주입하는 방법.

### 특징

- 선택, 변경 가능성이 있는 의존관계에 사용
- 자바빈 프로퍼티 규약의 setter 메서드 방식을 사용하는 방법.

```java
@Component
public class OrderServiceImpl implements OrderService{

    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;

		@Autowired
		public void setMemberRepository(MemberRepository memberRepository){
			this.memberRepository = memberRepository;
		}

		@Autowired
		public void setDiscountPolicy(DiscountPolicy discountPolicy){
			this.discountPolicy = discountPolicy;
		}
}
```

> 참고: @Autowired의 기본 동작은 스프링 컨테이너에서 해당 빈을 탐색하고 탐색결과 주입할 대상이 없으면 오류가 발생한다. 주입할 대상이 없어도 동작하게 할려면 @Autowired(required = false)로 지정하면 된다.

## 필드 주입

> 필드에 바로 의존관계를 주입하는 방법.

### 특징

- 코드가 간결해서 많은 개발자들을 유혹하지만 외부에서 변경이 불가능해서 테스트 하기 힘들다는 치명적인 단점이 존재.
- DI 프레임워크가 없으면 아무것도 할 수 없다.
- 사용하지 않는것이 좋지만 예외적으로 사용함.
    - 애플리케이션의 실제 코드와 관계 없는 테스트 코드
    - 스프링 설정을 목적으로 하는 @Configuration 같은 곳에서만 특별한 용도로 사용

```java
@Component
public class OrderServiceImpl implements OrderService{

		@Autowired
    private MemberRepository memberRepository;
		@Autowired
    private DiscountPolicy discountPolicy;

}
```

> 참고: 순수한 자바 테스트 코드에는 당연히 @Autowired가 동작하지 않는다. @SpringBootTest처럼 스프링 컨테이너를 테스트에 통합한 경우에만 사용가능.

> 다음 코드와 같이 @Bean에서 파라미터에 의존관계는 자동 주입된다. 수동 등록시 자동 등록된 빈의 의존관계가 필요할 때 문제를 해결할 수 있다.



```java
@Bean
OrderService orderService(MemberRepository memberRepository, DiscountPolicy discountPolicy){
		new OrderServiceImpl(memberRepository, discountPolicy)
}
```

## 일반 메서드 주입

> 일반 메서드를 통해서 주입 받을 수 있다.

### 특징

- 한번에 여러 필드를 주입 받을 수 있다.
- 일반적으로 잘 사용하지 않는다.

```java
@Component
public class OrderServiceImpl implements OrderService{

    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;

    @Autowired
    public void init(MemberRepository memberRepository, DiscountPolicy rateDiscountPolicy){
        this.memberRepository = memberRepository;
        this.discountPolicy = rateDiscountPolicy;
    }
}
```
