## DI (의존관계 주입)

**🌱 의존한다?**

- A가 B를 사용하고, B를 변경하면 A에 영향을 끼치는 관계.

```
Dependency Injection is a fundamental aspect of the Spring framework, 
through which the Spring container “injects” objects into other objects or “dependencies”. 
Simply put, this allows for loose coupling of components and moves the responsibility of managing components onto the container.
```

- 의존관계 주입이란, Spring 컨테이너가 다른 객체나 어떠한 종속성에 따라 객체를 주입하는 것을 의미하며, Spring Container가 사용하는 기본 전략이다.
- 각 객체간의 결합도를 낮춰주고, 컨테이너에게 각 객체를 관리하는 책임을 위임할 수 있다.

## **IoC (제어의 역전)**

- 오브젝트의 생성과 관계 설정, 사용, 제거 등의 작업을 외부로 위임하는 것.

### **IoC container (Spring Container)**

<img src="https://github.com/woowacourse-study/KISSTUDY/blob/main/spring/core/ioccontainer/image.png">

- 스프링 프레임워크에서 IoC를 통해 Object를 관리하기 위해 사용하는 개념
- BeanFactory / ApplicationContext 인터페이스
    - ApplicationContext가 BeanFactory를 상속받고 있기 때문에 일반적으로 더 많이 사용한다.
- IoC container에 등록된 Bean에 대해서 DI를 내부적으로 관리
    - 이때, 컨테이너에 등록하기 위해 개발자가 제공하는 정보를 ‘metadata’라고 표현
    - XML 기반 / 어노테이션 기반 구성으로 나누어진다.

---

## 스프링 컨테이너의 **라이프사이클은 어떻게 되는가?**

⭐️ **생성과 초기화를 분리하는 것이 핵심 역할이다**.

**🌱 빈이란?**

- Spring IoC 컨테이너에 의해서 인스턴스화 되고, 관리되는 객체이다.

### 스프링 컨테이너 생성

- 말 그대로 스프링 컨테이너가 생성되는 과정이다.

### **스프링 빈 생성**

**@Configuration +** **@Bean**

- 개발자가 컨트롤할 수 없는 외부 라이브러리들을 Bean으로 등록할 때 사용
- @Bean의 경우 메서드에 대해서만 지정이 가능하다.

@**Component**

- 개발자가 컨트롤할 수 있는 클래스에 대해서 사용
- Component의 경우 클래스에 대해서만 지정이 가능하다.
- @ComponentScan 어노테이션에 의해서 @Component 및 스테레오 타입 (@Controller, @Service, @Repository) 어노테이션이 부여된 클래스들을 자동으로 스캔하여 빈으로 등록한다.

### **의존 관계 주입**

- 생성자에 지정된 @Autowired를 보고, 스프링 컨테이너가 자동으로 빈을 찾아 조회한다.
    - 의존 관계 주입 시 ‘**타입이 같은 빈**’을 찾아서 조회한다.
    - 그래서 동일한 타입이 있다면 @Primary나 @Qualifier 어노테이션을 사용하여 어떤 빈을 우선으로 주입할지 지정해야 한다.

### **초기화 콜백**

- 스프링은 의존관계 주입이 완료되면, 스프링 빈에게 콜백 메서드를 통해서 어떠한 초기화 기능을 할 수 있도록 제공한다.

**InitializingBean** 

- InitializingBean 인터페이스의 afterProperiesSet() 메서드를 통해 초기화를 지원할 수 있다.
- 스프링 전용 인터페이스이기 때문에, 스프링에 의존적인 형태가 된다. (잘 사용하지 않는 형태)

**빈 설정 정보에 초기화 메서드 지정**

- @Bean(initMethod=””)을 통해서 초기화 메서드를 지정할 수 있다.
- 설정 정보를 활용하는 방법이기 때문에 수정할 수 없는 외부 라이브러리에 대해서도 초기화 메서드를 지정할 수 있다.

**⭐️ @PostConstruct 사용**

- 가장 권장하는 방법이다. 어노테이션 하나만 붙이면 돼서 간단하다.
- 자바 표준 기술이기 때문에 스프링에 종속적이지 않다.
- 외부 라이브러리에는 적용하지 못한다.

- 위의 방법들을 혼용하여 사용한다면, 다음과 같은 순서로 호출된다.

@PostConstruct → afterPropertiesSet() → @Bean(initMethod=””)

### 실제로 사용하기

- 우리가 실제로 애플리케이션 내에서 비즈니스 로직을 구성하며 사용하는 단계이다.

### **소멸 전 콜백**

- 싱글톤 빈들은 스프링 컨테이너 종료 시 빈들도 함께 소멸하기 때문에, 소멸전 콜백이 발생한다.

**DisposableBean 사용**

