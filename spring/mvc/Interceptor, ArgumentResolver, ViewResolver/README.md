# 스프링 Interceptor, ****ArgumentResolver, ViewResolver****

## 목차

1. 사용 목적 및 사용법
    - Interceptor란?
        - 사용목적
        - 사용법 + 추가설명(인터셉터의 간단한 동작 구조)
        - 실제코드
    - ArgumentResolver란?
        - 사용목적
        - 간단한 동작 과정 → 사용법
        - 실제코드
    - ViewResolver란?
2. 전체 구조와 작동 과정(HttpServletRequest)

---

# Interceptor란?

## 사용목적

사용자 요청에 의해 서버에 들어온 Request 객체를 컨트롤러의 핸들러(사용자가 요청한 url에 따라 실행되어야 할 메서드, 이하 핸들러)로 도달하기전에 낚아채서 개발자가 원하는 추가적인 작업을 한후 핸들러로 보낼수 있도록 해주는것이 인터셉터 이다.

컨트롤러(Controller)의 '핸들러(Handler)'를 호출하기 전과 후에 요청과 응답을 참조하거나 가공할수 있는 일종의 필터이다.

즉, **특정 Controller의 핸들러가 실행되기 전이나 후에 추가적인 작업을 원할때 Interceptor를 사용한다.**

대체로, 

- 보안 및 인증(로그인)/인가(권한) 공통 작업
- controller로 넘겨주는 정보(데이터)의 가공을 담당한다.

### 

## 사용방법

방법1 ) org.springframework.web.servlet의 `**HandlerInterceptor**` 인터페이스를 구현해야한다.

```java
public interface HandlerInterceptor {

    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
        throws Exception {
        
        return true;
    }

    default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
        **@Nullable ModelAndView modelAndView**) throws Exception {
    }

    default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
        @Nullable Exception ex) throws Exception {
    }
}
```

해당 인터페이스는 다음과 같은 3가지 메소드를 가지고 있다.

### 1. preHandle 메소드

: preHandle 메소드는 컨트롤러가 호출되기 전에 실행된다.

- 사용 목적 : 컨트롤러 메서드 실행 이전에 처리해야 하는 **전처리 작업**이나 **요청 정보를 가공하거나 추가하는 경우에 사용**한다.
- preHandle의 3번째 파라미터인 handler 파라미터는 **핸들러 매핑이 찾아준 컨트롤러 빈에 매핑**되는 HandlerMethod라는 새로운 타입의 객체로써, **@RequestMapping이 붙은 메소드의 정보를 추상화한 객체**이다.
- preHandle의 반환 타입은 boolean인데,
    - 반환값이 true이면 다음 단계로 진행이 되지만,
    - false라면 작업을 중단하여 이후의 작업(다음 인터셉터 또는 컨트롤러)은 진행되지 않는다.

### 2. postHandle 메소드

: postHandle 메소드는 컨트롤러를 호출된 후에 실행된다. 

- 사용 목적: 컨트롤러 이후에 처리해야 하는 후처리 작업이 있을 때 사용할 수 있다.
- 핸들러가 실행은 완료 되었지만 **아직 View가 생성되기 이전에 호출된다.**
- 이 메소드에는 컨트롤러가 반환하는 ModelAndView 타입의 정보가 제공되는데, Controller에서 View 정보를 전달하기 위해 작업한 Model 객체의 정보를 참조하거나 조작할수 있다.
    
    그러나, 최근에는 Json 형태로 데이터를 제공하는 **RestAPI 기반의 컨트롤러(@RestController)를 만들면서 자주 사용되지는 않는다**.
    
- 비동기적 요청처리 시에는 처리되지않음. ???
- 컨트롤러 하위 계층에서 작업을 진행하다가 중간에 예외가 발생하면 postHandle은 호출되지 않는다.

### 3. afterCompletion 메소드

: afterCompletion 메소드는 이름 그대로 모든 뷰에서 최종 결과를 생성하는 일을 포함해 모든 작업이 완료된 후에 실행된다. 요청 처리 중에 사용한 리소스를 반환할 때 사용하기에 적합하다. postHandler과 달리 컨트롤러 하위 계층에서 작업을 진행하다가 중간에 예외가 발생하더라도 afterCompletion은 반드시 호출된다.

방법2) 또는, **org.springframework.web.servlet.handler.HandlerInterceptorAdapter** 추상클래스를 오버라이딩 해야한다.

