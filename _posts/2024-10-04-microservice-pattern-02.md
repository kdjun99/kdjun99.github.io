---
layout: post
title: microservice pattern.02
date: 2024-10-04 10:14 +0900
description: microservice patterns 책을 통해 학습한 내용을 정리한 글입니다.
image:
category: ["msa"]
tags:
published: true
sitemap: true
author: kim-dong-jun99
---

## decomposition strategies

마이크로서비스 구조의 핵심 아이디어는 기능적 분리입니다. 하나의 큰 애플리케이션을 개발하는 대신, 애플리케이션을 여러 서비스의 집합으로 구성합니다.
애플리케이션을 이렇게 여러 서비스의 집합으로 구성하다보면, 몇몇 새로운 의문점들이 등장합니다.

이런 의문점들의 답을 찾기 위해서는 먼저 소프트웨어 아키텍처의 의미를 알아야합니다.

### software architecture

소프트웨어 아키텍처는 다음과 같이 정의할 수 있습니다.
> 컴퓨팅 시스템의 소프트웨어 아키텍처는 시스템에 대해 논리적으로 이해하기 위해 필요한 구조들의 집합으로, 이 구조들은 소프트웨어 요소들, 그들 간의 관계, 그리고 이들 양쪽의 속성들로 구성된다.

다소 추상적인 정의인데, 애플리케이션 아키텍처의 본질은 여러 엘리멘트들로의 분해 그리고 그 엘리멘트들의 관계입니다.

