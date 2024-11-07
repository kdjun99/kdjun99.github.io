---
layout: post
title: apache kafka.02
date: 2024-11-07 11:20 +0900
description: zookeeper, kafka 설치 및 설정에 대하여
image: https://data.it-novum.com/wp-content/uploads/2021/02/apache_kafka_logo-1200x547.png.webp
category: ["kafka"]
tags:
published: true
sitemap: true
author: kim-dong-jun99
---

## kafka 설치를 위한 환경 구성하기

kafka를 설치하기 위해선 먼저 java와 zookeeper가 설치되어야합니다.

> 도커 환경으로 kakfa 설치 및 설정을 진행할 것이기에 자바 설치는 생략하겠습니다.

### zookeeper

Kafka는 zookeeper를 이용해서 kafka 클러스터의 메타데이터를 저장합니다.
<img width="753" alt="Screenshot 2024-11-07 at 11 33 52" src="https://github.com/user-attachments/assets/34284413-d942-4aa5-8eaf-3bce1eeb9e12">

ZooKeeper는 설정 정보 유지, 네이밍, 분산 동기화 제공, 그룹 서비스 제공을 위한 중앙 집중식 서비스입니다.
Zookeeper에 대해서는 깊게 다루지는 않겠지만, kafka를 운영하기 위해서 알아야할 수준으로만 알아볼 예정입니다.

**docker로 zookeeper standalone server 실행하기**

우선 실행할 zookeeper의 설정 파일을 생성합니다.
```txt
// zoo.cgf
tickTime=2000
clientPort=2181
dataDir=/data
dataLogDir=/datalog
```

`docker run -p 2181:2181 --name zookeeper --restart always -d -v ./zookeeper/zoo.cfg:/conf/zoo.cfg  zookeeper` 명령어로 zookeeper 서버를 실행합니다.

zookeeper 서버를 실행하고 `telnet localhost 2181`명령어를 실행한 후 `srvr` 커맨드를 전송하면 zookeeper 서버의 실행 정보를 확인할 수 있습니다.
```console
❯ telnet localhost 2181
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
srvr
Zookeeper version: 3.9.3-c26634f34490bb0ea7a09cc51e05ede3b4e320ee, built on 2024-10-17 23:21 UTC
Latency min/avg/max: 0/0.0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x0
Mode: standalone
Node count: 5
Connection closed by foreign host.
```

**docker로 zookeeper ensemble 모드로 실행하기**

zookeeper는 ensemble이라고 불리는 클러스터로 동작하도록 설계됐습니다. 다음과 같은 docker-compose 파일을 작성해 zookeeper를 ensemble 모드로 실행할 수 있습니다.
```yml
version: '3.1'

services:
  zoo1:
    image: zookeeper
    restart: always
    hostname: zoo1
    ports:
      - 2181:2181
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181

  zoo2:
    image: zookeeper
    restart: always
    hostname: zoo2
    ports:
      - 2182:2181
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181

  zoo3:
    image: zookeeper
    restart: always
    hostname: zoo3
    ports:
      - 2183:2181
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
```

실행 결과 다음과 같은 zookeeper 서버들이 실행됩니다.
```console
❯ telnet localhost 2181
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
srvr
Zookeeper version: 3.9.3-c26634f34490bb0ea7a09cc51e05ede3b4e320ee, built on 2024-10-17 23:21 UTC
Latency min/avg/max: 0/0.0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x0
Mode: follower
Node count: 5
Connection closed by foreign host.

❯ telnet localhost 2182
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
srvr
Zookeeper version: 3.9.3-c26634f34490bb0ea7a09cc51e05ede3b4e320ee, built on 2024-10-17 23:21 UTC
Latency min/avg/max: 0/0.0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x0
Mode: follower
Node count: 5
Connection closed by foreign host.

❯ telnet localhost 2183
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
srvr
Zookeeper version: 3.9.3-c26634f34490bb0ea7a09cc51e05ede3b4e320ee, built on 2024-10-17 23:21 UTC
Latency min/avg/max: 0/0.0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x100000000
Mode: leader
Node count: 5
Proposal sizes last/min/max: -1/-1/-1
Connection closed by foreign host.
```

