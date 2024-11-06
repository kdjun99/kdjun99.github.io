---
layout: post
title: apache kafka.01
date: 2024-11-06 10:24 +0900
description: apache kafka 학습내용 정리
image: https://data.it-novum.com/wp-content/uploads/2021/02/apache_kafka_logo-1200x547.png.webp
category: ["kafka"]
tags:
published: true
sitemap: true
author: kim-dong-jun99
---

> [Kafka The Definitive Guide](https://www.confluent.io/resources/kafka-the-definitive-guide/) 책을 통해 학습한 내용을 정리한 글입니다.

## pub/sub messaging

카프카에 대해서 본격적으로 알아보기 이전에, pub/sub 메세징에 대한 개념과 메세징이 왜 data-driven 애플리케이션의 핵심 컴포넌트인지 이해하는 것이 중요합니다.
**publish/subscribe messaging** 센더가 메세지를 리시버에게 직접적으로 전송하지 않는 특징을 가지는 패턴입니다.
대신 센더는 메세지를 어떤 방식으로 분류하고, 리시버는 특정 종류의 메세지를 구독해서 전달받습니다.
pub/sub 시스템은 종종 이런 방식의 메세지 교환을 효과적으로 처리하기 위한 메세지 브로커를 가집니다.

<img width="753" alt="Screenshot 2024-11-06 at 11 32 13" src="https://github.com/user-attachments/assets/c3695d80-035f-419c-b162-38aa87500a93">

애플리케이션이 서로 직접적으로 연결하여 통신한다면, 위와 같이 매우 복잡한 구조를 가지게 됩니다.

<img width="753" alt="Screenshot 2024-11-06 at 11 33 59" src="https://github.com/user-attachments/assets/873f97bd-3ecb-41e5-9173-b23764ea6617">

pub/sub 시스템으로 구성하면, 애플리케이션들은 메세지 브로커를 통해 데이터를 주고 받을 수 있으며, 애플리케이션 간 결합도를 낮출 수 있습니다.

## kafka

Apache Kafka는 분산 스트리밍 플랫폼으로 묘사됩니다.
파일 시스템 혹은 데이터베이스 커밋 로그는 모든 트랜잭션을 내구성 있게 기록하며, 이를 replay함으로써 시스템의 상태를 일관되게 구축할 수 있도록 설계됩니다.
이와 유사하게, Kafka 내의 데이터는 내구성 있게 순서대로 저장됩니다. 또한, 데이터는 시스템 내에서 분산될 수 있어 장애로부터 추가적인 보호를 제공하고, 성능을 확장할 수 있는 중요한 기회를 제공합니다.

### message and batches

Kafka에서 데이터의 단위는 메세지라고 불립니다. 데이터베이스 관점에서 Kafka를 접근하면, row나 레코드와 유사하게 생각할 수 있습니다.
Kafka는 메세지를 단순한 바이트의 배열로 여깁니다. 그렇기에 메세지에 존재하는 데이터는 특정 형식이나 Kafka에게 특정 의미를 가지지 않습니다.
메세지는 key라고 불리는 부가적인 메타데이터를 가질 수 있습니다. 
key 역시 바이트 배열이고, 메세지와 마찬가지로 Kafka에게 특별한 의미를 가지지 않습니다.
Key는 메세지를 파티션하는데 사용됩니다.

효율성을 위해 메세지는 Kafka에 배치로 작성됩니다. **batch**란 같은 토픽과 파티션으로 발행되는 메세지의 컬렉션을 의미합니다. 
메세지 단위로 Kafka에 작성하는 것은 너무 많은 network round trip을 발생하고, 메세지를 batch 단위로 모으는 것은 이를 감소시킵니다.
이렇게 batch 단위로 메세지를 전송하는 것은 latency와 throughput 간의 trade off를 가집니다.
배치가 클수록, 단위 시간 동안 더 많은 메세지를 처리할 수 있지만, 개별 메세지가 전달되는 데 시간이 더 걸리게 됩니다.
배치는 보통 압축되기 때문에 일부 처리 성능을 소모하는 대신, 데이터 전송과 저장이 더욱 효율적으로 이루어집니다.

### schemas

메세지는 Kafka에게 모호한 바이트 배열이지만, 메세지를 이해할 수 있기에, 메세지 내용에 추가적인 스키마를 포함하는 것이 권장됩니다.
메세지 스키마에는 다양한 옵션이 존재합니다.
JSON과 XML 같이 사람이 읽기 편한 옵션도 존재하지만, 이러한 옵션은 강력한 타입 처리나 스키마 버전 간의 호환성 같은 기능이 부족합니다.
많은 Kafka 개발자들은 `Apache Avro`를 주로 사용합니다. Avro는 compact한 직렬화 형식을 제공하며, 메세지 페이로드와 분리된 스키마를 사용하여 스키마 변경 시 코드 생성을 필요로 하지 않습니다. 또한 강력한 데이터 타입과 스키마 진화 기능을 지원하여, 스키마가 변경되더라도 이전 버전과 앞으로의 버전 모두와 호환될 수 있습니다.

Kafka에서 일관된 데이터 형식은 매우 중요합니다. 이를 통해 메세지의 쓰기와 읽기 작업을 분리할 수 있습니다.
이러한 작업이 밀접하게 결합되어 있을 경우, 메세지를 구독하는 애플리케이션은 기존 형식과 함께 새로운 데이터 형식을 처리할 수 있도록 업데이트되어야 합니다.
그 이후에야 메세지를 발행하는 애플리케이션들이 새로운 형식을 사용할 수 있도록 업데이트할 수 있습니다. 잘 정의된 스키마를 사용하고 이를 공용 저장소에 저장함으로써 Kafka의 메세지를 별도의 조정 없이 이해할 수 있습니다.

### topic and partitions

Kafka 메세지들은 topics들로 분류됩니다.
토픽과 가장 비슷한 개념은 데이터베이스 테이블 혹은 파일 시스템의 폴더입니다.
토픽은 추가적으로 파티션으로 분해됩니다.
커밋 로그의 예시로 돌아가보면, 파티션은 하나의 로그 파일로 비유할 수 있습니다.
메세지는 하나의 파일에 추가되고, 읽을 때는 파일의 시작에서 끝나는 지점으로 읽습니다.
> 토픽은 다수의 파티션을 가지고, 토픽 내에서 메세지의 순서는 보장되지 않는 다는 것을 유의해야합니다.<br/>
> 하나의 파티션 내에서만 메세지의 순서가 보장됩니다.

<img width="753" alt="Screenshot 2024-11-06 at 15 49 17" src="https://github.com/user-attachments/assets/58da742d-ab17-43fe-9579-a310514c0024">

위 예시는 4가지 파티션을 가지는 토픽의 예시입니다. 파티션을 이용해서 Kafka는 확장할 수 있고, 데이터를 중복 저장할 수 있습니다.
각 파티션은 서로 다른 서버에 존재할 수 있습니다. 하나의 토픽을 다수의 서버로 수평적으로 확장해 성능을 향상시킬 수 있습니다.
추가적으로 파티션은 복제될 수 있습니다. 서로 다른 서버가 같은 파티션을 복제해 장애에 대비할 수 있습니다.

Kafka 같은 시스템에서 데이터에 대해서 얘기할 때, `stream`이라는 표현이 종종 사용됩니다.
대부분의 경우 스트림은 하나의 토픽 데이터로 고려됩니다.
이는 생산자에서 소비자로 이동하는 단일 데이터 흐름을 나타냅니다. 이 방식으로 메세지를 지칭하는 것은 스트림 처리를 논할 때 흔히 사용됩니다.
스트림 처리란 Kafka Streams, Apache Samza와 같은 프레임워크가 메세지를 실시간으로 처리하는 방식을 의미합니다. 이는 Hadoop 같은 오프라인 프레임워크가 대량의 데이터를 나중에 처리하도록 설계된 방식과 비교될 수 있습니다.

### producers and consumers

Kafka 클라이언트는 두가지 종류가 존재합니다. producers, consumers <br/>
그리고 좀 더 발전된 Kafka connect 같은 API도 존재합니다.
이런 발전된 클라이언트는 producer와 consumer를 기반으로 좀 더 고차원의 기능들을 제공합니다.

`Producers`는 새로운 메세지를 생성합니다. 다른 publish/subscribe 시스템에서는 publishers 혹은 writers라고도 불립니다.
메세지는 특정 토픽으로 발행됩니다. 기본적으로 producer는 메세지를 토픽의 파티션으로 균등하게 조절합니다.
이것은 보통 메세지 키의 해쉬 값을 활용해서 메세지의 파티션을 결정하는 것으로 처리됩니다.
이런 작업은 같은 키의 메세지는 같은 파티션으로 전송되는 것을 보장합니다.

`Consumers`는 메세지를 읽습니다.
다른 publish/subscribe 시스템에서 이런 클라이언트는 subscriber 혹은 readers라고도 불립니다.
consumer는 하나 혹은 그 이상의 토픽을 구독하고 각 파티션에 발행된 메세지를 순서대로 읽습니다.
consumer는 메세지의 offset를 추적하는 것으로 이미 메세지 처리 여부를 추적합니다.
`offset`이란 지속적으로 증가하는 정수 값으로 Kafka가 메세지를 생성할 때 추가하는 메타 데이터의 일부입니다.
파티션의 각 메세지는 유니크한 offset을 가지고, 이후에 추가되는 메세지들은 더 큰 offset 값을 가집니다.

Consumer는 토픽을 소비하는 하나 혹은 그 이상의 consumer로 구성된 `consumer group`의 일부로 동작합니다. 
그룹은 각 파티션이 멤버 중 한 멤버만 소비하는 것을 보장합니다.

<img width="753" alt="Screenshot 2024-11-06 at 19 21 18" src="https://github.com/user-attachments/assets/f8b89890-9984-4732-bed9-607566e01ce8">

위 예시에서 하나의 consumer group에 3개의 consumer로 구성되어있습니다. 위 그림에서 확인할 수 있듯이, 파티션은 그룹내 하나의 consumer만이 소비하는 것을 확인할 수 있습니다.

이런 방식으로 consumer는 많은 수의 메세지를 처리하기 위해 수평적으로 확장할 수 있습니다.
만약 하나의 consumer가 다운되면, 나머지 컨슈머가 파티션을 재할당 받아 다운된 consumer를 대신합니다.

### brokers and clusters

하나의 Kafka 서버는 `broker`라고 불립니다. broker는 producer가 발행한 메세지를 전달받고, 메세지의 offset을 부여하고, 메세지를 디스크에 저장합니다.
또한 consumer의 요청을 처리하여 파티션에 대한 fetch 요청에 응답하고, 발행된 메세지를 제공합니다.
하드웨어의 성능 특성에 따라 단일 브로커는 수천 개의 파티션과 초당 수백만 개의 메세지를 쉽게 처리할 수 있습니다.

Kafka broker는 `cluster`의 일부로 동작하기 위해 디자인됐습니다. 클러스터의 broker 중에서 한 broker는 클러스터 컨트롤러로 동작합니다.
컨트롤러는 broker에 파티션을 할당하거나 broker 장애를 모니터링하는 등 broker들을 관리하는 책임을 가집니다.
파티션은 클러스터내의 하나의 broker에 의해 소유되고 해당 broker를 파티션의 `leader`라고 부릅니다.
복제된 파티션은 추가적인 broker에 할당되고 이런 broker들을 파티션의 `follower`라고 부릅니다. 
이런 파티션의 복제는 broker  장애가 발생했을 때, broker 중 하나가 leadership을 차지할 수 있게합니다.
모든 producer는 메세지를 발행하기 위해 leader와 연결해야하지만, consumer는 leader 혹은 follower 중 하나에 연결하여 메세지를 읽을 수 있습니다.

<img width="753" alt="Screenshot 2024-11-06 at 20 09 00" src="https://github.com/user-attachments/assets/9bdcfa8e-6b91-4bc9-b786-f3a98292f638">

Kafka의 핵심 특성은 `retention`입니다. `retention`은 특정 기간동안의 메세지의 지속적인 저장소를 의미합니다.
Kafka broker들은 기본적으로 topic에 대한 retention을 설정할 수 있습니다. 이는 일주일 같은 시간 단위이거나 1GB 같은 공간 단위일 수도 있습니다.
이런 한계점에 도달하면, 메세지는 삭제됩니다.
이런 retention 설정으로 언제든지 최소한의 데이터가 항상 사용 가능하도록 정의합니다. 개별 토픽은 자체 retention 설정을 통해 필요한 기간 동안만 메세지가 저장되도록 설정할 수도 있습니다.
예를 들어, 추적용 토픽은 며칠 동안 보존될 수 있지만, 애플리케이션 매트릭스는 몇 시간 동안만 보존될 수 있습니다.

> Kafka의 배포 사이즈가 커져갈 수록, 다수의 클러스터를 구성하는 것이 많은 장점을 가집니다.<br/>
> Kafka는 `MirrorMaker`라는 툴을 사용해서 클러스터간 데이터 복제를 합니다.
<img width="753" alt="Screenshot 2024-11-06 at 20 21 05" src="https://github.com/user-attachments/assets/dfa4ad14-26d1-4831-bc4b-e4f77de452cc">

## Kafka의 장점, 왜 Kafka를 쓰는가

### multiple producers

Kafka는 여러 생산자가 많은 토픽을 사용하든 동일한 토픽을 사용하든 원할하게 처리할 수 있습니다. 이로 인해 시스템은 여러 프론트엔드 시스템에서 데이터를 집계하고 이를 일관되게 만드는 데 이상적입니다.

### multiple consumers

Kafka는 다수의 consumer로 서로 영향을 주지 않고 하나의 메세지 스트림을 읽을 수 있습니다.

### disk based retention

메세지는 디스크에 저장되기에, 트래픽이 폭등하는 상황 속에서도 메세지의 유실을 막을 수 있습니다.

### scalable

Kafka는 어떤 양의 데이터도 쉽게 처리할 수 있도록 유연하게 확장 가능합니다.

