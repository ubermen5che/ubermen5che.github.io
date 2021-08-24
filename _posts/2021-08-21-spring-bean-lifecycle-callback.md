---
title: "[Spirng]빈 생명주기 콜백"

categories:
  - JAVA
  - Spring
tags:
  - Bean
  - Bean Lifecycle
  - Callback
---

데이터베이스 커넥션 풀이나, 네트워크 소켓처럼 애플리케이션 시작 시점에 필요한 연결을 미리 해두고, 애플리케이션 종료 시점에 연결을 모두 종료하는 작업을 진행하려면, 객체의 초기화와 종료 작업이 필요하다. 이번 글에서는 스프링을 통해 이러한 초기화 작업과 종료 작업을 어떻게 진행하는지 예제로 알아보고자 한다.

> 참고: 애플리케이션 시작 시점에 DB 커넥션 풀과 연결을 미리 해두는 이유는 애플리케이션 서버와 데이터베이스간의 연결 과정에서 시간이 소요되기 때문이다.

## 예제 코드

```java
public class NetworkClient {

    private String url;

    public NetworkClient() {
        System.out.println("생성자 호출, url = " + url);
        connect();
        call("초기화 연결 메세지");
    }

    public void setUrl(String url){
        this.url = url;
    }

    public void connect(){
        System.out.println("connect: " + url);
    }

    public void call(String message){
        System.out.println("call: " + url + " " + "message = " + message);
    }

    public void disconnect(){
        System.out.println("close: " + url);
    }
}
```

위의 예제코드는 Application server에서 외부DB에 연결을 하는 과정을 가상으로 시뮬레이션하는 과정을 나타낸다. 아래는 코드는 테스트 코드이다.

```java
public class BeanLifeCycleTest {

    @Test
    public void lifeCycleTest(){
        ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
        NetworkClient client = ac.getBean(NetworkClient.class);
        ac.close();
    }

    @Configuration
    static class LifeCycleConfig {
        @Bean
        public NetworkClient networkClient(){
            NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("http://hello-spirng.dev");
            return networkClient;
        }
    }
}
```

실행해보면 다음과 같은 이상한 결과가 나온다.

```bash
생성자 호출, url = null
connect: null
call: null message = 초기화 연결 메세지
```

생성자 부분을 보면 url정보 없이 connect가 호출되는 것을 확인할 수 있다. 이는 당연한 결과로 써 setUrl 메서드로 url를 설정해주어야만 url이 존재하게된다. 그런데 여기서 의문을 가질 수 있다. 왜냐하면 생성자를 호출할 때 url를 같이 넘겨주면 위와 같은 이상한 결과를 얻지는 않을텐데 왜 위와 같은 코드를 사용할까?

이에 대한 답은 아래의 참고사항으로 적어두었으니 읽어보기를 권한다.

> 스프링 빈은 간단하게 다음과 같은 라이플사이클을 가진다.

- **객체 생성 → 의존관계 주입**

스프링 빈은 객체를 생성하고, 의존관계 주입이 다 끝난 다음에 필요한 데이터를 사용할 수 있는 준비가 완료된다. 따라서 **초기화 작업은 의존관계 주입이 모두 완료되고 난 다음에 호출해야 한다.** (위의 예제를 보아도 그렇다는것을 알 수 있다) 그런데 개발자가 의존관계 주입이 모두 완료된 시점을 어떻게 알 수 있을까？

**스프링은 의존관계 주입이 완료되면 스프링 빈에게 콜백 메서드를 통해서 초기화 시점을 알려주는 다양한 기능을 제공한다.** 또한 스프링은 스프링 컨테이너가 종료되기 직전에 소멸 콜백을 준다. 따라서 안전하게 종료 작업을 진행할 수 있다. 다양한 기능이 어떤것이 있는지는 다음번에 알아보도록 하고 스프링 빈의 이벤트 라이프사이클에 대해 알아보자.

### 스프링 빈의 이벤트 라이프사이클

**스프링 컨테이너 생성 → 스프링 빈 생성 → 의존관계 주입 → 초기화 콜백 → 사용 → 소멸전 콜백 → 스프링 종료**

- **초기화 콜백** : 빈이 생성되고, 빈의 외존관계 주입이 완료된 후 호출
- **소멸전 콜백** : 빈이 소멸되기 직전에 호출

스프링은 다양한 방식으로 생명주기 콜백을 지원한다.