하나의 leader 노드와 2개의 follower 노드가 실행 중인 것을 확인할 수 있습니다.

## Kafka 설치하기

zookeeper를 실행한 후에, 다음 명령어로 docker kafka 인스턴스를 실행할 수 있습니다.<br/>
`docker run -d -p 9092:9092 --name broker apache/kafka:latest`<br/>

다음 명령어로 실행중인 브로커 컨테이너 내부 shell를 생성할 수 있습니다.<br/>
`docker exec --workdir /opt/kafka/bin/ -it broker sh`<br/>

브로커 컨테이너 shell에서 다음 명령어로 토픽을 생성한 후, console producer를 통해 메세지를 발행할 수 있습니다.<br/>
`./kafka-topics.sh --bootstrap-server localhost:9092 --create --topic test-topic`<br/>
`./kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test-topic`<br/>

메세지를 발행하고 또 다른 shell에서 consumer를 실행하면 발행한 메세지를 소비할 수 있습니다.<br/>
`./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic --from-beginning`

![Screen Recording 2024-11-07 at 14 02 41](https://github.com/user-attachments/assets/9ed80046-5cdc-42b2-b530-48a257d74ddc)

### broker 설정하기

`broker.id`<br/>
모든 kafka broker는 정수 식별자를 가집니다. 기본 값으로 0이라는 값을 가지지만, 어떠한 값이라도 될 수 있습니다.
하나의 kafka 클러스터에서 broker들의 식별자 값은 유니크해야 합니다.

`listeners`<br/>
kafka의 오래된 버전은 간단한 포트 설정을 가졌습니다. 
이 구성은 여전히 간단한 설정의 백업으로 사용할 수 있지만, 이제는 deprecated된 설정입니다.
예제 설정 파일은 TCP 포트 9092에서 리스너를 사용하여 kafka를 시작합니다.
새로운 `listeners` 설정은 콤마로 구분된 URI의 리스트로 리스너 이름과 함께 설정해야 합니다.
리스너 이름이 일반적인 보안 프로토콜이 아닌 경우, 추가로 `listeners.security.protocol.map` 설정도 구성해야 합니다.
리스너는 `<protocol>://<hostname>:<port>` 형식으로 정의됩니다. 유효한 리스너 구성의 예시로는 `PLAINTEXT://localhost:9092, SSL://:9091`이 있습니다.
호스트 이름을 `0.0.0.0`으로 저징하면 모든 인터페이스에 바인딩되며, 호스트 이름을 비워두면 기본 인터페이스에 바인딩됩니다.

`zookeeper.connect`<br/>
broker의 메타데이터를 저장하는 zookeeper의 위치에 관한 설정입니다.
기본적으로 `localhost:2181`로 설정되어있습니다.

`log.dirs`<br/>
kafka는 모든 메세지를 디스크에 저장하고 로그 세그멘트들은 `log.dirs`에 명시할 디렉터리에 저장됩니다.
다수의 디렉터리를 사용하는 경우, `log.dirs` 설정을 사용하는 것이 권장됩니다.
만약 값이 설정되지 않았다면, `log.dir` 설정에 명시한 설정을 따를 것입니다.
`log.dirs`는 로컬 시스템의 경로들을 콤마로 구분한 목록입니다. 여러 경로가 지정된 경우 브로커는 파티션을 "최소 사용" 방식으로 저장하며, 하나의 파티션의 로그 세그먼트는 동일한 경로 내에 저장됩니다.
> 참고로 broker는 새로운 파티션을 현재 가장 적은 수의 파티션이 저장된 경로에 배치하며, 사용된 디스크 공간의 양이 가장 적은 경로에 배치하는 것이 아닙니다.
> 따라서 여러 디렉터리에 데이터가 균등하게 분배되는 것은 보장되지 않습니다.

`nums.recovery.threads.per.data.dir`<br/>
kafka는 설정 가능한 스레드의 풀을 사용하여 로그 세그멘트들을 처리합니다. 현재 이 스레드 풀은 다음의 경우에 사용됩니다.
- 정상적으로 시작할 때 각 파티션의 로그 세그멘트를 열기 위해
- 장애 발생 후 시작할 때 각 파티션의 로그 세그멘트를 점검하고 잘라내기 위해
- 종료 시 로그 세그멘트를 깨끗하게 닫기 위해

기본적으로 하나의 디렉터리 당 하나의 스레드가 사용됩니다. 
이러한 스레드들은 시작 및 종료 시에만 사용되므로, 작업을 병렬화하기 위해 더 많은 수의 스레드를 설정하는 것이 합리적일 수 있습니다.
특히 올바르지 않은 shtdown으로 부터 복구하는 상황에서 많은 파티션을 가진 broker를 복구하는데에 처리 시간이 몇 시간이 걸릴 수도 있습니다.
이 파라미터를 설정할 때 `log.dirs`에 명시한 디렉터리 별로 설정된다는 것을 유의해야합니다.
만약 이 파라미터를 8로 설정하고, 로그 디렉터리는 3개가 있다면, 24개의 스레드를 사용할 것입니다.

`auto.create.topics.enable`<br/>
기본 kafka 구성에서는 다음과 같은 경우 브로커가 자동으로 토픽을 생성하도록 설정되어 있습니다.
- producer가 토픽에 메세지 쓰기를 시작할 때
- consumer가 토픽에서 메세지 읽기를 시작할 때
- client가 토픽에 대한 메타데이터를 요청할 때

많은 상황에서 kafka 프로토콜을 통해 토픽의 존재를 확인할 방법이 없어서 요청 시 토픽이 생성되는 경우, 이러한 동작이 바람직하지 않을 수 있습니다.
토픽 생성을 수동으로 또는 프로비저닝 시스템을 통해 명시적으로 관리하는 경우 `auto.create.topics.enable` 설정을 false로 설정할 수 있습니다.


`auto.leader.rebalance.enable`<br/>
kafka 클러스터가 모든 토픽 리더십이 한 브로커에 집중되어 뷸균형해지지 않도록 하기 위해, 이 구성을 통해 리더십을 최대한 균형있게 유지할 수 있습니다.
이 설정은 백그라운드 스레드를 활성화하여 정기적(`leader.imbalance.check.interval.seconds` 설정으로 구성 가능)으로 파티션의 분포를 점검하고, 
리더십 불균형이 `leader.imbalance.per.broker.percentage` 설정을 초과하면, 리밸런스가 시작됩니다.

`delete.topic.enable`<br/>
데이터 보존 지침에 따라 클러스터에서 임의로 토픽이 삭제되지 않도록 잠그는 것이 필요할 수 있습니다. false로 설정하여 토픽 삭제를 비활성화할 수 있습니다.

### 토픽 설정하기

`num.partitions`<br/>
자동 토픽 생성이 활성화 되어있을 때, 새로운 토픽이 생성될 때, 파티션을 몇개 생성할지에 관한 파라미터입니다.
기본 설정 값은 1입니다.
이 설정을 할 때 파티션의 수는 오직 증가만 할 수 있다는 것을 유의해야 합니다.

`default.replication.factor`<br/>
토픽 자동 생성 옵션이 활성화 되어있을 때, 이 설정 값으로 새로 생성되는 토픽의 복제 계수를 설정할 수 있습니다. 복제 전략은 클러스터의 내구성이나 가용성 요구사항에 따라 달라질 수 있으며, 복제 계수를 `min.insync.replicas` 설정보다 최소 1이상으로 크게 설정하는 것이 강력히 권장됩니다.

`log.retention.ms`<br/>
kafka가 메세지를 얼마나 오랫동안 보존할지에 대한 가장 일반적인 구성은 시간 단위로 설정됩니다.
기본 값은 `log.retention.hours` 매개변수로 지정되며, 168시간(일주일)로 설정됩니다. `log.retention.minutes`, `log.retention.ms` 매개변수를 이용해 설정할 수도 있으며, 더 작은 단위 크기가 우선으로 적용됩니다.

`log.retention.bytes`<br/>
메세지를 만료시키는 또 다른 방법은 보존되는 메세지의 총 바이트 수를 기준으로 하는 것입니다. 이 값은 `log.retention.bytes` 매개변수를 사용하여 설정되며, 각 파티션마다 적용됩니다.
즉 만약 8개의 파티션을 가진 토픽이 있고, `log.retention.bytes`가 1GB로 설정되면, 해당 토픽에 대해 보존되는 데이터를 최대 8GB가 됩니다.
-1로 설정하면 무한 보존이 바이트 수에 제한을 두지 않습니다.

`log.segment.bytes`<br/>
앞서 살펴본 로그 보존 설정은 개별 메세지가 아닌 로그 세그멘트에 대해 동작합니다.
메세지가 kafka broker에 생산되면, 현재 파티션의 로그 세그멘트에 추가됩니다.
`log.segment.bytes` 에 지정된 크기(기본값 1GB)에 도달하면 로그 세그멘트를 닫히고 새로운 세그멘트가 열립니다.
로그 세그멘트가 닫힌 이후에 해당 세그멘트에 대한 만료가 고려됩니다.
**로그 세그멘트 크기가 작을 수록 파일을 더 자주 닫고 할당해야 하므로 디스크 쓰기의 전체 효율성이 떨어집니다.**

로그 세그멘트 크기를 조정하는 것은 토픽에 메세지 생산 속도가 낮을 경우 중요할 수 있습니다.
하루에 100MB의 메세지만 받는 토픽이 있고 `log.segment.bytes`가 기본값으로 설정된 경우, 하나의 세그먼트가 채워지는데 10일이 걸립니다.
메세지는 로그 세그멘트가 닫힐 때까지 만료되지 않으므로, `log.retention.ms = 604800000`(일주일)로 설정된 경우, 실제로는 닫힌 로그 세그멘트가 만료될 때까지 17일간 메세지가 보존됩니다.

`log.roll.ms`<br/>
로그 세그멘트가 닫히는 시점을 제어하는 또 다른 방법은 `log.roll.ms` 매개변수를 사용하는 것입니다. 이 매개변수는 로그 세그멘트를 닫아야 하는 시간을 지정합니다.
kafka는 크기 제한에 도달하거나 시간 제한에 도달할 때, 먼저 도달하는 조건에 따라 로그 세그멘트를 닫습니다.
기본적으로 `log.roll.ms`에 대한 설정이 없기 때문에, 로그 세그멘트는 크기 기준으로만 닫히게 됩니다.

`min.insync.replicas`<br/>
데이터 내구성을 위해 클러스터를 구성할 때, `min.insync.replicas`를 2로 설장하면 적어도 2개의 복제본이 생산자와 동기화되고 최신 상태를 유지하도록 보장됩니다.
이는 생산자 설정에서 `acks`를 all로 설정하는 것과 함께 사용됩니다.
이렇게 하면 최소한 2개의 복제본(리더와 다른 하나)의 쓰기 작업을 성공적으로 확인해야만 쓰기가 성공으로 간주됩니다.
이는 리더가 쓰기를 확인한 후 실패하고 리더십이 쓰기가 성공하지 않은 복제본으로 전환되는 경우와 같은 시나리오에서 데이터 손실을 방지할 수 있습니다.
이러한 내구성 있는 설정이 없다면, 생산자는 성공적으로 생산되었다고 생각할 수 있으며, 메세지가 손실될 수 있습니다.

> 하지만 내구성을 높이기 위한 설정은 추가적인 오버헤드로 인해 비효율적일 수 있습니다. 따라서 고처리량 클러스터에서는 간헐적인 메세지 손실을 허용할 수 있다면, 이 설정을 기본 값인 1에서 변경하지 않는 것이 좋습니다.

`message.max.bytes`<br/>
kafka broker는 생산자가 보낼 수 있는 메세지의 최대 크기를 제한하며, 이는 `message.max.bytes` 매개변수로 구성됩니다.
기본 값은 `1,000,000`바이트 (1MB)로 설정되어 있습니다. 만약 생산자가 이 크기보다 큰 메세지를 보내려고 하면, 브로커는 오류를 반환하고 메세지는 수락되지 않습니다.
브로커에서 지정한 모든 바이트 크기는 압축된 메세지 크기에 대해 설정되므로, 생산자는 메세지가 압축되어 설정된 `message.max.bytes` 크기 이하로 압축될 경우, 이보다 훨씬 큰 크기의 메세지도 보낼 수 있습니다.


