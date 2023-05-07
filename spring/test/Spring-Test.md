👉 [가독성이 많이 떨어질 것 같아서 블로그 링크를 첨부합니다 ㅠㅠ](https://cl8d.tistory.com/82)

---

### ✔️ 통합 테스트 vs 인수 테스트

- **통합 테스트**
    - 특정 작업을 수행하기 위해, 외부 작업들과 연관되어 있다면 해당 외부 작업들을 포함하여 구성 요소가 잘 돌아가는지 테스트하는 방법
    - ex) @SpringBootTest 어노테이션 이용
- **인수 테스트**
    - 사용자의 시나리오에 맞춰 수행하는 테스트
    - E2E 테스트는 주어진 시나리오에 따라 애플리케이션의 모든 계층이 서로 잘 동작하는지 확인하는 테스트
    - 인수 테스트를 위해서 E2E 테스트가 하나의 방법론으로 사용될 수 있다.
    - ex) RestAssured 이용


---

### ✔️ 테스트 프레임워크

스프링은 테스트에 사용되는 애플리케이션 컨텍스트를 생성하고 관리하고, 테스트에 적용해주는 기능을 가진 ‘테스트 프레임워크’를 제공한다. 그리고 이를 ‘테스트 컨텍스트 프레임워크’라고 부른다.

### 컨텍스트 관리 및 캐싱

- 테스트 대상 메서드에 @Test를 붙이면 해당 메서드가 속한 클래스는 ‘테스트 클래스’로서 동작하며, 각 메서드는 독립적인 테스트가 된다.
- 하지만, 테스트마다 테스트 컨텍스트를 매번 새로 생성하게 된다면 오버헤드가 크기 때문에 전체 테스트의 실행 속도가 느려져셔 개발자의 생산성이 떨어진다.
- 그렇기 때문에 테스트 컨텍스트는 자신이 담당하는 테스트 인스턴스에 대한 컨텍스트 관리 및 캐싱 기능을 제공한다.
    - 만약 특정 애플리케이션 컨텍스트를 구성하고 싶다면, 클레스 레벨에 @ContextConfiguration 어노테이션을 선언함으로서 구성 정보를 주입해줄 수 있다.

---

### ✔️ 컨텍스트 설정 정보 주입하기

- @ContextConfiguration 어노테이의 옵션으로 설정 정보가 담긴 클래스를 주입하면, 해당 설정 정보를 이용한 테스트 클래스를 구성할 수 있다.

```java
@ExtendWith(SpringExtension.class) // Spring TextContext Framework를 Junit5에 포함시키는 어노테이션
@ContextConfiguration(classes = {AppConfig.class, TestConfig.class}) 
class MyTest {

}
```

- 위와 같이 설정하게 되면 AppConfig, TestConfig 클래스를 바탕으로 ApplicationContext가 구성된다.

```java
@Configuration
public class Hello {
    @Bean
    public Nested nested() {
        return new Nested();
    }

    @Component
    static class Nested {

    }
}
```

- 위와 같이 nested class로 선언되어 있는 빈을 등록한다고 생각해보자.

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = {Hello.class})
public class Sample {

    @Autowired
    private Nested nested;

    @Test
    void test() {

    }
}
```

- 그럼 위와 같이 Hello라는 설정 정보 클래스를 ContextConfiguration으로 불러오게 되면 등록되어 있는 Nested라는 빈에 대해 의존관계를 주입받을 수 있게 된다.
- 만약 해당 어노테이션을 제거하면 아래와 같이 컴파일 오류가 발생한다.

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbevYsQ%2Fbtsd3edNK7t%2Fg8ARhxwZK1fJrmZZ2BCqjK%2Fimg.png">

---

### ✔️ 나는 지금까지 컨텍스트 정보를 지정한 적이 없는데?

- @ConfigurationContext를 지정하지 않았음에도 정상적으로 테스트가 동작한 경험이 있을 것이다.

<img src = "https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FIYOT9%2FbtsecuGEzyA%2FpiBzda1Wi0wCbBiDNUeCt0%2Fimg.png">


기본적으로 설정하지 않으면 위와 같이 예외가 발생한다.
하지만, 스프링 부트를 사용하고, @SpringBootApplication 어노테이션이 붙은 클래스가 존재하는 하위 패키지에 테스트를 두었다면, 알아서 적절한 정보를 찾게 된다. (그래서 @SpringBootApplication의 경우 보통 패키지의 최상단에 두는 게 일반적이다.)

<img src = "https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbYLCgc%2Fbtseh7YtSII%2F0WQf6XL9jNAStrlTsZpFq1%2Fimg.png">

실제로 보면 알아서 @SpringBootConfiguration을 찾게 되는데, 이는 패키지 최상단에 위치한 @SpringBootApplication이 붙은 클래스를 찾게 되는데, @SpringBootApplication의 경우 내부적으로 @SpringBootConfiguration이 설정되어 있기 때문에 에러 없이 동작한 것이다.

---

### ✔️ 컨텍스트 캐싱

- 테스트 컨텍스트 프레임워크는 테스트에 대한 ApplicationContext를 로드하면, 해당 컨텍스트가 캐싱되어 동일한 테스트 컨텍스트 구성을 사용하게 되면 해당 컨텍스트를 재사용하게 된다.
- 보통 ApplicationContext의 경우 이를 로드하기 위한 configuration parameter의  조합을 통해 고유하게 식별이 가능하다.
- configuration parameter로는 다음과 같은 정보들을 사용한다.
    - locations (@ContextConfiguration 어노테이션의 인자)
    - classes (@ContextConfiguration 어노테이션의 인자)
    - contextInitializerClasses (@ContextConfiguration 어노테이션의 인자)
    - contextCustomizers (ContextCustomizerFactory로부터 로드됨)
    - contextLoader (@ContextConfiguration 어노테이션의 인자)
    - parent (@ContextHierarchy)
    - activeProfiles (@ActiveProfile)
    - propertySourceLocations (@TestPropertySource)
    - propertySourceProperties (@TestPropertySource)
    - resourceBasePath (@WebAppConfiguration)
- 테스트 컨텍스트 프레임워크는 애플리케이션 컨텍스트를 정적 변수로 저장하며, 만약 테스트 자체가 별도의 프로세스에서 실행되는 경우 각 테스트 실행 사이의 정적 캐시가 지워지기 때문에 캐싱 작업 역시 함께 비활성화 된다.

---

### ✔️ @DirtiesContext

- 만약 강제로 컨텍스트르 제거하고 싶다면 클래스, 혹은 메서드 레벨에 @DirtiesContext 어노테이션을 추가하면 된다. 
- 이러면 동일한 애플리케이션 컨텍스트를 사용하더라도 테스트 실행 전에 강제적으로 기존 컨텍스트를 제거하고 새로운 컨텍스트를 실행해준다.

```java
/**
	새로운 스프링 컨테이너에서 실행해야 하는 경우
*/

