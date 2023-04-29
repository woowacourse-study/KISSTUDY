이전 포스팅에서 빈을 등록할 때 @Component 어노테이션을 사용하는 경우에는 ComponentScan을 사용해서 등록해야한다고 했다. 사실 어노테이션 뿐만이 아니라 Java code로 Configuration을 등록하는 방법도 있는데, 이 때에도 ComponentScan을 사용할 수 있다.

[2023.04.24 - \[자바/스프링\] - 스프링 공식 문서 뿌수기(5), ApplicationContext란?](https://konghana01.tistory.com/601)

이번 포스팅에서는 ComponentScan에 대해서 알아보도록 하겠다. 

공식 문서 상에서도 내용이 그리 많지 않아서 아마 짧은 글이 되지 않을까싶다.

# ComponentScan

우선 ComponentScan이 무엇인지부터 알아보도록 하자. 언제나 그랬듯이 문서부터 먼저 확인해보자. 

가장 먼저 ComponentScan을 설명하는 문장은 다음과 같다.

#### @Configuration 어노테이션을 붙인 클래스의 컴포넌트를 스캔해서 구성한다.

여기서의 구성(Configuration)은 빈을 등록하는 과정을 의미한다고 생각하면 된다. 

Component Scan, 어떻게 사용해?

그 다음 문장으로도 설명이 되어있듯이 ComponentScan을 사용하는 방법에는 크게 두 가지가 있다.

**XML에서 <context:component-scan/> 요소**를 추가하는 것과 **@ComponentScan 어노테이션**을 붙이는 것이다. 둘 다 모두 지원하니 빈 등록하는 방법에 맞춰서 이를 사용하면 된다. 

즉, 내가 xml로 빈을 등록했다면 그냥 xml에 component-scan 요소를 추가해주고, 만약 xml을 사용하지 않고 빈을 등록했다면(annotation, java code) 이 @ComponentScan 어노테이션을 사용하면 된다. 

#### xml로 component scan을 등록하는 방법

```
<beans>
    <context:component-scan/>
</beans>
```

base-package를 사용해 Component로 등록하고자하는 패키지 경로를 설정할 수 있다.

```
<beans>
    <context:component-scan base-package="org.example"
        name-generator="org.example.MyNameGenerator" />
</beans>
```

#### @ComponentScan으로 component scan을 등록하는 방법

이 또한 마찬가지로 basePackages를 설정할 수 있다.

하지만 basePackages를 사용하는 경우에는 철자를 잘못 적어서 틀리는 경우가 발생할 수 있다.

따라서 basePackageClasses를 사용하면 클래스의 이름으로 빈을 등록할 수 있기 때문에 훨씬 안전하다. basePackageClasses은 이름 때문에 헷갈릴 수도 있지만 해당 클래스가 있는 패키지부터 하위 패키지까지 component scan을 한다는 의미이다.

위와 같이 **basePackage나 basePackageClasses를 사용한다면 해당 패키지부터 하위 패키지까지 스캐닝**할 수 있다.

## Component Scan을 사용한 bean 등록

Component scan을 사용해서 bean을 등록하는 경우에는 Spring에서 정해놓은 규칙에 따라 bean의 이름을 생성한다. 기본적으로는 클래스의 이름을 가져온 뒤 첫 글자를 소문자의 형태로 바꾼다.

#### 'Service -> service'

그런데, 두 개 이상의 문자가 있고 첫 번째와 두 번째 문자가 모두 대문자인 경우에는 원래 대소문자가 유지된다.

#### 'AProperty -> AProperty'

이 부분은 나중에 등록된 빈을 불러오는 동작과 같이 빈을 참조해야할 때 발생할 수 있는 실수이니까 주의해서 사용하면 좋을 것 같다!

## ComponentScan 주의사항

마지막으로 ComponentScan과 관련된 주의사항이 하나 있다. @Configuration을 사용해 ApplicationContext에 빈을 등록할 수 있는데, 이때 AnnotationConfigApplicationContext 생성자를 통해 ApplicationContext를 생성할 수 있다. 

AnnotationConfigApplicationContext를 사용해 ApplicationContext를 생성하는 경우에는 항상 @ComponentScan을 사용한 것과 같이 동작한다.

## 그런데...

그런데 스프링 부트를 사용하면 어플리케이션 클래스에 @SpringBootApplication 어노테이션이 붙는 것을 알 수 있다. 그리고 이는 @ComponentScan도 포함하고 있다.

따라서 우리가 스프링부트를 사용한다면 아직까지는 @ComponentScan을 따로 설정해줄 필요가 없지 않을까싶다!

[정리한 블로그 주소](https://konghana01.tistory.com/602)
