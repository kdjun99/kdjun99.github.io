---
layout: post
title: microservice pattern.01
date: 2024-10-03 10:57 +0900
description: microservice patterns 책을 통해 학습한 내용을 정리한 글입니다.
image:
category: ["msa"]
tags:
published: true
sitemap: true
author: kim-dong-jun99
---

## monolithic hell

모놀리식 구조에서 시스템은 주로 하나의 실행 파일 혹은 배포 컴포넌트로 구성됩니다.
자바로 애플리케이션이 작성됐다면, 애플리케이션은 하나의 WAR로 패키징 될 것이고, golang으로 작성됐다면, 하나의 실행 파일이 생성될 것입니다.

애플리케이션 규모가 상대적으로 작을 때, 애플리케이션의 모놀리식 구조는 다음과 같은 장점을 가졌습니다.
- 개발하기 쉽다 : ide와 여러 다른 개발자 도구들은 하나의 애플리케이션을 개발하는데 초점이 맞춰있습니다.
- 급진적인 변경 사항을 만들기 쉽다 : 코드를 변경하고, 데이터베이스 스키마를 변경하고, 빌드하고 배포하면 됐습니다.
- 테스트 과정이 비교적 간단 명료하다 : end-to-end 테스트를 작성하고, rest api를 테스트하고, ui 테스트를 selenium으로 진행할 수 있습니다.
- 배포 과정이 비교적 간단 명료하다 : 톰캣 서버가 설치된 곳에 WAR 파일을 복사하기만 하면 됐습니다.
- 확장에 용이하다 : 로드 밸런서를 앞단에 두고 다수의 애플리케이션을 실행할 수 있습니다.

불행히도, 시간이 지나며 개발 과정, 테스트, 배포 과정 그리고 확장에 모놀리식 구조는 어려움을 겪게됩니다.

> 책에서는 FTGO의 사례를 들어서 설명합니다.
> FTGO 같은 성공적인 애플리케이션은 모놀리식 구조보다 몸집이 커가는 문제가 발생합니다.
> 각 스프린트마다 코드 베이스는 점점 더 커져갔고, 비즈니스는 성공하며 개발 팀의 규모는 커져갔습니다.
> 코드 베이스의 규모가 커져갈 때마다 유지보수 오버헤드도 덩달아 증가했습니다.
> 코드 베이스의 규모가 증가하며, 시스템은 너무 복잡해져서 어느 개발자도 전체를 이해하기 어려웠습니다.
> 그로인해, 버그를 고치고, 새로운 기능을 개발하는데 더 많은 시간이 필요하게됩니다.
>
> 복잡성 증가는 더 큰 문제를 불러오는데, 이해하기 어려운 코드는 개발자로 하여금 변경사항을 깔끔하게 만들기 어렵게합니다.
> 각 변경 사항들은 코드를 더 복잡하게 만들었고, 코드의 복잡성은 점점 더 증가했습니다.
>
> 더불어서 코드 베이스가 너무 커지며, 테스트 코드의 양도 방대해졌고, 전체 테스트 사이클을 돌리는데 몇일씩이 소요되게 되었습니다.
> 테스트 과정이 이렇게 복잡해지며, 애플리케이션 전체를 테스트하지 못하게되는 경우도 생겼고, production 코드에 버그가 포함되는 경우도 발생하게 됐습니다.
>
> 마지막으로 새로운 프레임 워크, 언어를 적용하는 것이 너무 어려웠습니다.
> 모놀리식 애플리케이션 전체를 새로운 언어, 프레임워크로 작성하는 것은 너무 risky한 작업이였습니다.
> 그렇기에 새로운 기술의 도입이 어려웠고, 개발자들은 종종 사용 시기가 지난 기술 스택으로 애플리케이션 개발을 진행해야 했습니다.

## scale cube and microservices

책에서는 마이크로서비스 아키텍쳐의 정의를 scale cube를 이용해 설명합니다.

