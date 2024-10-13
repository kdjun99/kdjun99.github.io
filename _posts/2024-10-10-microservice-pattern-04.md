---
layout: post
title: microservice pattern.04
date: 2024-10-10 21:41 +0900
description: microservice patterns 책을 통해 학습한 내용을 정리한 글입니다.
image: https://velog.velcdn.com/images/chancehee/post/80fefca2-c70a-4740-a434-0ca140002f4a/image.png
category: ["msa"]
tags:
published: true
sitemap: true
author: kim-dong-jun99
---

## communicating using the asynchronous messaging pattern

메세징을 사용할 때, 서비스는 비동기적으로 메세지를 교환하는 방식으로 소통합니다.
메세지 기반 애플리케이션은 통상적으로 메세지 브로커를 사용합니다. 메세지 브로커는 서비스간 중간 계층으로 동작하고, 브로커 없이 서비스끼리 직접적으로 소통하는 방식으로 동작할 수도 있습니다.
서비스 클라이언트는 메세지를 전송하는 것으로 서비스로 요청을 보냅니다. 만약 서비스 인스턴스로부터 응답이 와야한다면, 클라이언트로 별개의 메세지를 전송할 것입니다. 
통신은 비동기적으로 이뤄지기에, 클라이언트는 응답을 기다리는 동안 block되지 않아도됩니다.

### overview of messaging

`Enterprise Integration Patterns`라는 책에서 효과적인 매세지 모델링을 정의했습니다.
효과적인 매세지 모델링에서 메세지는 메세지 채널을 통해 교환됩니다.
센더(애플리케이션 혹은 서비스)는 채널에 메세지를 전송하고 리시버(애플리케이션 혹은 서비스)는 채널로부터 메세지를 읽습니다.

**messages**

메세지는 헤더와 바디로 구성됩니다. `헤더`는 이름-값 쌍의 컬렉션으로, 전송되는 데이터를 묘사하는 메타 데이터입니다.
메세지 헤더는 센더 혹은 메세징 인프라에서 생성되는 유니크한 메세지 아이디를 가지고, 선택적으로 응답이 작성되야할 return address가 포함됩니다.
메세지 바디는 전송될 데이터로 텍스트 기반 혹은 바이너리 형식입니다.

메세지에는 다양한 타입이 존재합니다.
- document : 제네릭 메세지로 데이터만을 포함합니다. 리시버가 어떻게 메세지를 해석할지를 결정합니다. 커멘드에 대한 답변이 다큐먼트 메세지의 예시입니다.
- command : RPC 리퀘스트와 동일한 메세지입니다. 실행할 오퍼레이션과 파라미터를 명시합니다.
- event : 센더에서 어떤 일이 발생했음을 의미하는 메세지입니다. 이벤트는 종종 고객, 주문과 같은 도메인 변경 사항을 나타내는 도메인 이벤트입니다.

**message channels**

메세지는 채널을 통해 교환됩니다.

<img width="758" alt="Screenshot 2024-10-10 at 18 17 30" src="https://github.com/user-attachments/assets/18077c13-0c83-47f0-95ca-58ce44b17e22">

센더에 있는 비즈니스로직은 내부 커뮤니케이션 로직을 캡슐화하는 `sending port` 인터페이스를 호출합니다.
`sending port`는 메세지를 메세지 채널을 통해 리시버로 전달하는  `message sender` 어댑터 클래스를 통해 구현됩니다.
`message channel`은 메세지 인프라구조를 추상화한 것입니다. 
리시버에 있는 `message handler` 어댑터 클래스는 메세지를 처리하기 위해 호출됩니다.
컨슈머의 비즈니스로직으로 구현되는 `receiving port` 인터페이스를 호출합니다.
단일 혹은 다수의 센더가 채널로 메세지를 보낼 수 있고 이와 유사하게 단일 혹은 다수의 리시버가 채널로부터 메세지를 읽을 수 있습니다.

채널에는 두가지 종류가 존재합니다.
- point-to-point: point-to-point 채널은 채널에서부터 메세지를 읽는 컨슈머 중 단 하나의 컨슈머에게 메세지를 전달합니다. one-to-one 상호작용 스타일을 사용하기 위해 point-to-point 채널을 사용합니다. 커맨드 메세지가 주로 point-to-point 채널로 전송됩니다.
- publish-subscribe : publish-subscribe 채널은 채널에 연결된 모든 컨슈머에게 메세지를 보냅니다. one-to-many 상호작용 스타일을 적용하기 위해 이 채널을 사용합니다. 이벤트 메세지가 주로 이 채널을 이용해 전송됩니다.