// 현재 테스트 클래스 이전에 컨텍스트를 더티 처리
@DirtiesContext(classMode = BEFORE_CLASS) 
class FreshContextTests {
}

// 각각의 테스트 메서드 전에 컨텍스트를 더티 처리
@DirtiesContext(classMode = BEFORE_EACH_TEST_METHOD)
class FreshContextTests {
}

// 현재 테스트 메서드 이전의 컨텍스트를 더티 처리
@DirtiesContext(methodMode = BEFORE_METHOD) 
@Test
void testProcessWhichRequiresFreshAppCtx() {
}

/**
	스프링 컨텍스트를 더럽히는 동작을 수행할 때
*/
// 현재 클래스 이후에 컨텍스트를 더티 처리
@DirtiesContext 
class ContextDirtyingTests {
}

// 각각의 테스트 메서드 후에 컨텍스트를 더티 처리
@DirtiesContext(classMode = AFTER_EACH_TEST_METHOD) 
class ContextDirtyingTests {
}

// 현재 테스트 메서드 이후의 컨텍스트를 더티 처리
@DirtiesContext 
@Test
void testProcessWhichDirtiesAppCtx() {
}
```

---

### ✔️ @SpringBootTest

위에서 언급했던 스프링 프레임워크 표준 어노테이션인 @ContextConfiguration 대신 @SpringBootTest 어노테이션을 활용할 수 있다. 
@SpringBootTest 애노테이션의 경우 기본적으로는 서버를 시작하지 않지만, webEnvironment 옵션을 통해서 어떤 식으로 테스트 환경을 조성할지 설정할 수 있다.

- MOCK (기본값)
  - ApplicationContext을 로드라고 가짜 웹 환경을 구성한다. 
  - 내장 서버가 시작되지 않으며, mock-based의 웹 애플리케이션의 테스트를 하기 위해서 @AutoConfigureMockMvc 또는 @AutoConfigureWebTestClient와 함게 사용할 수 있다.


- RANDOM_PORT
  - WebServerApplicationContext를 로드하고, 실제 웹 환경을 제공한다. 내장 서버가 실행되고, 임의의 포트에서 실행된다.
  - 만약, 사용한 포트 정보가 궁금하다면 @LocalServerPort 어노테이션을 통해서 해당 포트 정보를 얻어올 수 있다. 
    - @Value("${local.server.port}")와 완전히 동일한 역할을 한다.

- DEFINED_PORT
  - WebServerApplicationContext를 로드하고 실제 웹 환경을 제공하며, 내장 서버가 실행되지만 application.yml에 지정한 포트나 기본 포트인 8080을 사용한다.

- NONE
  - ApplicationContext를 로드하지만 웹 환경을 구축하지 않는다.

```
⭐️ 만약, @Transactional을 테스트 메서드에 적용할 경우, 각 테스트 메서드가 끝날 때 롤백된다. 
하지만 RANDOM_PORT, DEFINED_PORT를 통해 실제 웹 환경을 사용하게 되면 트랜잭션이 잡힌 테스트 메서드의 스레드와 웹 서버의 스레드가 다르기 때문에 
별도의 트랜잭션에 실행되어 테스트 메서드에서 실행된 메서드의 경우 트랜잭션 롤백이 되지 않는다.
````

---

### ✔️ @WebMvcTest

Spring MVC 인프라를 자동을 구성하고, 웹 계층에서 사용하는 컴포넌트들을 스캔한다.
이때 스캔되는 컴포넌트는 @Controller, @ControllerAdvice, @JsonComponent, Converter, GenericConverter, 
Filter, HandlerInterceptor, WebMvcConfigurer, WebMvcRegistrations 및 HandlerMethodArgumentResolver 등이 사용되며, 
우리가 흔히 사용하는 @Service, @Component, @Repository가 붙은 클래스들은 스캔 대상이 아니다.

### ✔️ @JdbcTest
@JdbcTest 어노테이션의 경우 기본적으로 내당 데이터베이스 및 JdbcTemplate 빈을 구성한다.
DB 레이어를 테스트할 때 많이 사용하며, 내부적으로 @Transactional 어노테이션이 선언되어 있기 때문에 클래스 레벨에 붙이면 각 테스트 메서드가 종료될 때 함께 롤백된다.

###  💬 @DataJdbcTest
@DataJdbcTest의 경우 @JdbcTest와 거의 비슷하긴 하다.
이는 Spring Data JDBC repositories를 사용하는 테스트에서 사용한다.