> **참고:객체의 생성과 초기화를 분리하자**
생성자는 필수 정보(파라미터를)받고, 메모리를 할당해서 객체를 생성하는 책임을 가진다. 반면에 초기화는 이렇게 생성된 값들을 활용해서 외부 커넥션을 연결하는등 무거운 동작을 수행한다. 따라서 생성자 안에서 무거운 초기화 작업을 함께 하는 것 보다는 객체를 생성하는 부분과 초기화 하는 부분을 명확하게 나누는 것이 유지보수 관점에서 좋다. 물론 초기화 작업이 내부 값들만 약간 변경한느 정도로 단순한 경우에는 생성자에서 한번데 다 처리하는게 더 나을 수 있다.

> 참고: 싱글톤 빈들은 스프링 컨테이너가 종료될 때 싱글톤 빈들도 함께 종료되기 때문에 스프링 컨테이너가 종료되기 직전에 소멸전 콜백이 일어난다. 싱글톤 처럼 컨테이너의 시작과 종료까지 생존하는 빈도 있지만, 생명주기가 짧은 빈들도 있는데 이 빈들은 컨테이너와 무관하게 해당 빈이 종료되기 직전에 소멸전 콜백이 일어난다.

## 빈 생명주기 콜백 3가지 방법

- 인터페이스(InitializingBean, DisposableBean)
- @Bean 설정 정보에 초기화 메서드, 종료 메서드 지정
- @PostConstruct, @PreDestroy 애노테이션 지원

### 인터페이스(InitializingBean, DisposableBean)

- 스프링 전용 인터페이스. 해당 코드가 스프링 전용 인터페이스에 의존한다.
    - implements를 통해 구현체 코드를 작성해 주어야한다.
    - 따라서 초기화, 소멸 메서드의 이름을 변경할 수 없다.
    - 내가 코드를 고칠 수 없는 외부 라이브러리에 적용할 수 없다.

> 인터페이스를 사용하는 초기화, 종료 방법은 스프링 초창기에 나온 방법들이고, 지금은 더 나은 방법들이 있어서 거의 사용하지 않는다.

## @Bean 설정 정보에 초기화 메서드, 종료 메서드 지정

### 특징

- 설정 정보에 @Bean(initMethod = "init", destroyMethod = "close")처럼 초기화, 소멸 메서드를 지정할 수 있다.
- 메서드 이름을 자유롭게 줄 수 있다.
- 스프링 빈이 스프링 코드에 의존하지 않는다.
- 코드가 아니라 설정 정보를 사용하기 때문에 코드를 고칠 수 없는 외부 라이브러리에도 초기화, 종료 메서드를 적용할 수 있다.
    - 즉, 외부 라이브러리 자체에서 내부적으로 초기화, 종료에 대한 메서드를 가지고 있고 이에 해당하는 메서드 명을 설정정보로 넘겨주면 되는것이다.

### 종료 메서드 추론

- `@Bean destroyMethod` 속성에는 아주 특별한 기능이 존재.
- 라이브러리 대부분 `close`, `shutdown`이라는 이름의 종료 메서드를 사용한다.
- @Bean의 destroyMethod는 기본값이 (inferred)으로 등록되어 있다.
- 이 추론 기능은 close, shutdown이라는 이름의 메서드를 자동적으로 호출해준다.
- 따라서 직접 스프링 빈으로 등록하면 종료 메서드는 따로 적어주지 않아도 잘 동작한다.

## @PostConstruct, @PreDestroy 애노테이션 방법

## 예제 코드

```java
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

public class NetworkClient {

    private String url;

    public NetworkClient() {
        System.out.println("생성자 호출, url = " + url);
        connect();
        call("초기화 연결 메세지");
    }

    public void setUrl(String url){
        this.url = url;
    }

    public void connect(){
        System.out.println("connect: " + url);
    }

    public void call(String message){
        System.out.println("call: " + url + " " + "message = " + message);
    }

    public void disconnect(){
        System.out.println("close: " + url);
    }

    @PostConstruct
    public void init(){
        System.out.println("NetworkClient.init");
        connect();
        call("초기화 연결 메세지");
    }

    @PreDestroy
    public void close(){
        System.out.println("NetworkClient.destroy");
        disconnect();
    }
}
```

## 특징

- 최신 스프링에서 가장 권장하는 방법.
- 애노테이션 하나만 붙이면 되므로 매우 편리.
- 패키지를 잘 보면 javax.annotation.PostConstruct이다. 스프링에 종속적인 기술이 아니라 JSR-250이라는 자바 표준이다. 따라서 스프링이 아닌 다른 컨테이너에서도 동작함.
- 컴포넌트 스캔과 잘 어울린다.
- 유일한 단점은 외부 라이브러리에는 적용하지 못한다는 것. 외부 라이브러리를 초기화, 종료 해야하면 @Bean의 기능을 사용하자.
