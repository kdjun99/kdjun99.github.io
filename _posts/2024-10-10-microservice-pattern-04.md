---
layout: post
title: microservice pattern.04
date: 2024-10-10 21:41 +0900
description: microservice patterns 책을 통해 학습한 내용을 정리한 글입니다.
image:
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