참고로, HandlerInterceptorAdapter 추상클래스 경우 HandlerInterceptor 인터페이스를 구현했다.

### +) 추가 설명

- Spring이 제공하는 기술로써, 디스패처 서블릿(Dispatcher Servlet)이 컨트롤러를 호출하기 전과 후에 **요청과 응답을 참조하거나 가공할 수 있는 기능을 제공**한다.
- 웹 컨테이너(서블릿 컨테이너)에서 동작하는 필터와 달리 **인터셉터는 스프링 컨텍스트에서 동작한다**.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c35cc7e6-46ad-45cb-af89-bef8e82f2d3a/Untitled.png)

- 작동과정
    1. 디스패처 서블릿은 핸들러 매핑을 통해 적절한 컨트롤러를 찾도록 요청한다.
    2. 그 결과로 실행 체인(HandlerExecutionChain)을 돌려준다. 
    3. 이 실행 체인은 1개 이상의 인터셉터가 등록되어 있다면 순차적으로 인터셉터들을 거쳐 컨트롤러가 실행되도록 하고, 인터셉터가 없다면 바로 컨트롤러를 실행한다.

## 실제 사용 예시

1. 사용목적 : httpServletRequest의 인증 헤더에 담긴 로그인 정보를 추출해, 인증(로그인)하고, 컨트롤러에 memberId라는 가공된 객체(Long타입)을 전달하기 위해 사용했다
    
    
2. 실제 코드
    
    ```java
    public class AuthenticationInterceptor implements HandlerInterceptor {
    
        private final BasicAuthorizationExtractor basicAuthorizationExtractor;
        private final CredentialDao credentialDao;
    
        public AuthenticationInterceptor(BasicAuthorizationExtractor basicAuthorizationExtractor,
                CredentialDao credentialDao) {
            this.basicAuthorizationExtractor = basicAuthorizationExtractor;
            this.credentialDao = credentialDao;
        }
    
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            CredentialEntity credential = basicAuthorizationExtractor.extract(request);
    
            CredentialEntity savedCredential = credentialDao.findByEmail(credential.getEmail())
                    .orElseThrow(()-> new IllegalArgumentException("존재하지 않는 회원입니다"));
    
            if(!credential.getPassword().equals(savedCredential.getPassword())) {
                throw new IllegalArgumentException("비밀번호가 일치하지 않습니다");
            }
    
            request.setAttribute("memberId", savedCredential.getMemberId());
    
            return true;
        }
    }
    ```
    

---

# ArgumentResolver(아규먼트 리졸버)란?

## 사용목적

스프링의 디스패처 서블릿은 컨트롤러로 요청을 전달한다. 

이 때, 요청에 들어온 값으로부터 컨트롤러에서 **필요로 하는 객체를 만들고 값을 바인딩**(매개변수에 인수를 연결)하기 위해 사용되는 것이 ArgumentResolver이다. 

스프링이 제공하는 다음과 같은 어노테이션들은 모두 ArgumentResolver로 동작한다.

- @RequestParam: 요청의 파라미터 값 바인딩
- @ModelAttribute: 요청의 파라미터 및 폼 데이터 바인딩
- @CookieValue: 요청의 쿠키값 바인딩
- @RequestHeader: 요청의 헤더값 바인딩
- @RequestBody: 요청의 바디값 바인딩

즉,  ArgumentResolver가 동작하면 컨트롤러로 전달할 객체가 만들어지고, 컨트롤러에게 전달된다.

## 사용 방법