### implementing the interaction styles using messaging

메세징의 중요한 특성 중 하나는 앞서 다뤘던 모든 상호작용 스타일을 지원할 수 있다는 점입니다.
몇몇 상호작용 스타일은 메세징을 이용해 직접적으로 구현되고, 나머지는 메세징을 기반으로 구현할 수 있습니다.

**implementing request/response and asynchronous request/response**

클라이언트와 서비스가 request/response 혹은 비동기 request/response를 이용해 상호작용할 때, 클라이언트는 리퀘스트를 전송하고, 서비스는 응답을 리턴합니다.
두 상호작용 방식간의 차이점은 request/response 방식은 클라이언트의 즉각적인 응답을 기대하고, 비동기 request/response는 그런 기대를 하지 않습니다.
메세징은 근본적으로 비동기적으로 동작하기에, 비동기 request/response 방식을 제공합니다.
하지만 클라이언트는 응답이 오기전까지 block할 수도 있습니다.

클라이언트와 서비스는 비동기 request/response 스타일 상호작용을 메세지 쌍을 교환하는 것으로 구현합니다.
<img width="758" alt="Screenshot 2024-10-10 at 19 24 41" src="https://github.com/user-attachments/assets/4d958abd-7ba5-4e55-9ce2-eab45b01a4e6">

클라이언트는 수행해야할 오퍼레이션과 파라미터를 명시한 커멘드 메세지를 서비스가 보유한 point-to-point 채널로 전송합니다.
서비스는 리퀘스트를 처리하고, 결과를 담은 응답 메세지를 클라이언트가 보유한 point-to-point 채널로 전송합니다.

클라이언트는 서비스에 응답 메시지를 어디로 보낼지 알려주어야 하며, 응답 메시지를 요청과 일치시켜야 합니다.
다행히도 이 두 문제를 해결하는 것은 어렵지 않습니다.
클라이언트는 커맨드 메세지에 `reply channel` 헤더를 포함해서 전송합니다. 서버는 응답 메세지를 보낼 때, `message id`와 동일한 값을 가지는 `correlation id`를 포함합니다.
클라이언트는 `correlation id` 값으로 메세지와 응답을 매칭합니다.

클라이언트와 서비스가 메세징을 이용해 소통하기에, 상호작용은 근본적으로 비동기적으로 동작합니다. 
이론적으로 메세징 클라이언트는 응답을 받기전까지 block할 두 있습니다. 하지만, 실제로는 클라이언트가 응답을 비동기적으로 처리합니다.
게다가 응답은 일반적으로 클라이언트 인스턴스 중 하나가 처리합니다.

**implementing one-way notifications**

one-way notifications은 비동기 메세징을 이용해 간단하게 구현할 수 있습니다.
클라이언트는 커맨드 메세지를 point-to-point 채널로 보냅니다. 채널을 구독한 서비스는 메세지를 처리하고 응답을 보내지 않습니다.

**implementing publish / subscribe**

메세징은 pub/sub을 위한 빌트인 지원을 포함하고 있습니다.
클라이언트는 다수의 컨슈머가 읽어가는 pub/sub 채널로 메세지를 발행합니다.
서비스는 pub/sub 을 사용해서 도메인 이벤트를 발행합니다. 
도메인 이벤트를 발행하는 서비스는 그 이름으로부터 유래된 pub/sub 채널을 소유합니다.
예를 들어 `OrderService`클래스는 `Order`이벤트를 `Order` 채널로 발행합니다.
특정 도메인 오브젝트에 관심있는 서비스들은 그 특정 채널에만 subscribe하면 됩니다.

**implementing publish/async responses**

publish/async 리스폰스 상호작용 스타일은 pub/sub과 request/response의 엘리멘트를 혼용하여 구현한 상위 레벨의 상호작용 스타일입니다.
클라이언트는 `reply channel` 헤더를 명시한 메세지를 pub/sub 채널로 발행합니다.
컨슈머는 `correlation id`를 포함한 응답 메세지를 응답 채널에 작성합니다.
클라이언트는 응답 메세지와 요청과 매치하기 위해 `correlation id`로 응답을 모읍니다.

