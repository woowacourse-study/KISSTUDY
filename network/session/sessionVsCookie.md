장바구니 미션을 하면서, Auth를 통해 service는 user를 검증할 수 있다는 사실을 알게되었다.

이를 위해선 cookie, session, token… 다양한 방법들이 있는데 각 의미가 무엇이고 어떻게 연결되는걸까? 그냥 요청 Header에 계정 정보를 넣으면 왜 안 되는걸까?

그리고 언제, 무엇을 써야할까?

알아보자 🔥

# 사용 이유? 계정 정보 그냥 요청 Header에 넣으면 안 되나?
- 보통 server로 Http 요청 보낼 때 따로 암호화하지 않는다
- 따라서, 해커가 HTTP 요청을 가로챈다면 사용자의 계정 정보를 알 수 있게 되기 때문에 보안에 치명적이게 된다
- 기본적으로 HTTP는 stateless인데, 이렇게하면 상태를 가지게 된다. 전혀 RESTful하지가 않은 것

## request와 response의 간단한 구조
- 기본적으로 브라우저가 request를 보내고, server가 response(응답)하는 구조
- 이 response에는 모든 data와 내가 찾는 page 정보가 담겨 있다
    - 쿠키도 있을 수 있어!!

## Cookie
- **cookie를 이용하면 server는 내 browser에 data를 넣을 수 있다**
- sessionID를 전달하기 위한 매개체
- cookie storage(browser, client)에 저장
    - 중요한 정보는 저장하지 않음
- browser가 server에게 요청 보냄 → web server에서 client(browser)에게 cookie를 생성→ browser에 저장하고자 하는 내용 cookie에 key:value 형태로 응답 Header에 담아 보냄 → client가 cookie를 cookie storage에 저장→ 다음 요청할 때마다 cookie를 같이 보내줘야한다
- 쿠키는 도메인에 제한된다 (유튜브가 준 쿠키는 유튜브에만 보내지게 된다)

### 장점

- browser에 저장되기 때문에 많은 사용자가 동시에 접속한다해도 부담없음
- 유효기간있음

### 단점

- 쿠키만으로 인증할 경우 HTTP 요청 해커가 뺏으면 모든 정보가 탈취된다
- 쿠키는 client측에서 조작될 가능성이 있어서 client에서 검증이 필요하고
    - 이를 보완하기 위해선 session을 함께 사용해야한다.
        - 인증의 책임을 server가 지게 할 수 있다!
    - client는 cookie를 이용, server는 cookie를 받아 session의 정보를 접근하는 방식으로 가능!
- JWT와 달리, 길이 제한이 있다

## HTTP
- 기본적으로 HTTP는 stateless
    - server로 가는 모든 요청이 **이전 request와 독립적**으로 다뤄진다
    - request끼리 연결이 없다
    - 메모리가 없다
    - 따라서, 요청이 끝나면 server는 user를 잊어버린다
    - 따라서 *요청할 때 마다 우리가 누군지 알려줘야한다*
        - **이걸 해주는 방법 중 하나가 바로 session!!!**

## Session
- cookie와 마찬가지로 client를 식별하는데 사용
- **server가 가지고 있는 정보**(client의 모든 정보 저장)
    - client는 sessionID만 가진다
- session storage에 저장
- client가 server에 요청 → 올바른 요청이라면(인증을 끝냈다면) server가 해당 client를 위한 session을 생성 → 이 session안에는 sessionID도 존재(이걸로 각 session들을 구분) → 해당 sessionID는 cookie를 통해 broswer로 돌아오고 저장된다 → 다른 페이지로 가도 browser는 sessionID를 가지고 있는 cookie를 server에게 보내게 된다→ server는 해당 sessionID를 가지고 session DB를 확인 → 이제서야 server가 내가 누군지 알게된다.
- client와 server는 sessionID가 담긴 cookie만을 주고 받는다
- (보안이 엄청 중요한 서비스에서는 session만을 사용하기도 한다)

### 장점

- client에 정보를 저장하지 않고, server측에 정보를 저장하고 관리하기 때문에 보안이 좋다
- DB가 있다
    - 모든 정보를 session storage에 저장하기 때문에, 기능 추가 가능
        - 넷플릭스처럼 계정 공유 숫자를 제한하거나
        - 인스타그램의 로그인 된 기기들 중 하나를 삭제한다거나

### 단점

- (server에 있는) session storage를 사용하기때문에, 동시에 많은 client가 접속하면 server의 부담이 높아진다
    - 해결책으로는 session에 최적화된 redisDB를 주로 사용
    - 혹은, Token을 사용할 수 있다
- client가 처음 방문했던 server와 다른 server에 요청을 보내면 인증 실패하게 됨
    - 해결책 : redisDB의 세션 통합
    - 혹은, JWT 사용
- 로그인한 유저들의 모든 sessionID를 session storage에 저장하고 있다
    - 요청이 들어올 때 마다, server는 cookie를 받아서 sessionID를 보고 일치하는 유저를 찾은 다음에야 다음 작업을 수행할 수 있다
    - 즉, 요청이 있을 때 마다 DB를 찔러야한다
    - 유저가 늘어나면 DB도 더 커져야해!
        - JWT가 해결 가능

### session을 사용해 IOS, Android Application을 만들 수 있다. 하지만, cookie는 사용할 수 없음 >> browser에만 있기 때문! 바로 이 경우, token을 사용하게 된다. server에 cookie대신 token을 보낸다고 생각하면 될듯.

## Token
- client와 server간 인증을 위한 **문자열**
    - sessionID보다 훨씬 길다 (제약이 없어서 엄청 길어도 된다)
- 보통 JWT(Json Web Token)형식으로 사용
- client가 인증 정보를 포함한 token을 생성하여 server에 전송 → server는 해당 token과 일치하는 유저를 찾음 (유효한 요청이라면 server는 DB에 뭔가 생성하는 게 없다) → email이든지 password든지 가져가서 sign algorithms을 통해 sign한다 → signed Info(사인된 정보)를 string 형태로 보낸다 → request를 다시 보낼 땐 signedInfo 혹은 token을 server에 보내야한다 → server는 token을 가지고 해당 sign이 유효한지 체크(token 조작 여부 체크) → 유효하다면 server가 인증.
- 보통 OAuth, OpenId Connect 등의 프로토콜에서 사용된다

### 장점

- session이 가졌던 단점 해결
    - session stroage 필요 없음
    - server가 많은 일을 하지 않아도 됨
- DB를 건드리지 않아서 간단하다
- 유효성이 짧아 보안성이 높다
- client에 저장되기 때문에 server 부하가 줄어든다
- server가 여러대 일 때 강점을 가진다
- DB없이 데이터를 사인하고, 유저에게 보내고, 해당 데이터를 돌려받을 때 유효성 검증이 가능
    - ex) QR 코드

### 단점

- JWT는 암호화 되지 않았다
    - 누구나 열어서 해당 컨텐츠를 볼 수 있다
    - 비밀정보를 JWT안에 두면 안 됨
- DB가 없기 때문에 session의 장점에서 언급한 그런 기능들은 불가

## Session vs JWT  정리하자면?
### session

- sessionID를 담는 cookie만 있으면 된다
- session에 대한 정보는 session storage라는 DB에 저장
    - request오면 server는 sessionID를 여기서 찾는다

### JWT

- 인증하는데 필요한 정보를 token에 저장
- token을 browser에 준다
- request 오면, DB를 거치지 않고 server는 해당 token을 sign하고 이를통해 유효한지만 검증