1. ArgumentResolver는 `HandlerMethodArgumentResolver`를 구현한다
    - **HandlerMethodArgumentResolver란?**
        
        : *Strategy interface for resolving method parameters into argument values in the context of a given.*
        
        (전략 인터페이스 :메서드 매개변수를 인자 값으로 변환하는 전략을 정의한다)
        
    - HandlerMethodArgumentResolver의 구현
        1. **`supportsMethod`**: 이 메서드는 인자로 주어진 메서드를 **해당 전략이 해결할 수 있는지 여부를 판단**합니다. 이 메서드는 해당 메서드의 매개변수 정보를 사용할 수 있습니다.
            
            boolean 값을 반환하며, true를 반환하면 이 ArgumentResolver가 이 메서드를 처리할 수 있음을 의미한다
            
        2. **`resolveMethodArguments` 메서드**: 
            
            Controller 메서드의 인자를 실제로 바인딩하는 데 사용되는데, **`supportsMethod`**에서 true를 반환한 메서드에 대해서만 호출된다. 
            
            이 메서드는 컨트롤러 메서드의 인자를 바인딩하는 데 필요한 모든 정보를 가지고 있으며, 바인딩된 인자를 반환한다.
            
            이 메서드는 컨트롤러 메서드의 각 파라미터를 바인딩하기 위해 호출되며, 다음과 같은 매개변수를 가집니다.
            
            - **parameter**: 바인딩할 파라미터에 대한 정보를 제공합니다. 이 정보에는 파라미터 이름, 타입, 어노테이션 및 기타 메타데이터가 포함됩니다.
            - **mavContainer**: 현재 요청에 대한 **ModelAndViewContainer. (**컨트롤러 메서드가 반환하는 모델과 뷰를 관리하는 데 사용됩니다.)
            - **webRequest**: 현재 요청에 대한 **NativeWebRequest**. **NativeWebRequest**는 HTTP 요청에 대한 정보와 처리를 제공하는 데 사용됩니다.
            - **binderFactory**: 현재 요청에 대한 **WebDataBinderFactory**입니다. **WebDataBinderFactory**는 요청에 대한 데이터 바인더를 생성하는 데 사용됩니다.
            
            따라서 **`resolveArgument`** 메서드는 각 파라미터를 바인딩하기 위해 필요한 모든 정보를 가지고 있으며, 이 정보를 사용하여 바인딩된 파라미터를 반환합니다. 이를 통해 컨트롤러 메서드에서 파라미터를 처리하는 데 필요한 로직을 중복하지 않고도 다양한 유형의 파라미터를 처리할 수 있습니다.
            

## 실제 사용 예시

1. 사용 목적 : httpServletRequest에 저장된 member_id를 (Member객체로 만들고 바인딩)/  (Long 타입 그대로 바인딩)하기 위해서 사용하였다.
2. @Authorization 어노테이션을 가진 컨트롤러의 매개변수에 바인딩하여 값을 전달한다.
3. 실제 코드
    
    ```java
    public class AuthenticationArgumentResolver implements HandlerMethodArgumentResolver {
    
        @Override
        public boolean supportsParameter(MethodParameter parameter) {
            return parameter.hasParameterAnnotation(Authorization.class);
        }
    
        @Override
        public Object resolveArgument(MethodParameter parameter,
                ModelAndViewContainer mavContainer,
                NativeWebRequest webRequest,
                WebDataBinderFactory binderFactory) {
            HttpServletRequest request = (HttpServletRequest) webRequest.getNativeRequest();
            return (Long) request.getAttribute("memberId");
        }
    }
    ```
    

---

# View Resolver란?

## 사용 목적

: 스프링 백엔드에서 데이터를 처리하거나 가지고 왔다면, 이 데이터를 View의 영역으로 전달을 해야 한다. 이때 **어떤 View를 사용할지 자유롭게 설정을 할 수 있는데 이 설정 역할을 하는 것이 View Resolver**이다.

즉, 

- 실행할 뷰를 찾는 일을 한다.
- 페이지 컨트롤러가 리턴한 뷰 이름에 해당하는 뷰 콤포넌트를 찾는 역할.

예를 들어, 데이터를 API 형태로 제공하길 원한다면 View Resolver를 통해서 JSON 형태(Json View 라고 함)로 전달하거나 Java 프론트엔드 언어인 JSP 페이지로도 보낼수도 있다.

(참고로 jsonView는 @RequestBody가 반환하는 json 형식과 다르다)

## 사용 방법

ViewResolver는 디스패처 서블릿이 관리하는 객체 중 하나로, 

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7c028b78-ba83-4abf-a5e6-f9dd7d95d475/Untitled.png)

1. 디스패처 서블릿이 핸들러 어댑터를 통해 논리 뷰 이름을 획득한다.
    
    예) “hello” 획득
    
