
# MVC Config

![공식문서](https://blog.kakaocdn.net/dn/MifK5/btsd4exD4y0/pDjcPThN8t21rr2rQAm6S1/img.png)

공식 문서에서 소개하고 있는 MVC Config는 **서블릿을 포함한 환경에 대한 Configuration**을 의미한다. 

'이전에 @Configuration 어노테이션을 활용하면 추가적인 환경설정을 할 수 있는 거 아니었나? 그런데 왜 또 나오지?'라고 생각할 수 있다. 
맞다! 

@Configuration 어노테이션을 사용하는 것만으로도 Configuration이 가능하다. 
그런데 이번 포스팅에서는 @EnableWebMvc 어노테이션을 사용해서 Configuration을 하는 과정에 대해서 알아볼 것이다.


**글을 다 적고 보니 내용이 조금 어수선하다. 쭉 따라서 읽어도 정리가 안될 수 있으니 뒤 쪽의 요약을 보고 글을 다시 보는 것을 추천한다!**

먼저 WebMvcConfigurer에 대해 가볍게 살펴보자.

## WebMvcConfigurer란?

![](https://blog.kakaocdn.net/dn/nwIy3/btsd4eYO2VV/8wmOAJMbzgmaLQJ3CkbQoK/img.png)

WebMvcConfigurer는 `기본적으로 제공해주는 MVC Configuration을 커스터마이징`할 수 있도록 인터페이스이다. 
여기에서는 formatter도 등록할 수 있고, interceptors나 resolver, 혹은 handler 등등도 직접 커스터마이징해서 등록할 수 있다.

**WebMvcConfigurer를 구현한 클래스에 @Configuration 어노테이션을 붙이면 기본적으로 스프링에서 제공해주는 Configuration에 더불어서 우리의 커스터마이징한 Configuration을 추가할 수 있는 것이다.**

![](https://blog.kakaocdn.net/dn/bAEPSw/btsd3fcK3WZ/jI3zpcGU4Q3Bk4h29y4RN1/img.png)

위의 코드는 그 예시이다. **기본적으로 스프링에서 제공해주는 Configuration에 커스터마이징한 Interceptor를 추가**해주었다.

WebMvcConfiguration과 관련된 메서드는 다음 포스팅에서 조금 더 자세히 살펴보도록 하겠다.

## EnableWebMvc

![](https://blog.kakaocdn.net/dn/d1LcZn/btsd9MAJVHV/JLzQEZVX6WIeBGSyBDW5Z0/img.png)

@EnableWebMvc 어노테이션도 마찬가지로 Configuration을 설정할 때 사용하는 어노테이션이다. 
그런데 이는 @Configuration 어노테이션과 함께 사용하는 경우에 `**WebMvcConfigurationSupport**에 있는 Spring MVC Configuration`을 가져올 수 있다고 한다. 


## DelegatingWebMvcConfiguration

![](https://blog.kakaocdn.net/dn/ckajgA/btsd5ZNOULA/0Bnzl8p1yFzIas4mgHvNvK/img.png)

#### DelegatingWebMvcConfiguration은 WebMvcConfigurationSupport를 상속받고 있다.

WebMvcConfigurationSupport는 **WebMvcConfigurer에 대한 모든 빈 타입을 커스터마이징**할 수 있도록 돕는다. 
풀이하자면 WebMvcConfigurationSupport 또한 MVC Configuration를 커스터마이징할 수 있도록 돕는 클래스라고 생각하면 된다. 
DelegatingWebMvcConfiguration 클래스는 이러한 WebMvcConfigurationSupport를 상속받았기에 MVC Configuration을 따로 설정할 수 있다.

먼저 DelegatingWebMvcConfiguration의 setConfigureres를 살펴보자.

![](https://blog.kakaocdn.net/dn/wXuZS/btsd4dewk26/VUipxGyZ1VsfaSFFXnAVKk/img.png)

setConfigurers 메서드는 **WebMvcConfigurer의 리스트를 전달받아서 현재 WebMvcConfigurerComposite에 추가**해주는 것을 볼 수 있다. 
WebMvcConfigurerComposite는 WebMvcConfigurer를 위임받아서 사용하는 객체라고 생각하면 된다.
다시 말해 이 과정을 통해 **기존의 Configuration과 관련된 설정을 WebMvcConfigurerComposite에서 위임**받아서 처리한다고 생각하면 된다. 

오버라이드된 메서드들도 그저 **주입받은 Configuration 관련 메서드를 단순히 호출**해줄 뿐이다.

```java
@Configuration(proxyBeanMethods = false)
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {

   private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();


   @Autowired(required = false)
   public void setConfigurers(List<WebMvcConfigurer> configurers) {
      if (!CollectionUtils.isEmpty(configurers)) {
         this.configurers.addWebMvcConfigurers(configurers);
      }
   }


   @Override
   protected void configurePathMatch(PathMatchConfigurer configurer) {
      this.configurers.configurePathMatch(configurer);
   }
	...

   @Override
   @Nullable
   protected MessageCodesResolver getMessageCodesResolver() {
      return this.configurers.getMessageCodesResolver();
   }

}
```

@EnableWebMvc 어노테이션을 @Configuration 어노테이션과 함께 사용하는 경우에는 이러한 DelegatingWebMvcConfiguration에 있는 Spring Configuration 정보를 가져온다고 했다.

그리고 **DelegatingWebMvcConfiguration은 WebMvcConfigurationSupport를 상속**받고 있는 것을 알 수 있다. 

## @Configuration과 @EnableWebMvc

아직까지는 DelegatingWebMvcConfiguration 클래스가 뭐하는건지 잘 이해가 안될 것이다. 
솔직히 DelegatingWebMvcConfiguration 그 자체는 별로 중요하지 않다. 
중요한 것은 DelegatingWebMvcConfiguration가 WebMvcConfigurationSupport를 상속받고 있다는 점이다. 
그리고 **@EnableWebMvc는 WebMvcConfigurationSupport에서 구성한 Configuration을 가져온다는 점 때문에 @Configuration만 사용해서 환경설정을 할 때와는 큰 차이**를 보인다.

#### **@EnableWebMvc를 사용하는 경우에는 기존의 Spring Configuration이 일부 무시**된다.

왜냐하면 Spring auto Configuration은 WebMvcAutoConfiguration이 담당하는데 이 클래스를 살펴보면 다음과 같은 어노테이션이 붙여져있기 때문이다.

#### @ConditionalMissingBean(WebMvcConfigurationSupport.class)

![](https://blog.kakaocdn.net/dn/kZ84h/btsd0vAetcF/ym7JVCxMWLv9qRK6hM4GUK/img.png)

자연어로 풀어서 설명하자면 WebMvcConfigurationSupport 클래스가 빈으로 등록이 안되어있는 경우에 해당 클래스가 AutoConfiguration을 활성화한다는 것이다. 
@EnableWebMvc 어노테이션을 사용하는 경우에는 WebMvcConfigurationSupport 클래스를 구현한 DelegatingWebMvcConfiguration를 import해서 Configuration으로 등록하기 때문에 Spring에서 제공해주는 기본 Configuration을 사용하지 못하는 것이다.

## @EnableWebMvc 테스트

솔직히 긴가민가해서 한번 실제로 직접 돌려봤다.

![](https://blog.kakaocdn.net/dn/bixn6n/btseggg1Ne6/t0G5sLXYoFkRBgctkRKKgk/img.png)

먼저 @EnableWebMvc 어노테이션을 주석으로 처리한 다음에 (1) **컨트롤러에서 ViewResolver를 출력**해봤다. 
그리고 (2) **String을 리턴해서 실제 ViewResolver가 어떻게 동작**하는지도 확인해봤다.

![](https://blog.kakaocdn.net/dn/bAiYBM/btseggORukq/klOKcqvO1TBjPJBZG53ix1/img.png)


#### \[콘솔 출력\]

> \[org.springframework.web.servlet.view.ContentNegotiatingViewResolver@35ef439e,  
> org.springframework.web.servlet.view.BeanNameViewResolver@7ae0cc89,   
> org.thymeleaf.spring5.view.ThymeleafViewResolver@4997552e,   
> org.springframework.web.servlet.view.ViewResolverComposite@de7e193,   
> org.springframework.web.servlet.view.InternalResourceViewResolver@13e5d243\]

총 5개의 ViewResolver가 등록되어있는 것을 확인할 수 있다. 실제로 브라우저에서 해당 url로 접속했을 때에도 아래와 같이 화면이 랜더링된다.

![](https://blog.kakaocdn.net/dn/c3HkL6/btsd9MngMRO/o41r7MuA9mfKZuQrdOIlaK/img.png)

이번 미션을 하면서 자주 봤던 화면일 것이다.

이번에는 @EnableWebMvc 어노테이션을 붙인 뒤에 동일한 코드를 실행해보았다.

![](https://blog.kakaocdn.net/dn/bZWEQm/btseh8iNqtY/WNz2uvHwmzMgDWmLrXm1lK/img.png)

#### \[콘솔 출력\]

> \[org.springframework.web.servlet.view.BeanNameViewResolver@322ba549,   
> org.thymeleaf.spring5.view.ThymeleafViewResolver@502a4156,   
> org.springframework.web.servlet.view.ViewResolverComposite@1163a27\]

ViewResolver가 3개로 줄어든 것을 확인할 수 있다. 위에서 언급한대로 기본적으로 Spring에서 제공해주는 Configuration 중 일부가 사라진 것을 확인할 수 있다. 
또한 ViewResolver가 몇 개 없어졌기에 실제 랜더링되는 화면도 아래와 같이 변했다.

![](https://blog.kakaocdn.net/dn/UeOls/btsd1jsUWvo/3RtKVDwFWRCv3AJ1vwrWMk/img.png)

화면이 랜더링되는 것도 달라진 것을 알 수 있다. 
ViewResolver가 화면을 랜더링하는 역할을 하는데, 깔끔하게 랜더링해주는데 도움을 주는 어떤 ViewResolver가 사라졌기 때문에 이와 같이 안예쁜 화면이 뜨는 것이다. 

## 일단 알겠는데, 지금까지 나온 내용을 짧게 요약해줘

@EnableWebMvc 어노테이션을 사용해서 Configuration을 커스터마이즈하는 경우에는 기존에 Spring에서 제공하는 일부 Configuration을 무시한다. 

WebMvcAutoConfiguration은 Spring에서 제공하는 Configuration을 자동으로 설정하는데 사용되는 클래스이다. 
이 클래스에는 @ConditionalMissingBean(WebMvcConfigurationSupport.class)이라는 어노테이션이 붙어있다. 
따라서 WebMvcConfigurationSupport 클래스가 있는 경우에는 Spring에서 제공하는 Configuration을 자동으로 설정할 수 없게 된다. 
그런데 @EnableWebMvc 어노테이션을 사용하는 경우에는 WebMvcConfigurationSupport 클래스를 구현한 DelegatingWebMvcConfiguration 클래스를 import하고 있다.
따라서 @EnableWebMvc 어노테이션을 사용하면 WebMvcAutoConfiguration이 활성화되지 않고 기본적으로 Spring에서 제공하는 Configuration을 설정하지 못하게 된다.

일단 @EnableWebMvc에 대해서는 이 정도까지 알아보도록 하자. 다음에는 이어서 WebMvcConfigurer에서 제공하는 메서드에 대해 간략하게 알아보도록 하겠다.
