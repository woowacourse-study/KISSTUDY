# DispatcherServlet

## 목차

1. 간단한 개념 정리
    1. Servlet, Servlet Container의 간단한 개념
    2. Servlet LifeCycle
    3. Servlet Container의 스레드
2. DispatcherServlet 개념
    1. DispatcherServlet의 개념, 장점 → Front Controller, 어떻게 등록 되는지, ServletContainer와의 구조
    2. 전체적인 흐름
3. DispatcherServlet 디테일
    1. DispatcherServlet 상속 구조
    2. DispatcherServlet의 WebApplicationContext 구조
    3. ~~처리 흐름 소스코드로 따라가보기… 는 정리하기가 너무 힘들어서 우선 생략했습니다~~
        - ~~Interceptor가 언제 동작하는가 → preHandle, postHandle~~
        - ~~ArgumentResolver가 언제 동작하는가~~
        - ~~ReturnValueHandler가 언제 동작하는가~~
    4. 자주 사용하는 HandlerAdapter, HandlerMapping 구현체
    

#### 마크다운으로 깔끔하게 정리가 안돼서 [노션 주소](https://somsom13.notion.site/DispatcherServlet-15712ae1d3744f1c82e24a6a433a4f85)도 같이 첨부하겠습니다! 

---

## Servlet 관련 간단 개념 정리

### 1. Servlet이란?

```
💡 Servlet은 Java 웹 프로그래밍 기술로, Request를 받아 이에 따른 작업을 처리하고, Response를 보내는 역할을 하는 클래스이다.
-> 동적 자원 처리를 위한 것
```

ex) HTML form에서 사용자 입력을 받고 → 이에 관련된 데이터를 DB에서 조회 → 데이터로 구성된 동적 웹을 구성, 반환의 역할을 Servlet이 한다.

Servlet 인터페이스를 구현하는 다양한 구현체들이 존재하며, 그 중 하나가 HttpServlet 이다.

- HttpServlet은 Servlet의 기능에 HTTP 관련 지원이 추가된 구현체이다.
- doGet, doPost …

### 2. Servlet Container?

Serlvet 객체는 스스로 동작하지 않기 때문에 누군가가 제어를 해주어야 하는데, **Servlet을 제어하는 역할을 하는 것이 바로 ServletContainer이다.**

- 톰캣(Tomcat)이 가장 대표적인 서블릿 컨테이너이다.