비동기 API를 사용하는 애플리케이션의 각 서비스는 하나 이상의 구현 기술을 사용할 것입니다. 작업을 호출하기 위한 비동기 API를 가진 서비스는 요청을 위한 메시지 채널을 갖게 됩니다. 마찬가지로 이벤트를 발행하는 서비스는 이벤트 메시지를 이벤트 메시지 채널에 발행하게 됩니다.

### creating API specification for messaging based service API

비동기 API의 명세는 메세지 채널 명과 채널에서 교환하는 메세지 타입과 형식을 정의해야 합니다.
<img width="758" alt="Screenshot 2024-10-10 at 21 10 33" src="https://github.com/user-attachments/assets/557a0d8f-4608-41cf-88d8-a229fe46406c">
JSON, XML, Protobuf 같이 주고 받는 메세지 형식을 명시해야합니다. 
하지만 REST, OpenAPI와는 다르게, 채널과 메세지 타입을 명시하고 문서화하는 표준이 존재하지는 않습니다.
대신 문서를 작성합니다.

**documenting asynchoronous operations**

서비스 오퍼레이션은 두가지 상호작용 스타일을 이용해서 호출될 수 있습니다.
- request/async response-style API : 이 방식은 서비스의 명령 메세지 채널, 서비스가 수락하는 명령 메세지 유형과 형식, 그리고 서비스가 보내는 응답 메세지 유형과 형식으로 구성됩니다.
- one-way notification-style API : 이 방식은 서비스의 명령 메세지 채널과 서비스가 수락하는 명령 메세지 유형과 형식으로 구성됩니다.

하나의 서비스가 비동기 요청/응답과 단방향 알림 모두를 위해 동일한 요청 채널을 사용할 수 있습니다.

**documenting published events**

서비스는 pub/sub 상호작용 스타일을 이용해 이벤트를 발행할 수 있습니다.
이런 스타일의 API 명세사항은 이벤트 채널과, 주고 받는 이벤트 타입, 그리고 형식입니다.

메시지와 채널 기반의 메시징 모델은 훌륭한 추상화이며, 서비스의 비동기 API를 설계하는 좋은 방법입니다. 하지만 서비스를 구현하려면 특정 메시징 기술을 선택하고, 그 기술의 기능을 활용하여 설계를 구현하는 방법을 결정해야 합니다.

### using a message broker

메세징 기반 애플리케이션은 통상적으로 서비스가 통신하기 위한 인프라구조 서비스인 메세지 브로커를 사용합니다.
브로커 기반 구조가 유일한 메세징 구조는 아닙니다.
브로커를 사용하지 않고 서비스끼리 직접적으로 소통하는 방식도 존재합니다.
각 방식의 장단점은 존재하지만, 보통은 브로커 기반 구조가 더 나은 접근입니다.

<img width="758" alt="Screenshot 2024-10-10 at 21 50 58" src="https://github.com/user-attachments/assets/5286bed0-7e9a-4151-b51b-b6ea8ec030d2">
> broker less 방식은 네트워크 지연이 덜하고, (브로커를 거치지 않기에) 메세지 브로커로 인한 병목, SPOF가 발생하지는 않지만, 
> 감소된 가용성, 그리고 서비스 디스커버리의 필요하다는 점들이 동기적 request/response와 동일하기에 자주 사용되지 않습니다.

메세지 브로커는 모든 메세지가 흐르는 중간계층입니다.
센더는 메세지를 메세지 브로커에 전송하고, 브로커가 리시버에게 전달합니다.
브로커를 사용하는 이점은, **센더가 컨슈머의 네트워크 위치를 몰라도 된다**는 점입니다.
또 다른 이점은 **메세지 브로커는 컨슈머가 메세지를 처리할 수 있을 때까지 메세지를 대기**시킨다는 점입니다.

