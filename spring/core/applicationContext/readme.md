스프링 공식 문서 뿌수기 시리즈를 하고 있던 중 Annotated Controller 파트 초반에 ApplicationContext와 관련된 내용이 등장했다.

[2023.04.17 - \[자바/스프링\] - 스프링 공식 문서 뿌수기(2), Spring MVC의 Annotated Controllers - 1](https://konghana01.tistory.com/592)

 [스프링 공식 문서 뿌수기(2), Spring MVC의 Annotated Controllers - 1

지난주 금요일 우테코 레벨 2 피드백 강의 시간에 브리가 각 미션마다 크루들이 공부했으면 하는 커리큘럼에 대해 알려줬다. 그 내용을 하나씩 정리해보려고 한다. 우선 스프링의 가장 기본적인

[konghana01.tistory.com](https://konghana01.tistory.com/592)

당시 bean을 ApplicationContext에 등록해서 auto-detection에 사용한다고 설명했었다. 이번 포스팅에서는 ApplicationContext가 무엇인지 조금 더 자세히 알아보도록 하겠다. 

# IoC Container

ApplicationContext가 무엇인지 알아보기 이전에 IoC Container라는 개념부터 간단하게 짚어보고자 한다.

## IoC란?

#### IoC란 Inversion of Control의 약자로 **제어의 역전**을 의미한다.

우리가 생각하는 제어의 역전이 맞다. 제어의 역전이란 개발자가 작성한 객체나 메서드 등에 대한 제어를 외부에 위임하는 것을 의미한다. 

제어의 역전을 활용한 가장 대표적인 예시가 **프레임워크**이다. 예를 들어서 라이브러리를 사용한다는 것은 코드 내에서 라이브러리를 직접 사용해서 어플리케이션을 만드는 것에 초점을 두고 있는 것이다. 반면에 프레임워크를 사용한다는 것은 우리의 코드가 프레임워크의 일부분이 되어서 어플리케이션을 구성한다. 이러한 제어의 역전을 사용하는 경우에는 개발자는 핵심 비즈니스 로직에 더 집중할 수 있다는 장점이 있다. 

스프링에서는 이와 같은 IoC를 위해 DI(의존성 주입)를 사용한다. 의존성 주입을 사용함에 따라서 비즈니스 도메인의 의존성을 외부로부터 주입받게 된다. 이때 외부에서 의존성을 주입해주는 주체가 바로 IoC Container가 된다.

설명을 두루뭉실하게 표현하니까 잘 이해가 안가는 것 같다. 어떤 객체를 어떻게 어디에 주입해준다는 걸까?

그건 이제부터 알아보도록 하겠다.

## BeanFactory

BeanFactory라는 인터페이스는 위에서 설명한 IoC Container의 역할을 한다. 이를 자세하게 설명하기 위해선 Bean이라는 개념에 대해서 이해하고 있어야하는데, Bean이란 간단하게 말해서 **Spring에서 IoC Container가 관리하는 대상 객체**를 의미한다고 생각하면 된다. 이전에 포스팅에서 @Component라는 어노테이션에 대해 설명했는데, **@Component 어노테이션을 사용하면 해당 클래스 객체가 빈으로 등록**이 된다. 이렇게 빈으로 등록되면 나중에 의존성 주입의 대상이 될 수 있다. 뭐... 싱글톤 빈이니 프로토타입 빈이니, 빈에 대한 각종 유형이 있지만 지금은 ApplicationContext를 정리하고 있으니 이에 대해선 나중에 따로 글을 포스팅하도록 하겠다.

BeanFactory는 이러한 빈들에 대한 생성과 관리를 하는 역할을 한다고 생각하면 된다. 그런데 Spring을 사용하는 우리는 BeanFactory를 직접 사용하고 있지는 않다. 우리는 BeanFactory를 확장한 **ApplicationContext**를 사용한다. 


## ApplicationContext

ApplicationContext는 각 구성요소(Bean)에 접근하기 위해 BeanFactory를 상속받았다고 이야기하고 있다.

우리가 BeanFactory를 직접 사용하기보단 ApplicationContext를 사용하는 이유는 Bean을 관리하는 기능 외에도 부가적인 기능들이 더 있기 때문이다.

공식 문서에서는 Spring AOP 기능을 조금 더 쉽게 통합하거나 국제화를 위한 메세지 리소스 핸들링 등이 가능하다고 한다. 

즉, ApplicationContext를 사용하는 경우에는 BeanFactory를 사용하는 것보다 더 많은 부가기능을 사용할 수 있는 것이다. 

#### 따라서 일반적으로 BeanFactory가 아닌 BeanFactory를 상속한 `ApplicationContext`를 사용한다.

## BeanFactory or ApplicationContext

공식 문서에서는 이를 조금 더 자세히 비교해가면서 이야기하고 있다. 결론부터 말하자면 특수한 상황이 아닌 경우에는 ApplicationContext를 사용하는 것이 좋다고 한다. 

GenericApplicationContext나 AnnotationConfigApplicationContext를 커스텀 부트스트래핑을 공통적으로 구현할 때 사용하지 않는 이상 ApplicationContext를 사용하는 것이 좋다고 한다. 무슨 말인지 모르니까 넘기도록 하자. 어쨋든 중요한 포인트는 ApplicationContext를 사용하는 것을 권장한다는 점이다. 어차피 ApplicationContext는 BeanFactory의 주요한 기능을 모두 포함하고 있기 때문에 권장한다고 이야기 한다.  

## ApplicationContext Configuration Metadata

공식문서에서는 ApplicationContext가 빈을 정의하는 방법에 대해서도 설명하고 있다. 

ApplicationContext는 **Configuration 메타 데이터를 읽어서 이를 빈으로 등록**하는 과정을 거치는데, 이러한 Configuration 메타 데이터를 만드는 방법은 네 가지가 있다.

#### (1) XML Configuration

가장 먼저 XML로 이를 구성하는 방법이 있다. 

다음은 공식문서에 나온 XML로 Configuration 메타데이터를 설정하는 방법의 예시이다.

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="..."> (1) (2)
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

<bean>이라는 태그를 사용해서 id와 class를 전달하는데, 여기서 id는 빈들을 식별하는 문자열을 의미한다. class는 빈 타입을 의미하는 것으로, 다시 말해 **경로를 포함한 클래스명**을 작성하면 된다.

예를 들어 다음과 같은 태그가 있다고 하자.

#### <bean id = "gameDao" class = "racingcar.dao.GameDao" />

소스 코드에서 해당 빈을 가져오거나 참조하고 싶을 때에는 'gameDao'를 식별자로서 사용하면 된다.

그래서 getBean("gameDao", GameDao.class)와 같은 코드를 작성하면 위의 빈을 참조할 수 있게 된다. 

위와 같이 XML을 사용해서 메타데이터를 만드는 방법도 있지만,  Annotation이나 Java code를 사용해서도 빈을 정의할 수 있다.

#### (2) Annotation based Configuration

말 그대로 어노테이션을 사용해서 bean을 정의하는 방법이다. 빈으로 등록할 클래스의 이름 위에는 @Component 어노테이션을 붙이고, 의존성 주입을 할 대상에는 @Autowired 어노테이션을 붙여서 사용하면 된다. 개인적으로 이번 웹 자동차 경주를 구현하면서 빈을 등록하기 위해서 사용한 방법이었다.

이때 설정 파일에서 **ComponentScan을 설정해야지 어노테이션을 스캔해서 빈들을 등록**할 수 있다. 웹 자동차 경주 미션을 진행할 때에는 main 함수를 실행하는 클래스에서 @SpringBootApplication 어노테이션을 사용하는데, 이 어노테이션에는 @ComponentScan이 있어서 빈들을 자동으로 등록해준다.

---

#### (3) Java based Configuration

@Configuration 어노테이션을 붙인 새로운 설정 파일을 만들면 된다.

클래스를 정의한 부분 위에 @Configuration을 사용할 수 있다. 또한 각 메서드마다 @Bean 어노테이션을 붙이면 된다.(홍고의 configuration을 보면 자세히 나와있다.)

그리고 아래와 같이 AnnotationConfigApplicationContext를 사용해서 등록한 빈들을 확인할 수 있다.

이 또한 마찬가지로 @ComponentScan 어노테이션을 사용하면 Component를 사용한 클래스를 자동으로 빈으로 등록해준다. 

흐음... ComponentScan이 자주 등장한다. 로드맵에도 있었는데... 다음 포스팅으로 한번 다뤄보도록 하겠다.

---

#### (4) Groovy Bean 

공식문서에서는 곁다리 느낌으로 설명하고 있는 예시이다. 일단 그리 중요하지 않은 것 같으니 이는 예시만 보고 넘기기로 하자.

```
beans {
    dataSource(BasicDataSource) {
        driverClassName = "org.hsqldb.jdbcDriver"
        url = "jdbc:hsqldb:mem:grailsDB"
        username = "sa"
        password = ""
        settings = [mynew:"setting"]
    }
    sessionFactory(SessionFactory) {
        dataSource = dataSource
    }
    myService(MyService) {
        nestedBean = { AnotherBean bean ->
            dataSource = dataSource
        }
    }
}
```

## 빈 등록 방식이 다양한건 알겠는데, 어떤걸 쓰는게 좋아?

공식 문서도 이에 대해서 답변을 주고 있다. 

결론부터 말하자면 개발자가 알아서 판단하고 본인의 주관대로 사용하면 된다고 한다. XML을 사용하는 경우에는 **빈이 달라지더라도 소스 코드를 건드리지 않고 다시 컴파일하지 않는다는 장점**이 있다. 반면 어노테이션을 사용하는 경우에는 더 짧고 간결하게 작성할 수 있다. @Configuration을 사용해 값을 주입하는 경우에는 비즈니스 도메인을 직접 수정하지 않고도 해당 파일만 건드려서 빈을 다시 설정할 수 있는 장점이 있다. 

개발자 입장에서의 이런 장단점 말고도 차이점이 있긴 한데, 어노테이션을 사용한 주입은 XML의 주입보다는 먼저 실행된다는 점이다. 만약 어노테이션과 XML을 모두 사용해서 빈을 구성했다면, annotation으로 정의한 빈을 XML이 덮어씌워서 재정의할 수 있다는 점을 주의하자.

뭐... 내가 지금 느끼기에는 각각 큰 차이는 없어서 개발하는 환경의 컨벤션에 맞춰서 개발하면 좋을 것 같다.

# 이것저것 너무 많이 들어왔어...! 정리해줘~

#### Spring은 IoC(제어의 역전)을 활용해 개발자가 핵심 비즈니스 로직에만 집중할 수 있도록 돕는다.

#### Spring에서는 IoC를 위해 DI(의존성 주입)를 사용한다.

#### Bean은 DI의 대상이 되는 객체를 의미한다. 의존성을 주입하는 대상이 될 수도 있고, 의존성 주입을 받는 대상이 될 수도 있다.

#### BeanFactory는 이러한 Bean을 생성하고 관리하는 역할을 한다.

#### 우리는 일반적으로 BeanFactory를 사용하지 않고 여기서 몇 가지 부가기능(국제화 메세지, 리소스 로더 등등)이 더 추가된 ApplicationContext를 사용한다. 

#### ApplicationContext에서 Bean을 등록하기 위해서는 크게 세 가지 방법을 많이 사용한다.(XML, Annotation, Java code)

위에 있는 과정을 통해 Bean을 등록하면 ApplicationContext가 bean을 자동으로 주입해준다. 따라서 개발자가 신경쓸 부분이 많이 줄어든다!!