![image](https://user-images.githubusercontent.com/70891072/236691762-494791df-f927-4e4c-9042-68541bea66fd.png)

1. WAS의 Web Server가 Request를 받으면, 이를 Servlet Container에게 전달한다.
2. 서블릿 컨테이너는 해당 요청을 처리할 서블릿 인스턴스를 찾는다. (만약 인스턴스가 없다면 생성하고 초기화한다.)
3. 서블릿의 service() 메서드를 호출해서 요청을 처리하고, 응답을 Web Server에게 반환한다.

ServletContainer는 Servlet을 생성하고, 관리하는 역할을 한다.

- **ServletContainer가 하는 일**
    - Servlet 생성, 관리
    - Servlet LifeCycle 관리
    - Web Server와의 통신 지원
    - 멀티쓰레드를 통한 다중 요청 처리 지원
    - ServletRequest, ServletResponse 객체 생성

### 3. Servlet Life Cycle (생명 주기)

**init()**

- 서블릿이 생성될 때, 딱 한 번만 호출된다.
- 첫 요청이 클라이언트로 부터 왔을 때, 이 요청을 처리할 수 있는 서블릿 인스턴스가 메모리에 없다면
    1. Servlet 클래스를 로드한다.
    2. Servlet 인스턴스를 생성한다.
    3. init() 메서드를 통해 해당 인스턴스를 초기화한다.
- Servlet이 요청을 받기 전에 반드시 초기화가 완료되어 있어야 한다. 그렇지 않으면, Servlet Container가 서블릿을 service 상태로 둘 수 없다. (요청을 처리할 수 있는 상태로 둘 수 없다.)

**service()**

- Servlet이 초기화 된 후에만 실행할 수 있는 메서드
- 클라이언트가 보내는 Request를 처리하기 위해 Servlet Container가 Servlet의 Service를 호출한다.
    - `service(RequestServlet, ResponseServlet)`
    - **HttpServlet의 경우,** `request.getMethod()` 를 통해 요청 타입을 분석 → 내부에서 doGet, doPost 등의 메서드를 호출한다.

**destroy()**

- 서블릿 컨테이너가 서블릿의 service를 종료하기 위해 호출한다.
- 모든 스레드가 servlet의 service 메서드를 빠져나왔을 때, 혹은 일정 시간이 지난 후에 딱 한 번 호출된다.
- 한 번 destroy가 되면 servlet은 더 이상 service 메서드가 호출되지 않는다.

### 4. Servlet Container의 스레드

**Servlet 객체는 한 서블릿 컨테이너에 한 개씩만 존재한다. (즉 싱글톤으로 관리된다.)**

서블릿 컨테이너는 요청이 오면 해당 요청을 처리할 스레드를 생성한다. (이 때, 스레드 관리에 서블릿 컨테이너 내부의 스레드 풀을 사용한다.)

서블릿 컨테이너가 자체적으로 각 요청들을 모두 다른 스레드에서 처리하기 때문에, 개발자가 멀티 스레딩을 고려하지 않아도 된다. 

---

## DispatcherServlet 개념

### 1. DispatcherController의 개념, 구조

대부분의 웹 프레임워크는 Front Controller 패턴으로 디자인 되어있다.

- Front Controller: 가장 먼저 Request를 받는 컨트롤러를 의미한다. 요청을 받고, 이를 처리할 수 있는 핸들러를 찾아서 해당 핸들러에게 요청을 위임한다.
- 보통 MVC는 Servlet을 기반으로 구현이 되기 때문에, FrontController 역할을 하는 객체가 서블릿인 경우가 많다.

**Spring MVC에서도 DispatcherServlet이 Front Controller의 역할을 한다.**

> DispatcherServlet이 Request에 대한 공통적인 처리를 진행하고, 실제 비즈니스 로직은 해당 요청을 처리할 수 있는 개별 handler에 의해 실행된다.
> 

![image](https://user-images.githubusercontent.com/70891072/236691787-9866cc99-b5e9-4a52-880c-c39847f9ddb7.png)

**DispatcherServlet의 장점**

모든 요청을 DispatcherServlet이 받고, 내부에서 알아서 이를 처리할 controller를 찾아 책임을 위임해주기 때문에 개발자가 별도로 해당 부분을 신경쓰지 않아도 된다.

또한, web.xml에 모든 URL을 등록하고 해당 URL을 처리하는 Servlet을 구현해야 했던 과거와 다르게 이제는 DispatcherServlet 하나만 있으면 내부적으로 URL에 대한 controller를 매핑해준다.

![image](https://user-images.githubusercontent.com/70891072/236691799-2077f8b6-ed0d-4688-8769-4da46b55d465.png)

클라이언트로부터 요청을 받고, 처리하는 전체적인 구조는 위와 같다.

Servlet Container는 요청을 처리할 수 있는 Servlet 인스턴스들을 내부에서 관리하며, 요청이 오면 이를 처리할 수 있는 Servlet에게 위임한다.

- DispatcherServlet은 FrontController로서 가장 먼저 Request를 받는다. 그리고 해당 요청을 처리할 수 있는 handler에게 위임한다.
    - DispatcherServlet에 Request가 가기 전에 Filter를 먼저 거치긴 한다.
- 그 외의 다른 Tomcat Servlet으로는 JSP Servlet, Default Servlet, Error Handling Servlet 등이 있다. (이들은 Spring이 관리하는 Servlet이 아님)

**DispatcherServlet의 등록**

DispatcherServlet도 다른 서블릿들 처럼, Java Configuration이나 web.xml을 통해 선언되고, URL과 매핑되어야 한다.

```xml
// web.xml DispatcherServlet 등록 방식
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>
        org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern> // "/" URL 패턴으로 들어오는 요청들을 DispatcherServlet이 받게 된다.
</servlet-mapping>
```

> Spring Boot uses Spring configuration to bootstrap itself and the embedded Servlet container. `Filter` and `Servlet` declarations are detected in Spring configuration and registered with the Servlet container.
> 

> By default, if the context contains only a single Servlet, it is mapped to `/`.
> 

스프링 공식 문서에는 위와 같이 “SpringBoot는 자동으로 DispatcherServlet을 등록, 구성하기 때문에 수동으로 등록하지 않아도 된다.” 라고 이야기한다.

스프링 부트 공식 문서에 따르면 서블릿이 단 하나만 존재하는 경우, 해당 서블릿이 처리할 수 있는 URL Pattern은 `/` 으로 등록이 된다. 

- 즉 모든 주소와 매칭이 된다는 의미

### 2. DispatcherServlet의 동작 과정

![image](https://user-images.githubusercontent.com/70891072/236691818-365933df-e52c-46e6-bd86-03174cfd1e9d.png)

1. 요청을 가장 먼저 DispatcherServlet이 받는다.
2. DispatcherServlet이 해당 요청을 위임할 Handler를 찾는다. (by. HandlerMapping)
3. handler를 찾았다면, 요청을 진짜로 Handler에게 위임해줄 역할을 하는 HandlerAdapter를 찾는다.
4. Interceptor, ArgumentResolver 등의 전처리 후, HandlerAdapter가 controller에게 요청을 넘겨준다.
5. 컨트롤러가 자신의 비즈니스 로직을 수행한다.
6. 컨트롤러가 수행 결과를 반환한다.
7. HandlerAdapter가 controller의 결과를 후처리하고, 이를 다시 DispatcherServlet으로 반환한다.
8. 서버가 클라이언트로 응답을 반환한다.
<br><br>

1. **요청이 들어오면, 이를 DispatcherServlet이 받는다.**
    1. 가장 먼저 HttpServlet의 *service(ServletRequest, ServletResponse)* 가 호출된다.
        - ServletRequest, ServletResponse는 Servlet Container에서 생성된다.
    2. *service* 메서드 내에서 HttpRequest의 타입을 분석, 이에 맞는 메서드를 호출한다.
        - doGet, doPost 등등..
    3. doXXX 메서드를 처리하는 과정에서 요청을 처리할 Handler를 찾는 작업이 시작된다.
        - LocaleResolver, ThemeResolver, FlashMapManager 등의 전처리가 진행되지만 요청 흐름 처리의 핵심은 아니니 생략…ㅎㅎ
        
2. **요청을 위임할 Handler를 찾는다.**
    
    > Handler: 스프링에서 웹 요청을 처리하는 객체를 큰 범위에서 부르는 용어.
    Controller는 Handler의 일종이다.
    우리는 Controller를 사용한 핸들링을 공부하기 때문에 Controller == Handler로 생각해도 무방할 것 같다.
    > 
    1. Request에 담긴 정보를 사용해서 handlerMapping 객체로부터 handler를 찾는다. 
        
        스프링에서는 총 5개의 HandlerMapping을 검사하면서 요청을 처리할 수 있는 Handler를 찾는다.
        
    2. Handler가 찾아지면, handler 객체 + 이와 연관된 Interceptor들이 담긴 **HandlerExecutionChain** 이 반환된다.
    
3. **요청을 핸들러로 넘겨주는 역할을 수행할 HandlerAdapter를 찾고, 요청과 핸들러를 전달한다.**
    - DispatcherServlet이 Handler를 찾아도, 즉시 Request를 Handler에게 전달하지 않는다.
    - **실제로 Request를 Handler에게 위임하는 것은 HandlerAdapter의 역할이다**.
        - 이유: 컨트롤러의 구현 방식이 다양하기 때문
        - 지금은 Annotation Based Controller를 사용하지만, 과거에는 Controller 인터페이스를 구현해서 controller를 구현하는 방법을 사용했다.
        - 다양한 방식으로 구현된 Controller에 모두 대응하기 위해 HandlerAdapter로 구현에 상관없이 요청을 위임할 수 있도록 한 것이다.
    - HandlerAdapter도 마찬가지로 Dispatcher가 기본적으로 가지고 있는 총 4개의 HandlerAdapter들 중에서, 해당 Handler를 처리할 수 있는 adapter를 찾는다.
    
4. **HandlerAdapter가 Handler(Controller)에게 요청을 위임한다.**
    1. 이 때, handler에게 요청을 넘기기 전에 공통적인 전처리 작업을 수행한다.
    2. Interceptor의 실행 → *handlerAdapter.handle()* → ArgumentResolver를 통한 객체의 변환 → *doInvoke(파라미터들)*
    3. 전처리가 모두 완료되면, 컨트롤러의 파라미터들과 함께 실제 컨트롤러 메서드가 호출된다. (by. Reflection)
    - HandlerAdapter 내부에 ArgumentResolver, ReturnValueHandler, MessageConverter 들을 가지고 있다.
    
5. **요청을 위임받은 controller가 자신의 비즈니스 로직을 수행한다.**
    
    
6. **Controller가 반환값을 반환한다.**
    - View 이름을 나타내는 문자열 + 값이 담긴 Model → 응답 페이지를 보여주는 경우
    - ResponseEntity → JSON 응답인 경우
    
7. **HandlerAdapter가 Controller의 반환값을 처리한다.**
    - ReturnValueHandler를 통해서 HandlerAdapter가 controller 반환값을 후처리, DispatcherServlet에게 전달한다.
    - ResponseEntity인 경우, 객체를 직렬화 하여 JSON으로 변환한다.
    - View의 이름을 나타내는 문자열인 경우, View를 반환하기 위한 준비 작업을 처리한다.
    
8. **서버의 응답을 클라이언트로 반환한다.**
    - 응답이 데이터라면 그대로 반환된다.
    - 응답이 화면이라면 ViewResovler가 이름에 맞는 적절한 화면을 반환해준다.
    

---

## DispatcherServlet 디테일

### 1. DispatcherServlet의 상속 구조

```plainText
💡 DispatcherServlet → FrameworkServlet → HttpServletBean → HttpServlet → GenericServlet → Servlet
위의 구조로 상속된다.

DispatcherServlet의 코드를 분석할 때는 다음 메서드들을 위치를 파악해두시면 아마 더 편하게 분석하실 수 있을 것 같습니다… 하하

- DispatcherServlet: **doService()** → doDispatch() → getHandler() → getHandlerAdapter() → processDispatchResult()

- FrameworkServlet: **service()**, processRequest()

- HttpServlet: service() → public, protected 순으로 **(가장 먼저 요청을 받는 메서드!)**

```

- HttpServlet: 기본 Servlet에 HTTP 요청 처리와 관련된 기능들이 추가된다.
    - 단, PATCH 메서드를 처리하는 기능은 구현되어 있지 않다.
    - 이는 PATCH 메서드가 뒤늦게 공식적으로 추가된 메서드이기 때문이다.
- HttpServletBean: Spring이 HttpServlet 추상클래스를 구현한 것
- FrameworkServlet: 웹 프레임워크의 기반이 되는 서블릿
    - doXXX 메서드 오버라이딩 → doXXX 메서드가 항상 내부에서 processRequest() 를 호출 → 내부에서 doService 호출
- DispatcherServlet: Front Controller Pattern으로 구현된 서블릿. doService가 구현되어 있다.
    - Handler를 찾고, 요청을 위임하는 doDispatch 메서드가 doService 내부에서 호출된다.

### 2. DispatcherServlet의 Context Hierarchy(컨텍스트 계층)

DispatcherServlet은 자신만의 WebApplicationContext(ApplicationContext를 상속)를 필요로 한다.

![image](https://user-images.githubusercontent.com/70891072/236692343-daf014fd-03c3-4812-bdd9-15358d8faa02.png)

위의 구조처럼 1 DispatcherServlet - 1 Servlet WebApplicationContext를 가진다.  

그리고 Servlet이 여러개인 경우, 여러 Servlet들은 내부에 Servlet Context를 가지면서 하나의 Root ApplicationContext를 공유한다.

- Servlet WebApplicationContext
    - 1 Servlet  - 1 Servlet WebApplicationContext
    - DispatcherServlet이 직접 사용하는 bean들을 관리한다.
        - controller, handlerMapping 등
- Root WebApplicationContext
    - Infrastructure Bean들을 관리한다.
        - repository, 비즈니스 로직을 가지는 service 등
    - 즉, 여러 servlet들에 의해 공유되어야 하는 bean들을 관리한다.

이러한 구조를 선택한 이유는 **웹 기술에 의존적인 부분과 아닌 부분을 구분하기 위해** 라고 한다.

### 3. 우리가 주로 사용하게 될 HandlerMapping, HandlerAdapter 구현체

**HandlerMapping: RequestMappingHandlerMapping**

- `@Controller`, `@RequestMapping` 어노테이션이 붙어있는 컨트롤러는 해당 HandlerMapping에 의해 handler로 찾아진다.

**HandlerAdapter: RequestMappingHandlerAdapter**

- `@RequestMapping` 이 붙은 메서드들을 처리한다.
- 또한, 해당 어노테이션이 붙은 메서드들이 필요로 하는 ArgumentResolver, ReturnValueHandler들을 내부에 가지고 있다
    
    ![image](https://user-images.githubusercontent.com/70891072/236692359-92c83cfd-fe8c-4ae5-8be6-4509f6d6efea.png)

- HttpEntityMethodProcessor: ResponseEntity를 처리하는데 사용하는 returnValueHandler
    - 내부에 MessageConverter들을 가지고 있고, 그 중 MappingJackson2HttpMessageConverter가 자바 객체 → JSON 직렬화를 하는 converter이다.
- RequestResponseBodyMethodProcessor:
    - `@RequestBody` 어노테이션이 붙은 파라미터는 JSON → 자바 객체로 역직렬화를 해준다. (resolveArgument)
    - `@ResponseBody` 어노테이션이 붙은 반환값을 handle해준다. (handleReturnValue)