2. 1. ViewResolver는 설정된 ViewResolver의 우선순위에 따라 찾을 View 객체를 결정한다. ViewResolver의 우선순위는 설정 파일에 등록된 순서에 따라 결정된다.
3. ViewResolver는 찾은 **View 객체를 반환**합니다. 
    
    스프링에서 기본으로 사용하는 ViewResolver는 **`InternalResourceViewResolver`**입니다. 이 ViewResolver는 주어진 뷰 이름을 JSP 파일, Velocity 템플릿, Thymeleaf 템플릿 등의 웹 애플리케이션 리소스와 연결하여 최종적으로 렌더링할 뷰를 결정한다.
    
    이때, View 객체는 JSP, Thymeleaf, Freemarker, Velocity 등과 같은 템플릿 엔진에서 생성될 수 있다.
    
4. 컨트롤러에서 반환한 모델 데이터는 ViewResolver가 찾은 View 객체에서 사용할 수 있도록 전달된다.
5. View 객체는 모델 데이터를 이용하여 클라이언트의 브라우저에 보여질 HTML 코드를 생성한다.

우리의 코드를 예시로 들자면, 

스프링 부트의 기본 viewresolver는 Thymeleaf 템플릿 엔진을 사용한다. 

따라서 index.html은 Thymeleaf 템플릿으로 작성되어 있어야 한다. 

+ index.html 파일은 스프링 부트 프로젝트의 src/main/resources/templates 디렉토리에 있어야 함.

만약 다른 viewresolver를 사용하고 싶다면, 

스프링 부트의 설정 파일(application.properties 또는 application.yml)에서,

해당 viewresolver를 설정할 수 있다. 

```
예를 들어, JSP를 사용하고 싶다면 다음과 같이 설정할 수 있습니다:

spring.mvc.view.prefix: /WEB-INF/jsp/
spring.mvc.view.suffix: .jsp
```

이렇게 설정하면, 

view 이름이 "index"인 요청에 대해 "/WEB-INF/jsp/index.jsp" 파일을 렌더링하는 InternalResourceViewResolver가 사용된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/547fcc5c-c587-4dea-81d4-c5e0b87011fb/Untitled.png)

원하는 뷰 타입의 Dependency를 추가하게 되면 다양한 방식으로 보여줄 수 있는데 아래와 같은 View를 제공한다.

### View Type

- **Thymeleaf (대표적)**
    - Dependency 추가 시 별도 설정 없이 HTML 파일로 View 제작
    - HTML 이지만 JSP 처럼 동작 가능
    - Spring에서 가장 밀어주고 있는 View
- **JsonView**
    - API 형태로 제공할 때 가장 많이 사용하는 View
- **JSP**
    - 국내에서는 가장 많이 사용하는 표준 View
    - **Spring boot에서는 별도의 설정을 해야 사용이 가능할 정도로 권장하지 않고 있음**
    

## 실제 사용 예시

---

# 동작 구조(순서)