![https://microservices.io/i/DecomposingApplications.021.jpg](https://microservices.io/i/DecomposingApplications.021.jpg)

**X 축 : horizontal duplication**
- X 축으로의 확장은 모놀리식 애플리케이션의 확장 방법과 공통됩니다. 로드밸런서 뒤에 여러 애플리케이션 인스턴스를 실행할 수 있습니다.
- 로드 밸런서는 N 개의 애플리케이션 인스턴스들로 요청을 분배합니다.
<img width="672" alt="Screenshot 2024-10-03 at 12 34 09" src="https://github.com/user-attachments/assets/963b0750-0216-415e-a011-eed131dac581">


**Z 축 : data partitioning**
- Z 축으로의 확장도 마찬가지로 여러 모놀리식 애플리케이션을 실행하는 것과 유사하지만, 각 인스턴스는 데이터의 일부에만 책임을 가지고 있습니다.
- 인스턴스 앞단의 라우터가 리퀘스트의 attribute을 확인하고, 적합한 인스턴스로 route합니다.
<img width="756" alt="Screenshot 2024-10-03 at 12 36 12" src="https://github.com/user-attachments/assets/9e89c2dd-2873-4b4f-8caa-3564bdc61b6d">

**Y 축 : functional decomposition**
- Z축과 X축으로의 확장은 애플리케이션의 capacity와 availiability를 향상시킵니다.
- 하지만 그 중 어느 것도 개발 과정과 애플리케이션의 복잡성 문제를 해결하는 것은 없습니다.
- 이것을 해결하기 위해서는 functional decomposition이 필요합니다.
- 다음과 같이 애플리케이션을 서비스 별로 분리하는 것이 필요합니다.
<img width="758" alt="Screenshot 2024-10-03 at 12 41 03" src="https://github.com/user-attachments/assets/451ab52d-d693-4403-84a1-63b421078cb6">

`서비스`란 집중된 기능을 구현하는 미니 애플리케이션입니다. 서비스는 x축 확장을 이용해 확장되고, 몇몇 서비스는 z축 확장을 이용해 확장 될 수도 있습니다.
마이크로서비스 구조의 개념적 정의는 애플리케이션을 여러 서비스들로 분해하는 architectural 스타일입니다.
정의에 사이즈에 대한 언급은 찾아볼 수 없는 것에 유의하고, 각 서비스가 명확한 책임을 갖는 것이 중요합니다.

### modularity

모듈화는 복잡하고 큰 애플리케이션을 개발할 때 필수적입니다. 현대 애플리케이션은 개인에 의해 개발되기에는 너무 큰 규모이고, 한 사람이 전부 이해하기에는 너무 복잡합니다.
애플리케이션은 모듈로 분화되고, 각 모듈은 서로 다른 사람들에게 이해되고 개발됩니다.

모놀리식 애플리케이션에서 모듈은 자바 패키지 같이 프로그래밍 언어의 구성과 자바 jar 파일 같은 빌드 아티펙트로 정의됩니다.
마이크로서비스 아키텍쳐에서는 모듈화의 단위로 서비스를 사용했습니다. 서비는 API를 가지는데, API는 자바 패키지 내부 클래스처럼 쉽게 접근하고 내부 구현을 변경할 수 업습니다.
그렇기에 애플리케이션에서 모듈화를 제공하기에 훨씬 수월합니다.

### service with its own database

마이크로서비스 아키텍쳐의 핵심적인 특징 중 하나는 서비스는 느슨하게 연결되어있고 API를 통해 통신한다는 것입니다.
서비스간의 결합도를 낮추는 방법 중 하나는 서비스 별로 각자의 데이터 저장소를 가지는 것입니다.
온라인 스토어를 예로들면, `orderSevice`는 `ORDERS`테이블을 포함한 데이터베이스를 가지고, `customerService`는 `CUSTOMERS`테이블을 포함한 데이터베이스를 가집니다.
개발시에 개발자들은 서비스의 스키마를 다른 서비스에서 개발하는 개발자들과 조율 없이 변경할 수 있습니다.
런타임에는 각 서비스들은 서로 독립되어서 실행됩니다.
한 서비스는 다른 서비스가 데이터베이스 락을 획득하고 있기에 block되는 상황이 발생하지 않습니다.

## benefits and drawbacks of the microservice architecture

마이크로서비스 아키텍쳐는 다음과 같은 장점을 가집니다.
- continuous delivery가 가능하고, 큰 규모의 복잡한 애플리케이션의 배포가 가능합니다.
- 서비스는 쉽게 관리, 유지됩니다.
- 서비스는 독립적으로 배포 가능합니다.
- 서비스는 독립적으로 확장 가능합니다.
- 새로운 기술의 도입이 쉽습니다.
- 재해 대비가 상대적으로 쉽습니다.

마이크로서비스 구조의 가장 중요한 이점은 continuous delivery와 규모가 크고 복잡한 애플리케이션의 배포입니다.
마이크로서비스 구조는 3가지 방법을 이용해 continuous delivery와 deployment을 가능하게 합니다.
- testability : 자동화된 테스트는 cd의 중요한 부분입니다. 각 서비스는 비교적 작은 규모이기에 테스트의 실행은 상대적으로 쉽고 실행도 빠릅니다. 결과적으로 애플리케이션 버그를 줄일 수 있습니다.
- deployability : 각 서비스는 다른 서비스와 독립적으로 배포될 수 있습니다. 만약 개발자가 서비스내에만 영향을 미치는 변경 사항에 대한 책임을 지고 있다면, 다른 개발자와 조율 없이 개발할 수 있습니다. production 환경에 더 쉽고 자주 배포할 수 있습니다.
- autonomous and loosely coupled development teams : 개발 조직을 작은 팀들로 구성할 수 있습니다. 각 팀은 독립적으로 개발하고, 배포하고 서비스를 확장할 수 있습니다.

마이크로서비스는 장점만 가지지는 않고, 몇가지 단점도 가지는 구조입니다.
> 이후에 마이크로서비스 구조가 가지는 단점도 해결하는 방법에 대해서 다룰 것입니다.
- 적합한 서비스 구성을 찾는 것이 어렵습니다.
- 분산 시스템은 복잡합니다. 개발과 테스트, 그리고 배포 난이도가 올라갑니다.
- 특정 기능을 배포하는 것은 여러 서비스에 영향을 미치므로 세심한 조율이 필요합니다.
- 마이크로서비스 구조의 도입 시점을 결정하는 것이 어렵습니다.

분산 시스템을 개발하는 것은 복잡성을 증가시킵니다.
서비스는 interprocess communication 메커니즘을 사용해야합니다.
interprocess communication은 간단한 메소드 호출보다 복잡한 방식으로 동작합니다.
더불어서 서비스는 원격 서비스가 응답하지 않거나, 높은 지연 시간으로 응답하는 것에 대한 처리를 해야합니다.

각 서비스는 서비스만의 데이터베이스를 가지기에, 트랜잭션이나 서비스간 쿼리를 작성하기 어렵습니다.
서비스들간 데이터 정합성을 맞추기 위해 `saga`라는 것을 사용하거나, 각 서비스에로부터 데이터를 받아오지 않으려면 `cqrs`을 도입해야합니다.

## microservice arcitecture pattern language

패턴이란 특정 상황에서 발생하는 문제에 대한 재사용 가능한 해결책입니다. 
real-world architecture에서 유래된 개념으로 소프트웨어 아키텍처와 디자인에서 유용함이 증명된 개념입니다.
software design pattern은 서로 상호작용하는 소프트웨어 엘리멘트들의 집합을 정의하는 것으로 소프트웨어 아키텍쳐 혹은 다자인 문제를 해결합니다.
> 예를들어, 여러가지 초과 인출 정책을 지원하는 뱅킹 애플리케이션을 개발하려고 할 때, 각 정책에 대한 제한 사항이 있을 수 있습니다.
> 이런 문제를 strategy pattern을 이용해서 해결할 수 있습니다.

패턴이 중요한 이유 중 하나는 패턴은 패턴이 적용될 context를 반드시 설명해야합니다.
솔루션이 특정 상황에만 적합하고 다른 상황에서는 적용되지 않을 수 있다는 아이디어가 기술에 대한 논의에 비해 발전된 부분입니다.
> 항상 모든 것을 완벽하게 해결하는 기술은 사실상 없습니다. 특정 기술은 특정한 상황에서 발생하는 문제를 해결할 수 있지만, 그에 대비되는 단점도 존재합니다.
> 몇몇 사람들은 자신이 좋아하는 기술이 모든 것에 대한 완벽한 해결책인 것 처럼 주장하곤 하지만, 실상을 그렇지 않습니다.

패턴의 가치는 현재 문제가 되는 상황에 대한 충분한 고려 뿐만 아니라, 종종 간과되는 솔루션의 크리티컬한 단점들을 묘사하게합니다.
자주 쓰이는 패턴 구조들은 다음과 같은 3가지 섹션을 포함합니다.
- forces
- resulting context
- related patterns

### forces : the issues that you must address when solving a problem

패턴의 forces 섹션은 주어진 상황에서 문제를 해결할 때 반드시 논의해야하는 forces(이슈들)을 묘사합니다.
forces들은 서로 충돌할 수도 있습니다, 모든 forces들을 해결하는 것은 불가능할 수도 있습니다.
어느 force가 더 중요한지는 context에 의존합니다.
어떤 force를 해결할지, force 별로 우선 순위를 정해야합니다.
> 코드는 가독성이 좋아야하며, 좋은 성능을 보여야합니다. 리액티브 스타일로 작성된 코드는 동기적으로 작성된 코드보다 좋은 성능을 보여주지만, 가독성이 떨어집니다.
forces들을 명시적으로 나열해야 해결해야할 이슈들이 명시적으로 보이기에 유용합니다.

### resulting context : the consequences of applying a pattern

resulting context 섹션은 패턴을 적용했을 때의 결과들을 묘사합니다.
3가지 부분으로 구성됩니다.
- benefits : 패턴의 이점, 해결된 forces들을 포함합니다.
- drawbacks : 패턴의 단점, 해결되지 않은 forces들을 포함합니다.
- issues : 패턴을 적용하면서 새롭게 발생하는 문제들
resulting context 섹션은 솔루션에 대해 덜 편향된 관점을 제공합니다.

### related patterns : the five different types of relationships

related patterns 섹션은 패턴과 다른 패턴들 간의 관계를 묘사합니다.
패턴들 간의 관계에는 5가지 타입이 있습니다.
- predecessor : predecessor 패턴은 이 패턴이 필요하게 느끼게끔한 패턴을 의미합니다. 마이크로서비스 구조가 다른 패턴들의 predecessor 패턴이라고 할 수 있습니다.
- successor : 이 패턴으로 인해 발생한 문제를 해결한 패턴을 successor 패턴이라 합니다.
- alternative : 다른 솔루션을 제공하는 패턴입니다.
- generalization : 어떤 문제에 대한 일반적인 솔루션인 패턴입니다.
- specialization : 특정 패턴의 특별한 형태입니다.

다음과 같이 패턴들 간의 관계를 나타낼 수 있습니다.
<img width="758" alt="Screenshot 2024-10-03 at 16 49 59" src="https://github.com/user-attachments/assets/71ce984b-7f29-4e29-8448-25a71b207da8">
- predecessor, successor 관계를 나타냅니다.
- 같은 문제에 대한 대체 패턴을 확인 가능합니다.
- 한 패턴이 다른 패턴의 특별한 형태임을 의미합니다.
- 특정 문제에 적용되는 패턴들을 확인 가능합니다.

microservice architecture pattern language는 마이크로서비스 구조를 사용하는 애플리케이션을 설계하는데 도움이 되는 패턴들의 모음입니다.
pattern language는 우선 마이크로서비스 구조를 도입할지 말지를 결정하는데 도움을 줍니다. 모놀리식 구조와 마이크로서비스 구조의 장점과 단점을 묘사합니다. 
만약 마이크로서비스 구조가 더 적합하다면, pattern language는 다양한 아키텍쳐와 디자인 이슈들을 해결하는데 도움을 줍니다.
<img width="758" alt="Screenshot 2024-10-03 at 16 57 01" src="https://github.com/user-attachments/assets/b31d1ce7-1f42-4620-89d8-1f0cdef99a47">

microservice architecture pattern은 3가지 레이어로 구분됩니다.
- infrastructure patterns : 대부분 개발 외적, 인프라 구조 관련 이슈들을 해결합니다.
- application infrastructure : 개발에도 영향을 미치는 인프라 이슈들을 해결합니다.
- application patterns : 개발자가 직면하는 문제를 해결하는 패턴입니다.

**pattern for decomposing an application into services**

어떻게 시스템을 서비스로 분리할지는 매우 추상적인데, 도움을 받을 수 있는 몇가지 전략 들이 존재합니다. 

<img width="758" alt="Screenshot 2024-10-03 at 17 02 59" src="https://github.com/user-attachments/assets/96320f92-2d0a-42e0-b312-c0db19ee3768">

언급된 패턴들은 추후에 더 다룰 예정입니다.

**communication patterns**

마이크로서비스 구조를 이용해 개발된 애플리케이션은 분산 시스템입니다.
그로인해 interprocess communication (IPC)가 마이크로서비스 구조에서 중요한 부분이 되게됩니다.
서비스들끼리 그리고 외부 세상과 어떻게 통신할지에 관한 여러 아키텍쳐, 디자인 관련 결정을 해야합니다.
- communication style : 어떤 IPC 메커니즘을 사용할 것인지
- discovery : 어떻게 서비스의 클라이언트가 서비스의 IP 주소를 결정할 것인지
- reliability : 서비스가 가용하지 않은 상태에서도 어떻게 서비스의 신뢰도를 보장할지
- transactional messaging : 어떻게 메세지와 이벤트 통신과 데이터 베이스 트랜잭션을 통합시킬 것인지
- external API : 어떻게 클라이언트가 서비스들과 통신할 것인지

<img width="758" alt="Screenshot 2024-10-03 at 17 08 23" src="https://github.com/user-attachments/assets/87e4ab61-c5cc-40d2-ae7b-59cbbb9aa35e">

**data consistency patterns for implementing transaction management**

느슨한 결합을 보장하기 위해, 각 서비스는 서비스만의 데이터베이스를 가집니다. 불행히도 서비스 별로 데이터베이스를 가지는 것은 중대한 이슈를 발생시킵니다.
분산 데이터베이스들 간의 데이터 정합성을 맞춰야하고, 이를 위한 여러 패턴이 존재합니다.

**data consistency patterns for implementing transaction management**

서비스별로 데이터베이스를 가지면서 발생하는 또 하나의 문제는 여러 서비스들이 보유한 데이터를 조인해서 쿼리를 실행해야하는 경우가 종종 있습니다.
서비스의 데이터는 API를 통해서만 접근 가능합니다. 그렇기에 쿼리를 위한 여러가지 패턴들이 존재합니다.
<img width="758" alt="Screenshot 2024-10-03 at 17 14 18" src="https://github.com/user-attachments/assets/a5918792-e883-4ca7-aebe-fcb3c72e72c5">
<img width="758" alt="Screenshot 2024-10-03 at 17 14 26" src="https://github.com/user-attachments/assets/de421252-f59a-4086-ab51-6572265abdb7">

**service deployment patterns**

모놀리식 애플리케이션을 배포하는 것은 항상 쉬운 일은 아닙니다. 하지만, 하나의 애플리케이션만 배포하면 되기에 간단 명료합니다.
그에 반해 마이크로서비스 애플리케이션은 무척 복잡합니다. 다양한 언어와 프레임워크로 작성된 수십, 수백개의 서비스가 있을 수 있습니다.
<img width="758" alt="Screenshot 2024-10-03 at 17 18 14" src="https://github.com/user-attachments/assets/95171578-00f5-411f-8565-863d059c79cc">

**observability patterns provide insight into application behavior**

애플리케이션의 런타임 행동 방식과 런타임에서 발생하는 문제 해결은 애플리케이션 운영 부분의 핵심입니다.
모놀리식 애플리케이션을 이해하고, 트러블슈팅하는 것이 항상 쉽지는 않았지만, 리퀘스트는 간단 명료하게 처리되긴 합니다.
각 요청은 로드밸러서로 인해 분배되어 특정 애플리케이션 인스턴스에 도달하고, 몇번의 데이터베이스 호출 이후 리턴됩니다.
하지만 마이크로서비스는 보다 더 복잡하게 처리됩니다.
리퀘스트는 리턴되기 이전에 다양한 서비스들로 이동할 수 있고, 가장 중요한 점은 살펴봐야할 로그 파일이 하나가 아닙니다.
지연 시간 문제가 발생해도 원인을 찾기 쉽지 않습니다.

다음 패턴들은 서비스를 관찰하는데 사용됩니다.
- health check API : 서비스의 상태에 대한 정보를 노출하는 엔드포인트를 노출
- exception tracking : 예외를 exception tracking service에 보고, 예외에 대한 중복을 제거하고, 개발자들에게 알림
- application metrics  
- audit logging

**patterns for automated testing of services**

마이크로서비스 구조는 각 서비스 애플리케이션의 규모가 모놀리식 애플리케이션보다 작기에, 테스트하기 보다 수월합니다.
동시에 다른 서비스 애플리케이션과 함께 동작하는 것을 테스트하는 것이 중요합니다.

> 이외에도, 보안 관련 패턴, 서비스 디스커버리 패턴 같은 것들이 존재합니다.


