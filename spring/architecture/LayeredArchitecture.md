## SW와 Architecture. Architecture를 쓰는 이유?

---

- SW는 복잡하다. 큰 계획 없이 개발하다보면 얽히고 설킨 구조의 코드가 만들어진다
    - 하지만 SW는 주기적으로 변경이 필요하다
    - 변경하고 싶다면, 이 얽히고 설킨 구조를 끊어내고 변경해주고, 다시 얽히고 설킨 구조로 돌려놔야한다
    - 따라서 SW는 변경에 용이해야한다
- 아키텍처를 적용하면 구조가 잘 정리된다
    - 변경에 용이해진다

## Architecture의 목표

---

- SW를 쉽게 변경할 수 있는 구조로 설계해 유지보수를 쉽게 함
- Domain을 보호하고, 집중하게끔 만들기 위함

## Architecture 적용

---

SW의 큰 구성은 Domain, Infra Structure로 나눌 수 있다

![스크린샷 2023-04-17 오후 2.12.08.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5a178078-c54e-49cc-8863-adac88741346/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-04-17_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_2.12.08.png)

**Domain**: SW를 통해 이루고자하는 핵심적인 요소들 

ex) 주문 접수 채널 운영시 주문을 접수하는 프로세스, 정책이 Domain이다

**InfraStructure**: Domain을 SW로 제공하기 위한 것들 

ex) UI, DB, API

## Hierarchical Architecture

---

- 계층형 아키텍처
- 목적이 같은 코드들을 계층으로 그룹화(관심사의 분리)
    
    ![스크린샷 2023-04-17 오후 2.18.06.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/834bb521-1712-47c3-9635-5c5eef7cf712/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-04-17_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_2.18.06.png)
    
- 하지만 위 그림과 같이 직접적으로 계층을 참조하는 경우, 몇 가지 문제점이 있다
    - **연쇄적인 참조관계**: presentation Layer → Domain Layer → Persistence Layer이 결국 Presentation Layer → Persistence Layer이 되는 것
        - 결과적으로 Persistence Layer의 data를 변경하면 그 변경의 영향이 Domain Layer → Presentation Layer로 영향미친다
        - 이런 변경의 영향을 “의존성을 가진다”라고 한다
        - 이렇게 되면 코드 변경이 어려워진다. 한 곳을 변경하면 그것을 참고하고 있던 곳에도 변경에 영향이 전파되기 때문
        - 테스트도 쉽지 않다
            - Domain Layer를 test하려면 영속성게층의 data를 모두 준비시켜야한다
            - test를 위해 많은 준비가 필요하다
        - 보호받아야 할 Domain Layer안의 업무 Logic들이 여러 계층의 변경에 영향을 받는다

## Clean Architecture

---

- Hierarchical Architecture의 의존성 문제가 보완된 것
- 이 4개의 원중에서 가장 중요한 항목은 Use Cases, Entities
    - 이 2개의 원에 Business 규칙이 정의되어 있음
    - Entities: Domain 객체
    - Use Cases: Service. Domain 참조해 구현
- 초록색, 파란색은 Use Cases, Entities에서 정의된 걸 어떻게 보여주고 저장할 것인지 정의
- 의존성의 방향에 주목
    - 화면/data 등의 하부 세부사항(Controllers, DB, UI…)들이 고수준 정책(Use cases, Entities)을 단방향으로 의존
    - 그래야 바깥쪽이 변경되더라도 domain은 영향을 받지 않는다

presenter(어댑터): Controller

Use case: service

Entity: domain 객체

## Clean Architecture를 적용한 Hierarchical Architecture

---

![스크린샷 2023-04-17 오후 2.52.55.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e50f53b3-dc34-40cb-86c0-b8f25ba8cb34/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-04-17_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_2.52.55.png)

- Use case는 양쪽에 Adapter를 통해 외부와 연결됨
    - Adapter: 220V를 110V로 바꿔주는 Adapter와 매우 유사. 외부의 data를 Domain에서 사용할 수 있는 data 형식으로 변환
        - Presenter: 화면에 입력된 걸 Domain에 맞는 data로 변환
        - Repository: 외부의 DB, API data를 Domain에 맞는 data로 전환
- 의존성의 방향이 모두 Domain을 향해있다
    - 하지만, Repository는 의존성이 반대로 향하게 되는데 이를 의존성 역전이라 한다
    - Usecase와 Adapter 사이에는 Interface가 존재
        - UseCase, Presenter, Repository 모두 interface를 구현해야함
        - ex) View에서 사장님이 주문접수, 조리시간 입력
        - Presenter에서 (Interface에서 정의된) 주문을 접수하다를 호출
        - Use Case(Service)는 (Interface에서 정의된) 주문 정보를 조회하라 호출
        - Entity를 통해 주문 접수 처리로 상태 변경
        - Repository는 (Interface에서 정의된) 주문 정보를 저장하라를 호출, Remote Data Source, Local Data Source(Local DB)에 저장
        

### 장점

- 외부에서 변경되어도 Domain Layer는 변경에 영향을 받지 않는다
    - 의존성이 안쪽으로만 향하기 때문
- Domain Layer 양측에 Interface를 구현시, Local DB를 원격 DB로 바꾸는 등 큰 작업도 진행 가능
- 의존성을 낮추면 각 요소를 독립적으로 Test할 수 있다
    - Domain Entity와 Use case는 참조하는 것이 없어서 독립적으로 Test 가능

## Hexagonal Architecture

---

![스크린샷 2023-04-17 오후 4.50.27.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/37280856-845a-4d74-851d-e3972b7cb2f5/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-04-17_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_4.50.27.png)

- 육각형 아키텍쳐
- Clean architecture를 구현하는 방법을 구체화한 아키텍처
- Use case, Entity가 육각형 안에 위치
    - 외부의 변화로부터 보호하기 위함
- 육각형의 경계에는 Interface가 감싸는 구조
    - 왼쪽은 Adapter에서 들어오는 입력을 받는 Input Port
    - 오른쪽은 처리 결과를 저장할 Output Port
- 외부 Adapter들은 Use case, Entity에 직접 접근불가. 반드시 Interface로 정의된 Port를 통해 접근
- 안, 밖의 관심사를 분리하고 독립적으로 각자의 역할에 충실하기 위함
