## 인증(Authentication)과 인가(Authorization)

인증은 클라이언트가 사용자라고 주장할 때, 주장한 사용자가 맞는 지를 검사하는 과정이다.

인가는 이미 인증된 사용자가 어떤 작업을 수행하려고 할 때, 사용자에게 접근 자격이 있는 지를 검사하는 과정이다.

즉, 인증은 클라이언트가 누구인지를 검증하고, 인가는 클라이언트가 무엇을 할 수 있는 지를 검증한다.

![](Authentication_vs_Authorization.png)



## HTTP의 무상태성(Stateless)

클라이언트와 서버는 요청과 응답을 주고받기 위해 `HTTP`를 사용한다. 위에서 나온 인증과 인가의 절차또한 `HTTP`를 사용해서 이루어진다.



`HTTP`는  무상태성(Statless)이라는 성질을 가진다.

무상태성은 서버가 클라이언트의 정보를 저장하지 않는다는 의미이다. 

때문에 클라이언트가 A요청을 한 뒤, B요청을 이어서 보냈을 때 서버는 A요청에 대한 정보는 기억하지 못한다. 이해하기 쉬운 예시가 있어 가져와봤다.

```
손님: 노트북 모델 Z의 가격은 얼마인가요?
직원A: 100만원입니다.

손님: 이 모델 2개 구매하겠습니다.
직원B: 어떤 모델 말씀이실까요?

손님: 카드로 결제하겠습니다.
직원C: 어떤 모델을 몇 개 구매하시는 건가요?
```



![](http-stateless.png)



뭔가 이상하지 않은가? 

어떤 게시판 서비스가 있을 때, 인증된 사용자만 글을 작성할 수 있다고 해보자.

글을 작성하고 싶은 클라이언트는 우선 인증을 받기 위해 서버에 요청을 보낼 것이다. HTTP를 사용해서 서버는 클라이언트를 인증했다.

인증을 했으니 이제 글을 작성할 차례이다. 클라이언트가 서버에게 글을 작성하기 위한 요청을 보낸다.

하지만 HTTP의 무상태성대로라면, 서버는 이미 클라이언트에 대한 정보를 잊었다. 요청한 클라이언트가 인증된 사용자인지, 아닌지를 알 수가 없다.

그럼 서버는 어떻게 해야할까? 클라이언트가 요청을 보낼 때 마다 계속 인증 과정을 거쳐야할까?

하지만 우리가 사용하는 서비스들은 그렇지 않다. 이는 클라이언트와 서버가 클라이언트에 대한 인증 정보를 어딘가에 저장해서 사용하고 있다는 의미이다.

클라이언트와 서버가 어떤 식으로 인증을 거치고, 인증 정보를 유지하는지 알아보도록 하자.

<br/>


## Basic 인증

이름에서부터 알 수 있듯이 가장 기본적인 인증이다.

사용자ID와 비밀번호를 base64로 인코딩한 뒤, 헤더에 담아 서버에게 보내는 인증 방식이다.

<br/>




### Basic 인증 과정

Basic 인증 과정은 다음과 같다.

클라이언트가 인증 요청을 보내면 서버는 DB에서 해당하는 사용자를 찾는다. 올바른 사용자를 찾았다면 클라이언트의 인증 정보를 담은 응답을 보낸다. 클라이언트는 받은 인증 정보를(사용자ID&비밀번호) 브라우저에 저장한다.

이후 클라이언트가 요청을 보낼 때 인증 과정은 아래 사진과 같다.




![](basic.png)



* 클라이언트가 요청을 보낸다.
* 서버가 `401(Unauthorized)` 응답과 함께 `WWW-Authenticate` 헤더에 클라이언트가 어떻게 인증을 해야할 지를 담아 보낸다.
* 클라이언트는 `WWW-Authenticate`에 담긴 인증 방식대로 `Authorization`헤더에 인증 정보를 담아 다시 요청을 보낸다.
* 서버는 클라이언트의 인증 정보를 보고 `200(OK)` or `403(Forbidden)` 응답을 한다.