다음과 같이 다양한 메세지 브로커가 존재합니다.
- [ActiveMQ](https://activemq.apache.org/)
- [RabbitMQ](https://www.rabbitmq.com/)
- [Apache Kafka](https://kafka.apache.org/)

혹은 AWS Kinesis나 AWS SQS 같은 클라우드 기반 메세지 브로커도 존재합ㅂ니다.

메세지 브로커를 선택할 때, 다음과 같은 고려사항들을 고려하여 선택해야합니다.
- supported programming languages : 브로커가다양한 프로그래밍 언어를 지원하는지 
- supported messaging standards : 브로커가 AMQP, STOMP 같은 표준을 지원하는지
- messaging ordering : 브로커가 메세지의 순서를 보장하는지
- delivery guarantees : 브로커가 어떤 방식의 전송 보장을 제공하는지
- persistence : 메세지가 디스크에 저장되고, 브로커가 중지되어도 휘발되지 않는지
- durability : 컨슈머가 브로커에 다시 연결될 때, 연결되지 못했을 때 발행된 메세지들도 받을 수 있는지
- scalability : 브로커가 얼마나 확장 가능한지
- latency : end-to-end 지연시간이 얼마나 되는지
- competing consumers : competing consumers를 지원하는지

각 브로커는 다른 장단점을 가집니다. 예를 들어 매우 낮은 지연시간을 제공하는 브로커는 메세지의 순서를 보장하지 못하거나, 메세지의 전송 보장을 제공하지 못할 수 있습니다.
전송을 보장하고 디스크에 메세지를 저장하는 브로커는 지연 시간이 상대적으로 클 수 있습니다.

**implementing message channels using a message broker**

메세지 브로커마다 메세지 채널의 개념을 서로 다른 방식으로 구현합니다.

| Message Broker | Point-to-point channel | Publish-subscribe channel |
|----------------|------------------------|---------------------------|
| JMS | Queue | Topic |
| Apache Kafka | Topic | Topic |
| AMQP-based brokers, such as Rabbit MQ | Exchange + Queue | Fanout exchange and a queue per consumer |
| AWS Kinesis | Stream | Stream |
| AWS SQS | Queue | X |

거의 대부분의 메세지 브로커는 point-to-point 채널과 publish-subscribe 채널을 지원합니다.

브로커 기반 메세징의 장점은 다음과 같습니다.
- **loose coupling** : 클라이언트는 서비스 인스턴스를 몰라도 적합한 메세지 채널에만 리퀘스트를 보내면 됩니다. 서비스 디스커버리 메커니즘이 필요 없습니다.
- **message buffering** : 메세지 브로커는 메세지가 처리될까지 버퍼에 저장할 수 있습니다. HTTP 같은 동기적 request/response 프로토콜은 요청과 응답이 교환될 때, 클라이언트와 서비스 모두 available 해야됩니다. 메세징에서는 consumer에서 메세지가 처리될 때까지 메세지를 큐에 저장할 수 있습니다. 온라인 쇼핑몰을 예시로 들면, 주문 재고 처리 시스템이 느리거나 가용하지 않은 상황에서도 주문을 처리할 수 있습니다. 메세지는 큐에 쌓일 것이고, 나중에 사용가능할 때 처리 될 것입니다.
- **flexible communication** : 메세징은 앞서 언급된 모든 상호작용 스타일이 가능합니다.
- **explicit interprocess communication** : RPC 기반 메커니즘은 원격 서비스를 호출하는 것을 로컬 서비스를 호출하는 것과 동일해 보이도록 시도했습니다. 하지만 실상은 좀 다릅니다. 메세징은 로컬 서비스와 명시적으로 구분되므로, 개발자들이 잘못된 안전감에 빠지지 않도록 합니다.

단점은 다음과 같습니다.
- **potential bottleneck** : 메세지 브로커가 성능의 병목이 될 수 있지만, 메세지 브로커는 확장 가능합니다.
- **potential single point of failure** : 시스템의 신뢰도에 영향을 미치지 않으려면, 고가용성이 제공되야하는데, 다행히도 메세지 브로커는 고가용성을 제공합니다.
- **additional operational complexity** : 메세징 시스템은 또 다른 시스템 컴포넌트이기에, 설치되고, 설정되고 운영되야합니다.

### competing receivers and message ordering

메세지 브로커의 한가지 과제는 어떻게 메세지의 순서를 보장하면서 리시버를 확장하는 것입니다.
메세지를 동시에 처리하기 위해서는 다수의 서비스 인스턴스가 요구됩니다.
더불어서 단일 서비스 인스턴스더라도 스레드를 사용해 다수의 메세지를 동시에 처리할 수 있습니다.
다수의 스레드와 서비스 인스턴스를 이용해서 동시에 메세지를 처리하는 것은 애플리케이션의 처리량을 향상시킵니다.
하지만 메세지를 동시에 처리하는 것의 과제는 메세지가 한번만 처리되고 순서를 보장하는 것입니다.

예를 들어, 3개의 서비스 인스턴스가 같은 point-to-point 채널에서 메세지를 읽고 센더는 주문 생성, 주문 업데이트, 주문 취소 이벤트 메세지를 순차적으로 발행합니다.
간단한 메세징 구현은 각 메세지를 다른 리시버에게 동시에 전달할 수 있습니다. 네트워크 이슈나, 가비지 컬렉션 같은 이유로 딜레이가 발생할 수 있고, 메세지는 순서대로 처리되지 않을 수 있습니다.
이론적으로 한 서비스 인스턴스가 주문 취소 메세지를 다른 인스턴스가 주문 생성 메세지를 처리하기 전에 처리할 수도 있습니다.

Apache Kafka나 AWS Kinesis 같은 현대 메세지 브로커는 `shared channel`을 사용해서 이런 문제를 해결합니다.

<img width="758" alt="Screenshot 2024-10-11 at 10 16 18" src="https://github.com/user-attachments/assets/589b5cac-cb9c-422a-a3d5-6509aa7ebd2d">

1. shared channel은 각자가 채널처럼 동작하는 두개 이상의 shard로 구성됩니다.
2. 센더는 통상적으로 임의의 문자열이나 바이트의 시퀀스인 shard key를 메세지 헤더에 명시합니다. 브로커는 shard key를 사용해서 메세지를 특정 shard, 파티션에 할당합니다. 해쉬나 모듈러 연산을 수행해서 shard를 선택할 수도 있습니다.
3. 메세징 브로커는 여러 리시버 인스턴스를 묶고 그들을 같은 논리적 리시버로 다룹니다. Apache Kafka는 `consumer group` 이라는 표현을 사용합니다. 메세지 브로커는 각 샤드에 하나의 리시버를 할당합니다. 리시버가 시작하거나 종료될 때 샤드를 재할당합니다.

### handling duplicate message

또 다른 과제는 중복 메세지를 처리하는 것입니다.
메세지 브로커는 이상적으로 메세지를 단 한번만 처리해야하는데, 단 한번만 메세지 처리를 보장하는 것은 너무나 비용이 큽니다.
대신에 대부분의 메세지 브로커는 메세지가 적어도 한번 처리되는 것을 보장합니다.

시스템이 정상적으로 동작할 때, 적어도 한번 전달을 보장하는 메세지 브로커는 각 메세지를 한 번만 전달합니다.
하지만 클라이언트, 네트워크 혹은 메세지 브로커의 장애로 인해 메세지는 여러번 처리될 수 있습니다.
클라이언트가 메세지를 처리하고, 데이터베이스를 업데이트한 후 하지만 메세지의 ack를 전송하기 전에 장애가 났다고 해봅시다.
메세지 브로커는 ack을 받지 못한 메세지를 장애가 난 클라이언트가 재실행되면 재전송하거나 클라이언트의 다른 레플리카에 전송할 것입니다.

이상적으로 메세지를 재전송할 때, 메세지의 순서를 보장하는 브로커를 사용해야 합니다.
클라이언트가 동일한 주문에 대해 먼저 주문 생성 이벤트를 처리한 후 주문 취소 이벤트를 처리했는데, 어떤 이유로 주문 생성 이벤트가 확인되지 않은 상황을 가정해봅시다.
메세지 브로커는 주문 생성과 주문 취소 이벤트를 재전송해야합니다. 
만약 주문 생성 이벤트만 재전송된다면, 주문 취소 이벤트가 실행되지 않을 수 있습니다.

중복 메세지를 처리하는데는 다양한 방법들이 존재합니다.
- 멱등성을 가진 메세지 핸들러를 작성합니다.
- 메세지를 추적하고 중복된 메세지는 버립니다.

**writing idempotent message handlers**

만약 메세지를 처리하는 애플리케이션 로직이 멱등성을 가진다면, 중복된 메세지는 문제가 되지 않습니다.
애플리케이션 로직이 같은 인풋 값을 주어지고 여러번 호출했을 때, 발생하는 추가적인 문제가 없다면 애플리케이션 로직이 `멱등성을 가진다`라고 합니다.
예를들어, 이미 취소된 주문을 취소하는 것은 멱등성을 가지는 오퍼레이션입니다. 
메세지 브로커가 메세지를 재전송할 때, 메세지 순서만 보장해준다면, 멱등성을 가지는 메세지 핸들러는 안전하게 여러번 수행할 수 있습니다.

하지만 불행히도 애플리케이션 로직은 종종 멱등성을 가지지 않습니다.
혹은 메세지 브로커가 메세지를 재전송할 때, 메세지의 순서를 지키지 않는 메세지 브로커를 쓸 수 도 있습니다.
중복되거나 순서가 어긋난 메세지는 버그를 발생시킬 수 있습니다.
이런 상황에서는 메세지를 추적하고 중복된 메세지는 제거하는 메세지 핸들러를 사용해야합니다.

**tracking messages and discarding duplicates**

고객의 신용카드를 승인하는 메세지 핸들러를 생각해봅시다.
이 경우에는 주문 당 정확히 한번만 카드를 승인해야합니다.
이런 경우에는 애플리케이션 로직이 실행될때마다 영향을 미치게 됩니다.
만약 중복된 메세지가 메세지 핸들러로 하여금 같은 로직을 여러번 실행하게 하면, 애플리케이션은 틀린 방식으로 동작하게 됩니다.
이런 애플리케이션 로직을 처리하는 메세지 핸들러는 중복 메세지를 확인하고 제거함으로서 멱등성을 가져야합니다.

간단한 해결 방법은 메세지 컨슈머로 하여금 `message id`를 이용하여 이미 처리한 메세지를 추적하고, 중복을 제거하는 것입니다.
<img width="758" alt="Screenshot 2024-10-11 at 14 00 18" src="https://github.com/user-attachments/assets/de5bbe62-b4e3-4c75-a317-ed7376f671ed">

메세지 컨슈머는 자신이 소비한 메세지의 아이드를 데이터베이스 테이블에 저장할 수 있습니다.
컨슈머가 메세지를 처리할 때, `message id` 값을 비즈니스 엔티티를 생성하거나 업데이트하는 트랜잭션에 포함시켜 기록합니다.
위 예제에서는 컨슈머가 `PROCESSED_MESSAGES` 테이블에 소비한 `message id` 값을 추가했습니다.
만약 메세지가 중복된다면, `INSERT` 작업은 실패할 것이고, 컨슈머는 메세지를 제거할 것입니다.

또 다른 옵션은 메세지 핸들러가 `message id`를 애플리케이션 테이블이 아닌 지정된 테이블에 기록하는 것입니다.
이런 접근 방법은 NOSQL 데이터베이스처럼 제한된 트랜잭션 모델을 가질 때 효과적입니다.

### transactional messaging

서비스는 종종 데이터베이스를 업데이트하는 트랜잭션의 일부로 메세지를 발행해야 하는 경우가 있습니다.
지금까지 비즈니스 엔티티가 생성되거나 수정되는 도메인 이벤트를 발행하는 서비스의 예시들을 보았습니다.
데이터베이스의 업데이트와 메세지의 발행이 반드시 트랜잭션 내부에서 진행되야합니다.
그렇지 않다면, 서비스는 데이터베이스만 업데이트하고 장애가 발생해 메세지를 보내지 못하는 경우도 발생합니다.
서비스가 이 두 오퍼레이션을 원자적으로 수행하지 못한다면, 이 장애로 인해 시스템의 consistency는 깨지게 됩니다.

전통적인 해결방법은 데이터베이스와 트랜잭션을 아우르는 분산 트랜잭션을 사용하는 것 입니다.
하지만, 이후에 다룰텐데 분산 트랜잭션은 현대 애플리케이션에 적합하지 않습니다.

결과적으로 애플리케이션은 다른 메커니즘을 사용해서 이런 문제를 해결해야 합니다.

**using a database table as a message queue**

관계형 데이터베이스를 사용하는 애플리케이션이 있습니다. 메세지를 발행하는 간단한 방법은 **Transactional outbox pattern**을 적용하는 것 입니다.
이 패턴은 데이터베이스 테이블을 임시 메세지 큐처럼 사용합니다.

<img width="758" alt="Screenshot 2024-10-11 at 14 21 06" src="https://github.com/user-attachments/assets/88bc1fa7-a63a-4a3f-a90c-7ae0b7b11935">

메세지를 전송하는 테이블은 `OUTBOX`라는 테이블을 가집니다. 비즈니스 오브젝트를 생성, 수정, 삭제하는 데이터베이스 트랜잭션의 부분으로 서비스는 `OUTBOX`테이블에 메세지를 추가하는 것으로 메세지를 전송합니다.
ACID 트랜잭션 내에서 동작하기에, 원자성이 자동으로 보장됩니다.

`OUTBOX` 테이블은 임시 메세지 큐처럼 동작합니다.
`MessageRelay`는 `OUTBOX`테이블로부터 메세지를 읽는 컴포넌트이고, 메세지 브로커로 메세지를 발행합니다.

이와 유사한 접근 법을 몇몇 NoSQL 데이터베이스에 적용할 수 있습니다.
`record`로 저장된 각 비즈니스 엔티티는 발행되야하는 메세지의 리스트를 속성으로 가지고 있습니다.
서비스가 데이터베이스 내 엔티티를 업데이트하면, 엔티티의 메세지 리스트에 메세지가 추가됩니다.
단일 데이터베이스 오퍼레이션이기에 이 동작은 원자적입니다.
다만 이벤트를 가지고 발행할 비즈니스 엔티티를 효율적으로 찾는 것이 과제로 남긴합니다.

데이터베이스로부터 메세지를 브로커로 전송하는 다른 방법들도 조냊합니다.

**publishing events by using the polling publisher pattern**

만약 애플리케이션이 관계형 데이터베이스를 사용한다면, `OUTBOX` 테이블로 추가된 메세지를 발행하는 가장 심플한 방법은 `MessageRelay`로 하여금 테이블에서 발행되지 않은 메세지를 뽑는 것입니다.
주기적으로 테이블에 다음과 같은 쿼리를 실행합니다.
```sql
SELECT * FROM OUTBOX ORDERED BY ... ASC
```

그리고 `MessageRelay`는 조회된 메세지를 목적지 채널로 보내는 것으로 메세지를 브로커에 발행합니다.
최종적으로 `OUTBOX` 테이블에 발행된 메세지를 제거합니다.
```sql
BEGIN
  DELETE FROM OUTBOX WHERE ID in (...)
COMMIT
```

데이터베이스에서 메세지를 추출하는 것은 서비스의 규모가 작을 때 사용할 수 있는 간단한 접근 방법입니다.
이것의 단점은 데이터베이스에서 자주 메세지를 추출하는 것은 비용이 크다는 점입니다.
또한 이 접근 방식을 NoSQL 데이터베이스에서 사용할 수 있는지는 해당 데이터베이스의 쿼리 기능에 따라 다릅니다.
`OUTBOX` 테이블을 쿼리하는 대신 애플리케이션이 비즈니스 엔티티를 쿼리해야 하기 때문에, 이것이 효율적으로 가능한지 여부가 문제입니다.
이러한 단점과 제한 사항으로 인해, 더 정교하고 성능이 좋은 방법은 데이터베이스 트랜잭션 로그를 후미에서 처리하는 방식이 종종 더 나은 선택이며, 경우에 따라서는 필수적일 수 있습니다.

**publishing events by applying the transaction log tailing pattern**

좀 더 정교한 솔루션은 `MessageRelay`로 하여금 데이터베이스 트랜잭션 로그를 `tail`하는 것입니다.
애플리케이션이 만드는 모든 커밋된 업데이트 내용들은 데이터베이스 트랜잭션 로그에 기록됩니다.
트랜잭션 로그 마이너는 로그를 읽고 각 변경 사항을 메세지로 브로커에 발행할 수 있습니다.

<img width="758" alt="Screenshot 2024-10-11 at 14 42 46" src="https://github.com/user-attachments/assets/8a157ffc-f6f4-4f33-a8e8-a86c5aca3038">

`TransactionLogMiber`는 트랜잭션 로그 엔트리를 읽습니다. 마이너는 삽입된 메세지와 대응되는 로그 엔트리를 메세지로 변환하고 각 메세지를 브로커에 발행합니다.
이런 접근 방법은 RDBMS `OUTBOX` 테이블에 작성된 혹은 NoSQL 데이터베이스의 레코드로 추가된 메세지들을 발행할 수 있게 합니다.

이런 접근 방법의 한가지 과제는, 개발자의 노력이 좀 필요하다는 것입니다.

### libraries and frameworks for messaging

서비스는 메세지를 발행하고, 수신하기 위해서는 라이브러리를 사용해야합니다.
한가지 접근 방법은 메세지 브로커의 클라이언트 라이브러리를 사용하는 것입니다.
하지만, 라이브러리를 직접 사용하면 몇가지 문제점들이 발생합니다.
- client library는 메세지 브로커 API에 메세지를 발행하는 비즈니스 로직들을 결합시킵니다.
- 메세지 브로커의 클라이언트 라이브러리는 통상적으로 low level이고 메세지를 보내거나, 수신하는데 많은 양의 코드가 필요합니다. 개발자로서, 반복되는 boilerplate 코드는 원하지 않습니다. 
- 클라이언트 라이브러리는 보통 메세지를 전송하고 수신하는 기본적인 메커니즘만 제공합니다. 

보다 더 나은 접근 방법은 low-level 디테일을 감추고 바로 higher-level 상호작용 스타일을 지원하는 high-level 라이브러리나 프레임워크를 사용하는 것 입니다.
> 이 책에서는 Eventuate Tram 프레임워크를 사용합니다.

`Eventuate Tram`은 다음과 같은 두 중요한 메커니즘을 구현합니다.
- Transactional messaging : 데이터베이스 트랜잭션의 일부로 메세지를 발행합니다.
- Duplicate message detection : Eventuate Tram 메세지 컨슈머는 중복 메세지를 탐지하고 제거합니다.

### Eventuate Tram

**basic messaging**

기본적은 메세징 API는 두가지 자바 인터페이스로 구성됩니다. `MessageProducer` 그리고 `MessageConsumer`
프로듀서 서비스는 `MessageProducer` 인터페이스를 사용해서 메세지를 메세지 채널로 발행합니다.
```java
MessageProducer messageProducer = ...;
String channel = ...;
String payload = ...;
messageProducer.send(destination, MessageBuilder.withPayload(payload).build());
```

컨슈머 서비스는 `MessageConsumer` 인터페이스를 사용해서 메세지를 구독합니다.
```java
MessageConsumer messageConsumer;
messageConsumer.subscribe(subscriberId, Collections.singleton(destination),
message -> { ... });
```

`MessageProducer`와 `MessageConsumer`는 비동기 request/response와 도메인 이벤트 발행을 위한 higher level API의 토대가 됩니다.

**domain event publishing**

Eventuate Tram은 도메인 이벤트를 발행하고 소비하는 API들을 가지고 잇습니다.
> 도메인 이벤트는 aggregate(비즈니스 객체)가 생성, 수정, 삭제될 때 발생하는 이벤트임을 설명합니다.

서비스는 `DomainEventPublisher`인터페이스를 이용해서 도메인 이벤트를 발행합니다.
```java
DomainEventPublisher domainEventPublisher;
String accountId = ...;
DomainEvent domainEvent = new AccountDebited(...);
domainEventPublisher.publish("Account", accountId, Collections.singletonList(
domainEvent));
```

서비스는 도메인 이벤트를 `DomainEventDispatcher`를 이용해서 소비합니다.
```java
DomainEventHandlers domainEventHandlers = DomainEventHandlersBuilder
  .forAggregateType("Order")
  .onEvent(AccountDebited.class, domainEvent -> { ... })
  .build();

new DomainEventDispatcher("eventDispatcherId",
  domainEventHandlers,
  messageConsumer);
```

**command / reply-based messaging**

클라이언트는 서비스에게 커맨드 메세지를 `CommandProducer` 인터페이스를 사용해 보낼 수 있습니다.
```java
CommandProducer commandProducer = ...;
Map<String, String> extraMessageHeaders = Collections.emptyMap();
String commandId = commandProducer.send("CustomerCommandChannel",
  new DoSomethingCommand(),
  "ReplyToChannel",
  extraMessageHeaders);
```

서비스는 `CommandDispatcher` 클래스를 이용해서 커맨드 메세지를 소비합니다.
`CommandDispatcher`는 `MessageConsumer` 인터페이스를 사용해 특정 이벤트를 구독합니다.
`CommandDispatcher`는 각 명령 메세지를 적절한 핸들러 메서드로 디스패치합니다.

```java
CommandHandlers commandHandlers =CommandHandlersBuilder
  .fromChannel(commandChannel)
  .onMessage(DoSomethingCommand.class, (command) -> { ... ; return withSuccess(); })
  .build();

CommandDispatcher dispatcher = new CommandDispatcher("subscribeId", commandHandlers, messageConsumer, messageProducer);
```

앞서 확인한 것처럼, `Eventuate Tram` 프레임워크는 자바 애플리케이션을 위해 transactional messaging을 구현합니다.
이 프레임워크는 메시지를 트랜잭션 방식으로 송수신하기 위한 저수준 API를 제공하며, 도메인 이벤트를 발행하고 소비하거나 명령을 송신하고 처리하기 위한 고수준 API도 제공합니다.