좀 더 구체적으로 살펴보면, 애플리케이션 아키텍처는 다양한 관점에서 바라볼 수 있습니다. Phillip Krutchen은 [4 + 1 view model of software architecture](https://www.cs.ubc.ca/~gregor/teaching/papers/4+1view-architecture.pdf)라는 글을 작성했습니다.
4 + 1 모델에서는 소프트웨어 아키텍처의 4가지 다른 뷰를 정의합니다. 

<img width="758" alt="Screenshot 2024-10-04 at 15 06 49" src="https://github.com/user-attachments/assets/2f32a088-5305-47d9-8e3b-81c38f1e2343">

- logical view : 개발자로부터 만들어지는 소프트웨어 엘리멘트입니다. 객체 지향 언어에서, 이런 엘리멘트들은 클래스 혹은 패키지들입니다. 이런 클래스들은 서로 상속, 의존과 같은 관계를 맺습니다.
- implementation view : 빌드 시스템의 결과물입니다. 이 뷰는 모듈들로 구성됩니다. 자바에서 모듈은 jar 파일 혹은 war 파일로 구분됩니다. 모듈들은 서로 의존 관계 등을 가집니다.
- process view : 런타임 컴포넌트입니다. 각 엘리멘트는 프로세스이고, 이때 관계는 프로세스간 커뮤니케이션을 의미합니다.
- deployment : 이 뷰의 엘리멘트는 머신과 프로세스입니다. 머신 간의 관계는 네트워킹을 의미합니다.
- scenario : 각 시나리오는 다양한 컴포넌트들이 특정 뷰에서 요청을 처리하기 위해 서로 상호작용 하는 것을 나타냅니다. 


### architectural styles

특정 architectural 스타일은 제한된 범위의 엘리멘트와 관계들을 제공하고, 이것들을 사용해 애플리케이션 아키텍처의 뷰를 정의할 수 있습니다. 
애플리케이션은 통상적으로 여러 architectural 스타일을 조합하여 사용합니다.

**layered architectural style**

architectural style의 고전적인 예시는 layered 아키텍처입니다.
레이어드 아키텍처는 소프트웨어 엘리멘트들을 레이어로 조직합니다.
각 레이어마다 명확한 책임이 존재합니다.
레이어드 아키텍처에서는 레이어 간의 의존성을 제약합니다. 레이어는 자신보다 낮은 레이어에만 의존할 수 있습니다.

앞서 언급했던 4가지 뷰 중 어떤 뷰에도 레이어드 아키텍처를 적용할 수 있습니다.
three-tier 아키텍처는 logical 뷰에 layered architecture를 적용한 것입니다.
logical 뷰는 애플리케이션의 클래스를 다음과 같은 레이어로 구성합니다.
- presentation layer : 유저 인터페이스 혹은 외부 API를 구현하는 코드를 포함합니다.
- business logic layer : 비즈니스 로직을 포함합니다.
- persistence layer : 데이터베이스와 상호작용하는 로직을 구현합니다.

레이어드 아키텍처는 architectural 스타일의 좋은 예시이지만, 다음과 같은 분명한 단점을 가지고 있습니다.
- 단일 presentation 레이어 : 애플리케이션이 하나 보다 많은 시스템에서 동작할 수 있다는 점을 간과합니다.
- 단일 persistence 레이어 : 애플리케이션이 하나 보다 많은 데이터베이스와 상호작용할 수 있다는 점을 간과합니다.
- 비즈니스 로직 레이어가 persistence 레이어에 의존하는 것에 기반하여 정의합니다. : 이론적으로 이 의존성으로 인해 데이터베이스 없이 비즈니스 로직을 테스트할 수 없습니다.

**hexagonal architecture style**

hexagonal architecture은 레이어드 아키텍처 스타일의 대안입니다.
<img width="758" alt="Screenshot 2024-10-04 at 15 46 24" src="https://github.com/user-attachments/assets/d20ad931-f5b9-4737-afef-8be95794ca30">
헥사고날 아키텍처는 비즈니스 로직을 중심에 위치시키는 방법으로 logical view를 구성합니다.
presentation layer 대신, 애플리케이션은 하나 혹은 그 이상의 inbound adapters를 가집니다. 
이 inbound adapters들은 비즈니스 로직을 유발하는 외부 요청들을 처리합니다.
data persistence layer와 유사하게, 애플리케이션은 하나 혹은 그 이상의 outbount adapters를 가집니다.
outbound adapters들은 비즈니스 로직에 의해 촉발되고, 다른 외부 애플리케이션을 호출합니다.
이 아키텍처의 핵심은, 비즈니스 로직이 adapter에 의존하지 않습니다. adapter 들이 비즈니스 로직에 의존합니다.

비즈니스 로직은 하나 혹은 그 이상의 포트를 가집니다. 포트는 오퍼레이션의 집합을 정의하고, 비즈니스 로직이 어떻게 외부와 상호작용하는 지를 그 자체입니다.
자바를 예로들면, 포트는 종종 자바 인터페이스입니다.
포트에는 2가지 종류가 존재합니다.
inbound, outbound port가 존재합니다.
inbound port는 비즈니스 로직으로 인해 노출되는 API입니다. 외부 애플리케이션은 이 API를 호출합니다.
inbound port의 예시로는 서비스 인터페이스를 들 수 있습니다. 서비스 인터페이스는 서비스 퍼블릭 메소드를 정의합니다.
outbound port는 비즈니스 로직이 외부 시스템을 호출하는 것을 정의합니다.
outbound port의 예시로는 repository interface를 들 수 있습니다.

adapters들은 비즈니스 로직을 감싸고 있습니다.
포트와 마찬가지로 adapters들도 두가지 종류가 존재합니다.
inbound adapters들은 외부로 부터 들어오는 요청을 inbound port을 호출하는 것으로 처리합니다.
inbound adapters의 예시로는 Spring MVC Controller를 들 수 있습니다.
또 하나의 예시로는 메세지를 구독하는 클라이언트 메세지 브로커를 들 수 있습니다.
다양한 inbound adapter가 같은 inbound port을 호출할 수 있습니다.

outbound adapter는 outbound port를 구현하고, 외부 애플리케이션, 서비스를 호출하는 비즈니스 로직의 요청을 처리합니다.
outbound adapter의 예시로는 데이터베이스 접근을 위한 오퍼레이션을 구현하는 dao 클래스를 들 수 있습니다.
또 하나의 예시로는 원격 서비스를 호출하는 프록시 클래스를 들 수 있습니다.

헥사고날 아키텍처의 중요한 이점은 바로 presentation, data access 로직과 비즈니스 로직을 분리시킨다는 점입니다.
비즈니스 로직은 presentation 로직 혹은 데이터 접근 로직에 의존하지 않습니다.
이로 인해, 비즈니스 로직을 독립되게 테스트하기 무척 쉬워졌습니다.
도 다른 이점은 이 구조가 현대 애플리케이션의 구조에 더 가깝습니다.
비즈니스 로직은 다양한 adapter로부터 호출 될 수 있고, 또 다양한 adapter를 호출할 수도 있습니다.
헥사고날 아키텍처는 마이크로서비스 구조의 서비스를 묘사하기 위한 좋은 방법입니다.

**microservice architecture is an architectural style**

마이크로서비스 구조와 모놀리식 구조는 모두 architectural 스타일입니다.
모놀리식 구조는 단일 컴포넌트로 implementation 뷰를 구성하는 architectural 스타일입니다.

마이크로서비스 구조도 architectural 스타일입니다.
마이크로서비스 구조는 implementation 뷰를 다수의 컴포넌트로 구성합니다. 
이 컴포넌트들은 서비스이고, 커뮤니케이션 프로토콜을 이용해 서비스들끼리 소통합니다.
각 서비스는 고유한 logical view architecture(주로 hexagonal architecture)를 가집니다.

<img width="758" alt="Screenshot 2024-10-04 at 16 18 34" src="https://github.com/user-attachments/assets/acfcd947-76a4-4279-b4f2-38ca73a5a5db">

마이크로서비스 구조의 핵심 제약 사항은 서비스들끼리 느슨하게 결합되어 있어야한다는 점입니다.
결론적으로 서비스들이 협업하는데 규제들이 존재하게 됩니다.
서비스들의 규제에 대해서 알아보기 이전에, 서비스 그리고 느슨하게 결합된 것의 의미에 대해서 알아보겠습니다.

### service

서비스란 독립적으로 배포 가능한 소프트웨어 컴포넌트로 특정 유용한 기능들을 구현합니다.
아래 그림은 서비스의 외부 관점을 보여줍니다.
<img width="758" alt="Screenshot 2024-10-04 at 16 22 03" src="https://github.com/user-attachments/assets/feaaf254-8803-471c-843b-9edd84bf1712">

서비스는 클라이언트에게 기능을 제공하기 위해 API를 가집니다. 
서비스에는 두 가지 종류의 오퍼레이션이 존재합니다.
API는 command, queries, events로 구성됩니다. 
`createOrder()`같은 command는 특정 작업을 처리하고, 데이터를 업데이트합니다.
`findOrderById()`같은 query는 데이터를 반환합니다.
서비스는 `OrderCreated`와 같은 이벤트를 발행할 수도 있습니다.

서비스의 API는 내부 구현을 캡슐화합니다. 
모놀리식과 달리 개발자는 API에 반하는 코드를 작성할 수 없습니다. 그로 인해 마이크로서비스 구조는 애플리케이션의 모듈화를 강제합니다.

마이크로서비스 구조에서 각 서비스는 서비스마다의 기술 스택, 아키텍처를 가집니다.
하지만 보통 서비스는 hexagonal 아키텍처를 가집니다.
API는 adapter들로 구현되고, adapter들은 비즈니스 로직을 호출하고, event adapter는 비즈니스 로직이 방출하는 event를 발행합니다.

### loose coupling

마이크로서비스 구조의 중요한 특성은 서비스들이 느슨하게 결합되어 있다는 점입니다.
서비스들의 모든 상호작용은 내부 구현 디테일이 캡슐화된 API를 통해 이뤄집니다.
느슨하게 결합된 서비스는 애플리케이션의 유지보수성, 테스트 용이성을 향상시키기 위한 중요한 개념입니다.

서비스들이 느슨하게 결합되고, API를 이용해서만 협업하는 것은 데이터베이스를 통해 상호작용하는 것을 금지합니다.
서비스의 저장된 데이터를 클래스의 필드처럼, private하게 두어야 합니다.
데이터를 private하게 유지하는 것은 개발자로 하여금 서비스의 데이터베이스 스키마를 다른 개발자들과의 조율없이 변경하는 것을 가능하게 합니다.
데이터베이스 테이블을 공유하지 않는 것은 런타임의 격리를 향상시킵니다.


### shared libraries

개발자들은 종종 특정 기능을 라이브러리로 제공합니다. 그러면, 다양한 애플리케이션에서 중복된 코드를 작성하지 않고, 재사용 가능합니다. 하지만 이때 서비스들을 결합시키는 것을 주의해야합니다.
여러 서비스가 사용하는 라이브러리의 특정 부분을 변경하면, 여러 서비스가 영향을 받습니다.
그렇기에, 라이브러리에는 변경 가능성이 매우 적은 코드만을 포함해야 합니다.

## defining an application's microservice architecture

애플리케이션은 요청을 처리하기 위해 존재합니다. 그렇기에 아키텍처를 정의하는 첫번째 단계는 애플리케이션의 요구 사항 중 key requests들을 추리는 것입니다.
이때 리퀘스트를 REST나 메세징 같은 특정 IPC 기술을 이용해 설명하기보단, 시스템 오퍼레이션이라는 추상적인 컨셉을 사용합니다.
> system operation이란 애플리케이션이 처리해야하는 요청을 추상화한 것입니다.
> system operation은 command일 수도 있고, query일 수도 있습니다.

두번째 단계는 서비스들로의 분해를 결정하는 것입니다.
이때 몇가지 전략을 선택할 수 있습니다.
전략 중 하나는 비즈니스의 가용성에 맞춰 서비스를 정의하는 것입니다. 다른 전략은 domain driven design 서브 도메인에 맞춰 서비스를 구성하는 것입니다.
결과물로 도출되는 서비스는 기술적인 컨셉보단 비즈니스 컨셉적으로 구성되어야합니다.

세번째 단계는 각 서비스 API를 결정하기 위해 애플리케이션 아키텍처를 정의하는 단계입니다.
세번째 단계를 진행하기 위해서는 먼저, 첫번째 단게에서 정의한 system operation들을 각 서비스에 assign 해야 합니다.
서비스는 한 오퍼레이션을 자체적으로 구현할 수도 있고, 다른 서비스와 함께 협업하여 구현할 수도 있습니다.
협업하는 경우, 어떻게 서비스들이 협업하는지를 결정해야합니다. 더불어서 각 서비스 API를 구현하기 위한 IPC 메커니즘을 결정해야 합니다.

서비스의 분해에는 여러 장애물이 존재합니다. 첫번째는 네트워크 지연입니다. 특정 분해는 너무 많은 서비스들을 거치기에 비 효율적인 것을 파악할 수도 있습니다.
또 다른 장애물은 서비스간 동기적 소통은 가용성을 줄입니다. 
추후에 다룰 self-contained service라는 개념을 사용해야될 수도 있습니다.
세번째 장애물은 서비스들간의 데이터 정합성입니다. 추후에 다룰 saga 같은 것을 사용해야될 수도 있습니다.
마지막 장애물은 god class라는 애플리케이션 전반에 걸쳐 사용되는 클래스입니다. 다행히도 domain-driven design을 이용해서 이런 god class들을 제거할 수 있습니다.

### system operations

애플리케이션의 아키텍처를 정의하는 첫번째 단계는 시스템 오퍼레이션을 정의하는 것입니다.
시작점은 유저 스토리와 유저 시나리오가 포함된 애플리케이션 요구 사항입니다.
시스템 오퍼레이션은 2가지 단계 프로세스를 통해 식별되고 정의됩니다.

<img width="758" alt="Screenshot 2024-10-04 at 17 26 07" src="https://github.com/user-attachments/assets/a2289bb4-6a55-4ea6-b81b-50188462a208">

첫번째 단계는 시스템 오퍼레이션을 묘사하는 어휘들을 제공하는 키 클래스들로 이루어진 하이 레벨 도메인 모델을 만드는 것입니다.
두번째 단계는 시스템 오퍼레이션을 식별하고, 각 행동 방식을 도메인 모델에 맞게 묘사하는 것입니다.

도메인 모델은 주로 유저 스토리들의 명사로부터 도출되고, 시스템 오퍼레이션은 동사로부터 도출됩니다.
도메인 모델은 이벤트 스토밍이라는 기술을 사용해 정의할 수 있습니다.
각 시스템 오퍼레이션의 동작 방식은 도메인 오브젝트들에 미치는 영향 그리고 그들과의 관계로부터 비롯됩니다.
시스템 오퍼레이션은 도메인 오브젝트를 생성하고, 업데이트하고 삭제할 수 있으며, 그들간의 관계를 생성할 수도, 제거할 수도 있습니다.

**high level domain model**

시스템 오퍼레이션을 정의하는 첫번째 단계는 애플리케이션을 위한 high level 도메인 모델을 스케치하는 것입니다.
이때 도메인 모델은 최종적으로 구현될 모델 보다 간단한 버전이라는 것을 유의해야 합니다.

도메인 모델은 유저 스토리, 시나리오에 자주 등장하는 명사를 분석하는 것으로 생성될 수 있습니다.
다음과 같이 주문을 하는 스토리를 예시로 들어보면
```text
Given a consumer
  And a restaurant
  And a delivery address/time that can be served by that restaurant
  And an order total that meets the restaurant's order minimum
When the consumer places an order for the restaurant
Then consumer's credit card is authorized
  And an order is created in the PENDING_ACCEPTANCE state
  And the order is associated with the consumer
  And the order is associated with the restaurant
```

위 유저 시나이로에서 등장하는 명사들은 클래스들의 힌트가 됩니다.
`Consumer, Order, Restaurant, Creditcard`

주문 처리 시나리오는 다음과 같이 작성될 수 있습니다.
```text
Given an order that is in the PENDING_ACCEPTANCE state and a courier that is available to deliver the order
When a restaurant accepts an order with a promise to prepare by a particular time
Then the state of the order is changed to ACCEPTED
  And the order's promiseByTime is updated to the promised time
  And the courier is assigned to deliver the order
```

시나리오를 통해 `Courier, Delivery` 클래스의 존재를 확인할 수 있습니다.
몇번의 추가적인 시나리오와 분석을 통해 다음과 같은 키 클래스들을 포함하는 클래스 다이어그램을 생성할 수 있습니다.
<img width="758" alt="Screenshot 2024-10-04 at 17 43 15" src="https://github.com/user-attachments/assets/9513ab05-ffa2-4079-a6b1-5ac99d4050d3">

각 클래스의 책임은 다음과 같습니다.
- Consumer : 주문을 생성하는 소비자
- Order : 소비자로부터 생성된 주문, 주문의 상태 관리
- OrderLineTime : 주문한 아이템
- DeliveryInfo : 주문을 배달할 시간과 장소
- Restaurant : 소비자에게 배달할 주문을 준비하는 식당
- MenuItem : 식당 메뉴 아이템
- Courier : 배달기사
- Address : 소비자 혹은 식당의 주소
- Location : 배달기사의 위도, 경도

**defining system operations**
high-level domain model을 정의한 이후, 다음 단계는 애플리케이션이 처리해야하는 리퀘스트를 식별하는 것입니다.
웹 애플리케이션의 경우 대부분의 리퀘스트는 HTTP 기반으로 발생하지만, 몇몇 클라이언트는 메세지 기반 통신을 사용할 수 있습니다.
> 최종적으로 시스템 오퍼레이션은 REST, RPC 혹은 메세징 엔드포인트에 대응되지만, 현재는 추상적으로 인식하는 것이 도움됩니다.

시스템 오퍼레이션의 커멘드를 정의할 때는 유저 스토리와 시나리오의 동사를 분석하는 것이 좋은 시작점입니다.
위 주문 생성 시나리오를 예로들면, `CreateOrder`가 제공되야하는 것을 유추할 수 있습니다.
다음과 같은 키 시스템 커멘드가 도출될 수 있습니다.

|actor|story|command|description|
|-----|-----|-------|-----------|
|Consumer|Create Order|createOrder()| 주문 생성|
|Restaurant|Accept Order|acceptOrder()| 식당이 주문을 받고, 일정 시간까지 음식을 준비함을 의미 |
|Restaurant|Order Ready for pickup|noteOrderReadyForPickup()| 주문이 픽업 완료됐음을 의미 |
|Courier|Update location|noteUpdatedLocation()| 배달 기사의 현재 위치를 업데이트 |
|Courier|Delivery picked up|noteDeliveryPickedUp()| 배달기사가 주문을 픽업 |
|Courier|Delivery delivered|noteDeliveryDelivered()| 배달기사가 배달을 완료했음을 알림 |

커멘드는 파라미터, 리턴 값 등등과 연관된 specification을 가지고 있습니다.
`createOrder()`관련 specification은 다음과 같이 정의할 수 있습니다.
- Operation : `createOrder(consumer id, payment method, delivery address, delivery time, restaurant id, order line items)`
- Returns : orderId, ...
- Preconditions : 
  - 고객이 존재하고, 주문을 할 수 있습니다.
  - 주문 아이템들은 식당의 메뉴 아이템과 대응됩니다.
- Post-conditions :
  - 주문은 PENDING_ACCEPTANCE 상태로 생성됩니다.

대부분의 시스템 오퍼레이션은 커멘드이지만,
애플리케이션은 데이터를 반환하는 쿼리도 구현해야합니다.

high-level domain 모델과 시스템 오퍼레이션은 애플리케이션 아키텍처를 정의하는데 도움을 줍니다.
각 시스템 오퍼레이션의 동작은 도메인 모델과 연관되어 묘사됩니다.
각 중요한 시스템 오퍼레이션은 아키텍처적으로 중요한 시나리오를 대표합니다.

시스템 오퍼레이션이 정의된 이후, 그 다음 단계는 애플리케이션의 서비스를 식별하는 것입니다.

### business capability pattern

마이크로서비스 구조를 생성하는 전략 중 하나로는 business capability에 맞춰 분해하는 것입니다.