<br/>

#### `401응답과 403 응답`

401은 인증되지 않은 유저일 때, 403은 해당 유저가 리소스에 접근할 권한이 없을 때 반환되는 응답이다.

401은 인증, 403은 인가와 관련된 응답이다.

<br/>


#### `WWW-Authenticate 헤더`

인증을 요구하는 401상태 응답과 함께 사용되는 헤더이다.

특정 리소스에 접근하기 위해 클라이언트가 어떤 인증 방식을 사용해야하는 지를 알려준다.

가장 기본적인 형태는 `WWW-Authenticate: <auth-scheme>`이며, auth-schema를 통해 어떤 인증 방식을 사용해야하는 지를 표시한다. auth-schema뒤에 부가적인 정보를 붙일 수 있다.

```
WWW-Authenticate: <auth-scheme> realm=<realm>
WWW-Authenticate: <auth-scheme> token68

// 예시
WWW-Authenticate: Basic realm="Corporate Financials"
```

자세한 정보는 [WWW-Authenticate](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/WWW-Authenticate)에서 확인할 수 있다.

<br/>


#### `Authorization 헤더`

인증 토큰을 보낼 때 사용하는 헤더이다. 

기본 형태는 `Authorization: <auth-scheme> <authorization-parameters>`이다.

- 예시 : `Authorization: Basic YWxhZGRpbjpvcGVuc2VzYW1l`

자세한 정보는 [Authorization](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization)에서 확인할 수 있다.

<br/>


### Basic 인증의 장점

#### 📌 간단한 구현 

`Basic`의 장점은 간단하다는 것이다. 복잡한 암호 방식을 사용하지도 않고 그저 유저ID와 비밀번호를 base64로 인코딩해 서버에게 넘겨주기만 하면 된다.

<br/>


### Basic 인증의 단점

#### 📌 추가적인 보안 필요

하지만 암호화를 하지 않기 때문에 HTTP Request를 보면 누구나 유저의 ID & 패스워드를 알 수 있다. 그렇기에 HTTP보다 보안이 강화된 HTTPS와 함께 쓰이는 것이 일반적이다.

<br/>


#### 📌 서버 성능 저하

또한,  `Basic`인증의 경우 서버가 요청을 받을 때마다 클라이언트가 보낸 유저ID&비밀번호와 일치하는 유저를 DB에서 매번 찾아야한다. 그렇기에 DB에 저장된 유저가 많을 수록, 트래픽이 많을 수록 서버 성능이 저하된다.

<br/>


#### 📌 정교한 권한 설정이 어려움

`Basic`인증 방법으로 정교하게 사용자 권한을 제어하려면 추가적인 작업이 필요하다고 한다. 솔직히 안해봐서 잘 모르겠다.

<br/>


## 세션 인증

세션 인증은 사용자의 인증 정보를 서버의 세션 저장소에 저장하는 방식이다. 오늘 소개되는 세 방법 중 유일하게 서버에 인증 정보를 저장한다.

![](session-based-authentication.png)

세션 인증 과정은 다음과 같다.

* 클라이언트가 로그인 요청을 보낸다.
* 서버는 인증 과정을 거친 뒤, 클라이언트의 인증 정보를 세션 저장소에 저장한다.
* 서버는 인증 정보가 저장된 세션의 아이디를 `Set-Cookie`헤더에 담아 응답한다
* 클라이언트는 세션 아이디를 쿠키에 저장한다. 이후 요청을 보낼때마다 `Cookie`헤더에 세션 아이디를 담아 보낸다.
* 서버는 받은 세션 아이디를 통해 클라이언트를 인증한다.

클라이언트에서 쿠키로 세션 아이디를 저장하지만, 실질적인 인증 정보는 서버의 세션 저장소에 저장된다.


<br/>

### 세션 인증의 장점

#### 📌 적은 트래픽 사용