- DisposableBean 인터페이스의 destroy() 메서드를 통해 소멸을 지원할 수 있다.

**빈 설정 정보에 소멸 메서드 지정**

- @Bean(destroyMethod=””)을 통해서 소멸 메서드를 지정할 수 있다.
- 설정 정보를 활용하는 방법이기 때문에 수정할 수 없는 외부 라이브러리에 대해서도 소멸 메서드를 지정할 수 있다.
    - 참고로, 외부 라이브러리를 대부분 close, shutdown이라는 이름으로 종료 메서드를 사용하고 있다. destroyMethod는 내부적으로 추론 기능이 있기 때문에 스프링 빈으로 등록했다면 종료 메서드를 따로 작성하지 않아도 알아서 동작한다.
    

**⭐️ @PreDestroy 사용**

- @PostConstruct와 동일한 특징을 가지고 있다.

- 위의 방법들을 혼용하여 사용한다면, 다음과 같은 순서로 호출된다.

@PreDestroy → destroy() → @Bean(destroyMethod=””)

### 스프링 컨테이너 종료

---

## **빈의 스코프는 무엇인가?**

- 스코프는 ‘빈이 존재할 수 있는 범위’를 의미한다.
- 스프링에서는 총 6가지의 스코프를 제공한다
    - @Scope 어노테이션을 이용하여 지정할 수 있다.

### **싱글톤 스코프**

- 스프링에서 기본적으로 사용하는 스코프이다.
- 스프링 컨테이너가 시작될 때 생성되어서, 종료될 때까지 유지한다. (가장 넓은 범위)
- IoC 컨테이너에서 조회하게 되면 항상 같은 인스턴스의 빈을 반환한다.

### **프로토타입 스코프**

- IoC 컨테이너는 빈의 생성과 의존 관계, 초기화까지만 관여하고, 그 이상 관리하지 않는 스코프이다.
    - 종료 콜백 메서드는 호출되지 않는다.
- IoC 컨테이너에 조회 시 때 빈을 생성하고, 필요한 의존관계를 주입하기 때문에 매번 새로운 인스턴스를 반환한다.

⭐️ **싱글톤 스코프 with 프로토타입 스코프**

- 싱글톤 스코프를 가진 빈은 컨테이너 생성 시점에 생성되고, 의존관계 주입이 발생한다. 만약 주입 시 필요한 빈이 프로토타입 빈이라면, 컨테이너는 이를 받아 새로운 인스턴스를 생성하여 반환한다.
- 그러면 싱글톤 빈은 여전히 프로토타입에 대한 빈의 의존 관계를 가지고 있기 때문에, 참조 관계를 보관하고 있다. 단, 과거에 주입이 끝난 상태로 반환받은 것이기 때문에 사용한다고 해서 새롭게 생성되는 것은 아니다.
- 만약, 싱글톤 빈을 사용할 때 내부적으로 프로토타입 빈을 사용하는 로직을 호출했다면 이미 주입이 끝난 시점이기 때문에 새로운 빈이 아닌 기존의 빈을 계속해서 사용하는 형태로 사용하게 된다. (=즉, 싱글톤 빈과 생명주기가 동일해진다)
- 이를 해결하기 위해서는 ObjectProvider를 사용할 수 있다. (컨테이너에서 조회하기 때문에 새로운 인스턴스를 생성하기 때문에 프로토타입의 기능을 사용할 수 있게 됨)

### **request 스코프**

- HTTP 요청과 동일한 생명주기는 스코프이다. HTTP 요청마다 인스턴스가 생성되고 관리되는 형태이다.

### **session 스코프**

- Http 세션과 동일한 생명주기를 가지는 스코프이다.

### **application 스코프**

- 서블릿 컨텍스트와 동일한 생명주기를 가지는 스코프이다.
- 과거에는 스프링 컨테이너를 여러 개 띄웠기 때문에 각 컨테이너마다 관리되는 빈을 다르게 하기 위해서 사용했었다. (지금은 하나의 스프링 컨테이너를 사용해서 처리하는 것이 대부분이다.)

### **websocket 스코프**

- 웹 소켓과 동일한 생명주기를 가지는 스코프이다.

**⭐️ 싱글톤 스코프 with 웹 스코프**

위의 4가지 경우를 웹 스코프라고 한다.

싱글톤 빈의 경우 스프링 컨테이너와 라이프 사이클이 동일하기 때문에 컨테이너 생성 시 같이 생성되지만, 웹 스코프의 경우 각각의 요청이 들어왔을 때 빈이 생성되기 때문에, 만약 싱글톤 빈이 웹 스코프 빈을 의존 관계로 가지고 있을 때 의존 관계 주입이 불가능해진다.

이를 위해서 2가지 방법을 사용할 수 있다.

