---
title: "[JAVA] Privacy Leaks"

categories:
  - JAVA
  - Spring
tags:
  - private
  - JAVA
  - Spring
---

클래스를 정의할 때 외부에서 접근하지 못하도록 instance variables를 private로 둔다. 그러나 private로 설정한다고 해서 privacy leaks을 막을 수 없다. 클래스를 잘못 설계하면 private으로 설정한 instance variable도 값이 변경될 수 있기 때문이다.

# 잘못 설계된 클래스 예

```java
public class Person(){

	private String name;
	private Date born;
	private Date died;

	//생성자
	//setter

	//getter
	public Date getBirthDate()
	{
		return born;
	}
	//...
	private static boolean consistent(Date birthDate, Date deathDate)
	{
		if (birthDate == null)
			return false;
		else if (deathDate == null)
			return true;
		else
			return (birthDate.precedes(deathDate) ||
				birthDate.equals(deathDate));
	}
}
```

**위의 예에서 getBirthDate method를 살펴보자.**

해당 메소드는 단지 **born**이라는 인스턴스 변수를 return하고 있다. 이론적으로 born의 레퍼런스를 return하기 때문에 어딘가 다른 class에서 아래와 같이 코드를 작성하게되면 Person클래스 내부에서 분명 private으로 **born**을 설정하였지만 **getBirthDate** 메소드를 통해 리턴받은 Date type 레퍼런스에 의해 **setYear**메소드를 사용할 수 있게 되었고 그에 따라 결국 private로 선언된 **born**의 인스턴스 변수의 값이 변경될 수 있음을 보여준다.

```java
//Date class에는 instance variable로 String month, int day, int year 존재.
//모두 private으로 설정하였고, getter, setter 존재.
Person person = new Person(...);
Date bornDate = person.getBirthDate();
bornDate.setYear(1000);
```

그러나 이러한 걱정은 자바에 의해 덜 수 있다. 자바는 private로 선언된 instance varialbes에 대해 내부적으로 clone하여 return해주기 때문에 위의 예시같은 일은 발생하지 않는다. 하지만 레퍼런스를 리턴하는 경우 뿐만 아니고 다른 privacy leaks을 발생시킬 수 있는 상황들이 존재하는데, 이상하게 정의된 생성자와 setter가 존재할 때 발생한다. 아래의 예시를 살펴보자.

```java
//Person class내부에 정의된 메소드
//consistent 메소드는 Person class내부에 정의된 static method로 올바른 입력인지 검사한다.
public void setBirthDate(Date newDate){

	if (consistent(newDate, died))
		born = newDate;
	else
	{
		//에러메세지 출력
		System.exit(0);
	}
}
```

위와 같은 메소드가 있다고 했을 때 아래의 코드를 실행한다면?

```java
Person personObject = new Person("Josephine",
				new Date("January", 1, 2000), null);
Date dateName = new Date("Feburuary", 2, 2002);
personObject.setBirthDate(dateName);

dateName.setYear(1000); //문제 발생
```

**personObject**의 instance variable인 **born**에 클래스 타입이 **Date**인 **object(dateName)**의 레퍼런스가 들어가게된다. 따라서 **dateName.setYear(1000)**과 같이 메소드를 실행할 수 있게되고 이렇게되면 **Person**의 **setBirthDate**의 **consistent**메소드를 거치지 않으므로 유효하지 않은 입력이 들어갈 수 있게될 뿐만 아니라 instance variable가 예기치 못하게 변경될 수 있는 문제점이 발생한다. 이를 해결하기 위해서는 아래와 같이 아주 조금만 코드 변경을 해주면 된다~

```java
public void setBirthDate(Date newDate){
	if (consistent(newDate, died))
		born = new Date(newDate);	//코드 수정
	...
```

# 스프링에서는

Privacy Leaks을 공부하면서 의문이 들었다. 스프링에서는 아래와 같은 코드를 자주 접할 수 있었다.

```java
@Autowired
    public MemberServiceImpl(MemberRepository memberRepository){
        this.memberRepository = memberRepository;
    }
```

그러나 분명 위와 같이 코드를 짜면 privacy leaks이 발생하는것이 아닌가 하는 의문이 생겼다.

스프링 빈은 싱글톤으로 관리된다고 하였는데 이를 상기해보면 당연한 이치라고 생각된다. 왜냐하면 하나의 객체만 생성해서 공유하기 때문에 privacy leak이 발생할 수 밖에 없는 구조인것이다. 그러나 실제로는 무상태로 싱글톤을 설계한다면 privacy leak이 발생하지 않는 방향으로 갈 수 있지 않을까 생각한다...