1. 인터셉터
    
    postHandle
    
    1) 사용자는 서버에 자신이 원하는 작업을 요청하기 위해 url을 통해 Request 객체를 보낸다.
    
    2) DispatcherServlet은 해당 Request 객체를 받아서 분석한뒤 '핸들러 매핑(HandlerMapping)' 에게
    
    사용자의 요청을 처리할 핸들러를 찾도록 요청 한다.
    
    3) 그결과로 핸들러 실행체인(HandlerExectuonChanin)이 동작하게 되는데, 이 핸들러 실행체인은 하나이상의 핸들러 인터셉터를 거쳐서 컨트롤러가 실행될수 있도록 구성되어 있다.
    
    (핸들러 인터셉터를 등록하지 않았다면, 곧바로 컨트롤러가 실행된다. 반대로 하나이상의 인터셉터가 지정되어 있다면 지정된 순서에 따라서 인터셉터를 거쳐서 컨트롤러를 실행한다)
    
    ![https://blog.kakaocdn.net/dn/yWXru/btq71aaDkS8/6IcMoizEyejCBT2kFN04o1/img.png](https://blog.kakaocdn.net/dn/yWXru/btq71aaDkS8/6IcMoizEyejCBT2kFN04o1/img.png)
    
2. 아규먼트 리졸버
    
    @Controller로 빈 등록이 되어 매핑된 값들을 RequestMapping으로 찾아 각각의 메서드들을 호출한다
    
    이때 메서드만을 호출하는 것이 아니라 클라이언트가 전달하는 데이터
    
    그리고 메서드를 반환할 때의 반환 값을 클라이언트와 서버는 주고받는다
    
    = 클라이언트가 요청을 할 때 그리고 서버가 응답을 할때, 단순히 요청과 응답뿐이 아닌 각종 **데이터**를 함께 넘겨주고 반환받는다
    
    이때 이 데이터를 처리하는 인터페이스가 바로 Argument Resolver와 ReturnValue Handler이다
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5fb1ca2e-1cf6-4f76-b6f9-7266b5076676/Untitled.png)
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/97e25942-e03a-4b9d-b67f-912926d9a325/Untitled.png)
    
    1. Handler를 호출하여 파라미터 값을 확인하여 정보를 전달받는다(Handler = Controller의 메서드)
        
        +어떻게?
        
    2. Argument Resolver를 호출하여 해당 파라미터의 객체 생성을 요구한다
        
        Argument Resovler가 호출되면 supportsParamter를 호출하여 **해당 파라미터를 지원하는지 체크(어노테이션으로 확인?)**한다,
        
        지원하게 되면 resolveArgument를 호출하여 실제 객체를 생성하고 다시 Handler를 호출한다
        
    
    3. Parameter의 객체를 생성하여 Handler를 호출함과 동시에 객체 또한 같이 전달한다
    
    (이미 알고 있는 내용 재 설명 : 위로가기 귀찮으니까)
    
    컨트롤러의 메서드의 인자로 사용자가 임의의 값을 전달하는 방법을 제공하고자 할 때 사용한다
    
    즉 HttpServletRequest, Model, @RequestParam, @ModelAttribute 등으로 작성된 전달 데이터를 기반으로
    
    매우 다양한 파라미터 값을 사용할 때 필요한 인터페이스이다
    
    RequestParam을 다이어그램으로 까보면 실제로 위에 Argument Resolver가 있는 것을 확인할 수 있다
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c7cafbf4-7889-4598-9c22-7626c33229ca/Untitled.png)
    
    1. 뷰리졸버
        
        작동과정 비교( **@Controller vs @ResponseBody)**
        
        ![https://blog.kakaocdn.net/dn/boeeLa/btrhBL7QBDW/SRhOppFkOPIWrkqASNKxhK/img.png](https://blog.kakaocdn.net/dn/boeeLa/btrhBL7QBDW/SRhOppFkOPIWrkqASNKxhK/img.png)
        
        1. Client는 URI 형식으로 웹 서비스에 요청을 보냅니다.
        2. Mapping되는 Handler와 그 Type을 찾는 DispatcherServlet이 요청을 인터셉트합니다.
        3. Controller가 요청을 처리한 후에 응답을 DispatcherServlet으로 반환하고, DispatcherServlet은 View를 사용자에게 반환합니다.
            
            **@Controller가 View를 반환하기 위해서는 ViewResolver가 사용되며, ViewResolver 설정에 맞게 View를 찾아 렌더링합니다.**
            
        
        +)추가 
        
        반면, 
        
        Spring MVC의 컨트롤러에서는 데이터를 반환하기 위해 **@ResponseBody** 어노테이션을 활용해주어야 합니다. 이를 통해 Controller는 Json 형태(대표적) 혹은 다른 형태로 데이터를 반환할 수 있습니다.
        
        ![https://blog.kakaocdn.net/dn/difDdS/btrhDF7MeAK/6LdpcNQdnkhheVUy6VxgPk/img.png](https://blog.kakaocdn.net/dn/difDdS/btrhDF7MeAK/6LdpcNQdnkhheVUy6VxgPk/img.png)
        
        1. Client는 URI 형식으로 웹 서비스에 요청을 보냅니다.
        2. Mapping되는 Handler와 그 Type을 찾는 DispatcherServlet이 요청을 인터셉트합니다.
        3. @ResponseBody를 사용하여 Client에게 Json 형태로 데이터를 반환합니다.
            
            **@RestController가 Data를 반환하기 위해서는 viewResolver 대신에 HttpMessageConverter가 동작합니다.** HttpMessageConverter에는 여러 Converter가 등록되어 있고, 반환해야 하는 데이터에 따라 사용되는 Converter가 달라집니다.
            
            단순 문자열인 경우에는 StringHttpMessageConverter가 사용되고, 객체인 경우에는 MappingJackson2HttpMessageConverter가 사용되며, 데이터 종류에 따라 서로 다른 MessageConverter가 작동하게 됩니다.
            
            Spring은 클라이언트의 HTTP Accept 헤더와 서버의 컨트롤러 반환 타입 정보 둘을 조합해 적합한 HttpMessageConverter를 선택하여 이를 처리합니다.