인증에 필요한 정보가 세션 아이디뿐이라서 적은 트래픽만으로도 통신이 가능하다.

동일한 값(abc123)을 쿠키와 JWT를 사용해 저장할 경우, 

[why-jwts-suck-as-session-tokens](https://developer.okta.com/blog/2017/08/17/why-jwts-suck-as-session-tokens) 포스팅을 보면 쿠키에 저장할 때는 6byte만 필요하지만(세션 인증이 이에 해당), basic한 header와 일반적으로 사용되는 긴 secret key를 사용한 JWT의 경우는 304byte가 필요하다고 한다.  

<br/>


#### 📌 보안에 강함

클라이언트의 쿠키가 탈취되어 세션 아이디가 드러나더라도, 유저의 실질적인 인증 정보는 서버의 세션 저장소에 저장되기 때문에 보안에 강력하다. 

<br/>


### 세션 인증의 단점

#### 📌 서버의 부담

유저의 인증 정보를 서버의 세션 저장소에 저장하기 때문에 유저가 많아질수록 서버가 저장해야하는 정보또한 늘어난다. 때문에 여러 개의 세션 저장소를 두는 등 서버의 부담을 줄이려는 노력이 필요하다.

<br/>


#### 📌 서버 확장에 약함

다른 인증 방법에 비해 서버를 확장하기가 어렵다. 서버의 부담을 줄이기 위해 다중 서버를 사용한다고 해보자. 각 서버는 세션 저장소를 가지고 있다. 

세션 인증의 경우 클라이언트가 세션 아이디를 가지고 인증을 수행한다. 클라이언트가 첫 번째 요청때는 서버A와 통신하고, 두 번째 요청때는 서버B와 통신한다면 인증을 수행할 수 있을까? 각 서버가 가진 세션 저장소가 다르기 때문에 클라이언트가 여러 개의 서버와 통신할 경우 세션인증을 수행할 수 없다.

그렇다면, 모든 서버가 동일한 세션 저장소를 사용한다면? 위에서 말했듯이 유저가 많아질수록 세션 저장소에 부담이 커질 것이다.

세션 불일치 문제를 보완하기 위해 `Sticky Session`, `Session Clustering`, Redis등을 사용해 세션 저장소를 아예 외부로 분리하는 방법이 있지만, 다른 인증방법에 비해 서버 확장면에서 자유롭지 못한 것은 사실이다.

세션 불일치 문제 해결 방법에 대해선 다음 자료를 참고하면 좋을 것 같다.

* [https://hudi.blog/session-consistency-issue/](https://hudi.blog/session-consistency-issue/)
* [https://traefik.io/glossary/what-are-sticky-sessions/](https://traefik.io/glossary/what-are-sticky-sessions/)
* [https://aws.amazon.com/ko/elasticache/what-is-redis/](https://aws.amazon.com/ko/elasticache/what-is-redis/)



<br/>


## 토큰 인증(JWT)

이번엔 토큰 인증을 알아보자. JWT를 사용한다고 가정하겠다.

<br/>


### JWT란?

토큰은 클라이언트의 인증 정보를 담은 암호화된 데이터이다. 그 중 `JWT`는 `Json Web Token`의 약자로 웹 서비스를 인증하는데 많이 쓰이는 토큰이다. 이름 그대로, 웹상에서 `Json`형태로 주고 받을 수 있는 토큰이다.
<br/>


다음은 JWT의 구조를 나타내는 사진이다.

![](jwt.png)

`JWT`는 `HEADER`, `PAYLOAD`, `VERIFY SIGNATURE`로 구성되어 있다. 

`JWT`는 암호화된 각 정보가 `.`으로 이어져있는 형태이다. 위 사진의 `Encoded`부분을 보면 알 수 있다.

> `Encoded HEADER` + "." + `Encoded PAYLOAD` + "." + `Encoed VERIFY SIGNATURE`



* `HEADER`
  * 일반적으로 토큰의 타입과 토큰이 어떤 암호 방식을 사용했는지를 담고있다.
* `PAYLOAD`
  * 유저 네임등 인증에 필요한 정보들을 담고있다.
* `VERIFY SIGNATURE` (서명)
  * 서버의 `secret key`를 사용해 서명한 내용을 담고있다. 
  * Encoded 된 `HEADER`, `PAYLOAD`와 `secret key`를 암호화한 정보를 담고있다.
  * 암호화 알고리즘은 `HEADER`에 적힌 방법대로 수행한다.

자세한 정보는 [JWT.io](https://jwt.io/introduction)에서 확인할 수 있다. 

<br/>




![](token-based-authentication.png)



토큰 인증 과정은 다음과 같다.

* 클라이언트가 로그인 요청을 보낸다.
* 서버는 인증 과정을 거친 뒤, 토큰을 생성한다.
* 서버는 토큰을 응답에 담아 보낸다.
* 클라이언트는 토큰을 브라우저에 저장한다.(localStorage나 쿠키 등)
* 클라이언트는 이후 요청을 보낼 때 마다 요청 헤더에 토큰을 담아 보낸다.
* 서버는 받은 토큰을 해석해 유저를 검증한다.



<br/>


### 토큰 인증의 장점

#### 📌 세션 확장에 강함

세션 인증처럼 세션 불일치 문제를 겪을 일이 없다. 토큰 인증은 클라이언트에서 토큰 정보를 저장하기에 서버를 확장하기에 유리하다.
<br/>



#### 📌 세션 부담이 적음

클라이언트에서 토큰 정보를 저장하기에 세션인증과 같이 서버에서 인증 정보를 저장할 필요가 없다.

<br/>


### 토큰 인증의 단점



#### 📌 (세션 인증과 비교해) 높은 트래픽

세션 인증에서 언급했듯이 JWT를 사용한 토큰 인증은 세션 인증보다 높은 트래픽 비용이 든다.



#### 📌 PAYLOAD 보안 문제

`JWT`의 `PAYLOAD`부분에는 유저의 인증 정보가 담겨져있지만, base64로 인코딩 되었을 뿐 암호화가 되지는 않은 영역이다. `PAYLOAD`가 드러나면 유저의 인증 정보가 노출될 위험이 있다. 때문에 암호화한 내용을 토큰에 담는 등 추가적인 보안이 필요하다.

<br/>



## 세 가지 방법 중 어떤 인증 방법을 사용하는 게 좋을까?

각 인증 방법은 장단점을 가지고 있기에 상황에 맞는 인증 방법을 사용하면 좋을 것 같다.

개인적으로는 토큰 인증을 가장 애용하게 될 것 같다. Basic의 경우 암호화 과정이 없으므로 보안에 불리하다는 점이 마음에 걸린다. 세션 인증은 보안에 강력하고 트래픽이 적게 든다는 장점이 있지만, 서버 확장성이 좋다는 토큰의 장점이 더 크게 다가왔다. 

서비스의 규모가 커져서 서버를 확장해야할 일이 생겼을 때 세션 불일치 문제때문에 골머리를 앓는 경우는 상상도 하기 싫다... 차라리 JWT를 사용하되 보안 문제를 더 신경쓰는게 마음 편할 것 같다...

<br/>


## 참고

* https://www.okta.com/kr/identity-101/authentication-vs-authorization/
* https://hudi.blog/session-based-auth-vs-token-based-auth/
* https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/WWW-Authenticate
* https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization
* https://velog.io/@tosspayments/Basic-%EC%9D%B8%EC%A6%9D%EA%B3%BC-Bearer-%EC%9D%B8%EC%A6%9D%EC%9D%98-%EB%AA%A8%EB%93%A0-%EA%B2%83
* https://velog.io/@leesomyoung/HTTP-%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C%EC%9D%98-%ED%8A%B9%EC%A7%95
* https://cl8d.tistory.com/77
* https://jwt.io/introduction