1. ObjectProvider 사용하기
    1. getObject()를 호출하는 시점까지 웹 스코프 빈의 생성 지연
2. 프록시 사용하기
    1. @Scope 어노테이션의 인자로 proxyMode = ScopedProxyMode.TARGET_CLASS 지정 (클래스일 경우, 인터페이스라면 INTERFACES 사용)
    2. 이러면 가짜 프록시 객체를 주입해주기 때문에 상관이 없어진다.
    3. 가짜 프록시 객체의 경우, 요청이 왔을 때 진짜 빈으로 요청을 위임한다.

---

## **모든 객체를 스프링 빈으로 등록해도 괜찮을까?**

공식 문서에서는 아래와 같은 답변을 내두었다.

```
These bean definitions correspond to the actual objects that make up your application. 
Typically, you define service layer objects, persistence layer objects such as repositories or data access objects (DAOs), 
presentation objects such as Web controllers, infrastructure objects such as a JPA `EntityManagerFactory`, JMS queues, and so forth. 
Typically, one does not configure fine-grained domain objects in the container, 
because it is usually the responsibility of repositories and business logic to create and load domain objects.
```

- 빈에 대한 정의는 애플리케이션을 구성하는 객체들로 이루어져 있다.
- 일반적으로 service layer, 레포지토리나 DAO 같은 persistence layer, 컨트롤러 같은 presentation layer, 혹은 infrastructure layer 등에서 사용되는 객체를 정의한다.
- 하지만, 도메인 객체를 생성하고 로드하는 것은 ‘레파지토리 및 비즈니스 로직의 책임’이기 때문에 IoC 컨테이너에서 이를 구성하는 것은 일반적이지 않다고 하였다.

---

## **의존성을 주입하는 방법에는 무엇이 있는가?**

- 의존 관계 주입은 크게 4가지 방법으로 나누어진다.

⭐️ 스프링 컨테이너가 관리하는 스프링 빈이어야만 의존관계 주입이 동작한다. (컴파일 시점에 해당하는 타입이 없으면 오류가 발생한다)

### 생성자 주입

- 생성자를 통해서 의존 관계를 주입받는 방법
- 생성자 호출 시점에 딱 1번만 호출된다.
    - 만약 필드를 private final로 선언했다면 (일반적) 불변 및 생성에 대한 보장을 해줄 수 있다. (Spring 팀에서 권장하는 주입 방식)
    - 컴파일 시점에 값이 설정되지 않았다면 오류가 발생해서 막아준다.
- 만약 생성자가 1개라면 자동으로 @Autowired가 붙은 것과 동일한 효과를 주기 때문에 생략이 가능하다.

⭐️ 기본적으로 스프링은 빈을 등록하는 단계와 의존관계를 주입하는 단계가 나누어져 있지만, **생성자 주입의 경우 빈을 등록하는 단계와 의존관계 주입이 동시에 일어난다**.

- 빈을 등록할 때 생성자를 호출해야 하기 때문에 의존 관계도 함께 주입되기 때문.
- 생성자 주입과 수정자 주입을 혼용하더라도 동시에 일어난다. (생성자 주입 이후 setter 주입 진행)

### 수정자 주입

- setter를 통해 의존 관계를 세팅해주는 방법
- 생성자 주입을 통해 의존 관계가 주입되었더라도, 변경하고 싶을 때 사용 가능
- 수정자에 @Autowired를 붙여서 주입

⭐️ 몇몇 글에서는 수정자 주입, 필드 주입을 진행하면 순환참조 문제를 잡을 수 없다고 (실제로 사용해야 잡힌다) 말하지만, 스프링 2.6부터는 순환 참조를 기본으로 안 되도록 변경했기 때문에 런타임 시점에 서버가 실행되지 않는다.

### 필드 주입

- 필드에 @Autowired를 선언하여 바로 주입하는 방법.
- 코드는 간결하지만, 외부에서 변경이 불가능하기 때문에 테스트하기 어려워진다.
- DI 프레임워크에 의존적인 형태이다.

### 일반 메서드 주입

- 일반 메서드를 통해서 주입하는 방법.
    - 수정자 주입처럼 setXXX의 형태여야 하지만, 수정자 주입과 다르게 여러 개의 파라미터를 넘길 수 있다.
- 일반적으로 잘 사용하지 않는 방법이다.


---

### **Reference**

[Core Technologies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-definition)

[스프링 핵심 원리 - 기본편 - 인프런 | 강의](https://www.inflearn.com/course/스프링-핵심-원리-기본편/dashboard)

[토비의 스프링 정리 프로젝트 #1.7 의존관계 주입(DI)](https://velog.io/@jakeseo_me/토비의-스프링-정리-프로젝트-1.7-의존관계-주입DI)
