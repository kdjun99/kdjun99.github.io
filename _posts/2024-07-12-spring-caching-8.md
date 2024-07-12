---
layout: post
title: Spring - Caching.8
date: 2024-07-12 12:54 +0900
description: apache ignite에 대하여
image:
category: ["spring-cache", "project"]
tags: ["apache ignite"]
published: true
sitemap: true
author: kim-dong-jun99
---

> 공식 문서를 참고하여 학습한 내용을 정리한 글입니다.      
> [https://ignite.apache.org/docs/latest/](https://ignite.apache.org/docs/latest/)

## apache ignite

공식 문서에서는 apache ignite를 다음과 같이 소개하고 있습니다.
> Apache Ignite is a distributed database for high-performance computing with in-memory speed.

### clustering

![https://ignite.apache.org/docs/2.16.0/images/ignite_clustering.png](https://ignite.apache.org/docs/2.16.0/images/ignite_clustering.png)

노드는 서버 노드 혹은 클라이언트 노드로 동작합니다.

- 서버 노드
  - 클러스터에서 데이터를 캐싱하거나, 계산 작업을 실행하는 노드입니다.
- 클라이언트 노드
  - 다른 보통 노드들처럼 클러스터에 속해있으나 데이터를 저장하지 않습니다.
  - 클러스터에 데이터를 stream 하거나 유저 쿼리를 작업을 처리합니다.

클러스터를 구성하기 위해서는 각 노드들은 서로 다른 노드와 연결되어 있어야하며 그것을 보장하기 위해서 discovery mechanism이 설정되어야합니다.

apache ignite는 multicast, tcp/ip, 그리고 zookeeper를 이용한 discovery 매커니즘을 지원합니다.

> multicast란        
> multicast는 ip 통신 방식 중 하나로 유니캐스트와는 다르게 1 대 N 통신 기술입니다. multicast에 대한 자세한 내용은 다음 블로그들을 참고해주세요.        
> [https://softtone-someday.tistory.com/14](https://softtone-someday.tistory.com/14)        
> [https://white-polarbear.tistory.com/114](https://white-polarbear.tistory.com/114)        
> [https://ethan-world.tistory.com/entry/멀티캐스트-정리-1](https://ethan-world.tistory.com/entry/멀티캐스트-정리-1)

### data modeling

Ignite에서 데이터가 어떻게 저장되고 사용되는지를 이해하기 위해 클러스터에서 데이터의 물리적 조직과 데이터의 논리적 표현 간의 차이를 구분해야합니다.

- 물리적 레벨
  - 각 데이터 항목(캐시 항목 또는 테이블 행)은 이진 객체(binary object) 형태로 저장되며, 전체 데이터 집합은 파티션이라는 더 작은 집합으로 나뉩니다. 파티션은 모든 노드 간에 고르게 분배됩니다. 데이터가 파티션으로 나뉘고 파티션이 노드로 나뉘는 방식은 친화 함수(affinity function)에 의해 제어됩니다.
- 논리적 레벨
  - 데이터는 작업하기 쉽고 최종 사용자가 애플리케이션에서 사용하기 편리한 방식으로 표현되어야 합니다. Ignite는 데이터의 두 가지 논리적 표현을 제공합니다: 키-값 캐시와 SQL 테이블(스키마)입니다. 이 두 가지 표현 방식은 다르게 보일 수 있지만 실제로는 동일한 데이터 집합을 표현할 수 있습니다.

> Ignite에서 SQL 테이블과 키-값 캐시 개념은 동일한(내부) 데이터 구조의 두 가지 동등한 표현입니다. 키-값 API나 SQL 문 또는 두 가지 방법을 모두 사용하여 데이터를 접근할 수 있습니다.

**key-value cache VS sql table**

캐시는 키-값 API를 통해 접근할 수 있는 키-값 쌍의 모음입니다. Ignite에서의 SQL 테이블은 전통적인 관계형 데이터베이스 관리 시스템(RDBMS)의 테이블 개념에 해당하며, 몇 가지 추가 제약이 있습니다. 예를 들어, 각 SQL 테이블은 기본 키(primary key)를 가져야 합니다.

기본 키가 있는 테이블은 기본 키 열이 키로, 나머지 테이블 열이 객체(값)의 필드를 나타내는 키-값 캐시로 표현될 수 있습니다.

![https://ignite.apache.org/docs/2.16.0/images/cache_table.png](https://ignite.apache.org/docs/2.16.0/images/cache_table.png)

이 두 표현 방식의 차이점은 데이터를 접근하는 방식에 있습니다. 키-값 캐시는 지원되는 프로그래밍 언어를 통해 객체를 다룰 수 있게 해줍니다. SQL 테이블은 전통적인 SQL 구문을 지원하며, 예를 들어 기존 데이터베이스에서 마이그레이션하는 데 도움을 줄 수 있습니다. 두 가지 접근 방식을 결합하여 사용 사례에 따라 각각 또는 둘 다 사용할 수 있습니다.

**binary object format**

ignite는 데이터 엔트리들을 *binary object*라는 형식으로 저장합니다. 이 형식은 다음과 같은 장점을 가지고 있습니다.

- 직렬화된 객체에서 전체 객체를 역직렬화하지 않고 임의의 필드를 읽을 수 있습니다. 이는 서버 노드의 클래스패스에 키와 값 클래스가 배포될 필요성을 완전히 제거합니다.
- 같은 유형의 객체에서 필드를 추가하거나 제거할 수 있습니다. 서버 노드에 모델 클래스 정의가 없으므로, 이 기능은 객체의 구조를 동적으로 변경할 수 있으며, 서로 다른 버전의 클래스 정의를 가진 여러 클라이언트가 공존할 수 있게 합니다.
- 클래스 정의 없이 타입 이름을 기반으로 새로운 객체를 생성할 수 있어 동적 타입 생성을 허용합니다.
- Java, .NET, C++ 플랫폼 간의 원활한 상호 운용성을 제공합니다.

### data partitioning

![https://ignite.apache.org/docs/2.16.0/images/partitioning.png](https://ignite.apache.org/docs/2.16.0/images/partitioning.png)

데이터 파티셔닝은 대규모 데이터 집합을 더 작은 조각으로 세분화하고 이를 모든 서버 노드 간에 균형 있게 분배하는 방법입니다.

파티셔닝은 친화 함수(affinity function)에 의해 제어됩니다. 친화 함수는 키와 파티션 간의 매핑을 결정합니다. 각 파티션은 제한된 집합(기본값으로 0에서 1023)에서 번호로 식별됩니다. 파티션 집합은 현재 사용 가능한 서버 노드 간에 분배됩니다. 따라서 각 키는 특정 노드에 매핑되어 해당 노드에 저장됩니다. 클러스터의 노드 수가 변경되면 파티션은 새로운 노드 집합 간에 재분배됩니다. 이 과정을 리밸런싱(rebalancing)이라고 합니다.

친화 함수(affinity function)는 친화 키(affinity key)를 인수로 받습니다. 친화 키는 캐시에 저장된 객체의 어떤 필드(또는 SQL 테이블의 어떤 열)도 될 수 있습니다. 친화 키가 지정되지 않은 경우 기본 키가 사용됩니다(SQL 테이블의 경우 PRIMARY KEY 열이 기본 키입니다).

**partitioned/replicated mode**

cache 혹은 sql 테이블을 생성할 때, partition mode 혹은 replicated mode로 동작할지를 결정해야합니다. 두 모드는 서로 다른 use case scenario를 바탕으로 설계 되었고, 서로 다른 장점과 성능을 제공합니다.

![https://ignite.apache.org/docs/2.16.0/images/partitioned_cache.png](https://ignite.apache.org/docs/2.16.0/images/partitioned_cache.png)

- PATITIONED
  - 모든 파티션들은 모든 서버 노드에 균등하게 나눠집니다.
  - 가장 확장 가능한 분산 모드로, 모든 노드에서 사용할 수 있는 총 메모리에 맞는 만큼의 데이터를 저장할 수 있습니다.
  - 노드가 많을 수록 더 많은 데이터를 저장할 수 있습니다.
  - REPLICATED 보다 업데이트 부하가 적고 조회 부하가 큽니다.
  - 캐시할 데이터의 크기가 크고 업데이트가 자주 일어나는 환경에 적합합니다.

![https://ignite.apache.org/docs/2.16.0/images/replicated_cache.png](https://ignite.apache.org/docs/2.16.0/images/replicated_cache.png)

- REPLICATED
  - 모든 데이터는 모든 노드에 replicated 됩니다.
  - 업데이트 시 모든 노드에서 변경이 일어나야되기에 업데이트 부하가 큽니다.
  - 캐시 사이즈는 노드의 여유 메모리 크기로 제한됩니다.
  - 캐시할 데이터 사이즈가 작고 업데이트가 자주 일어나지 않는 환경에 적합합니다.

### thin clients

thin client란 소켓 커넥션을 통해 클러스터와 연결하는 ignite lightweight client를 의미합니다. thin client는 클러스터의 멤버로 join하지 않고 ignite 노드와 연결하고 해당 노드를 통해 작업을 처리합니다.

ignite는 아래 thin client 들을 지원합니다.

- java
- .NET/C#
- C++
- python
- node.js
- php

모든 클라이언트는 현재 노드가 사용 가능하지 않을 때 다른 가용한 노드와 연결하는 connection failover 매커니즘을 지원합니다. 이 매커니즘이 동작하기 위해서 connection failover 발생시 연결할 다른 노드 address를 설정해 둬야합니다.

**partition awareness**

클러스터에 데이터는 노드들에 분산되어 있습니다. partition awareness가 없으면 다음 그림 같이 동작해야합니다. 애플리케이션은 클러스터의 특정 노드와 연결되어야하고 연결된 노드를 통해서 클러스터에 쿼리를 해야합니다. 

![https://ignite.apache.org/docs/2.16.0/images/partitionawareness01.png](https://ignite.apache.org/docs/2.16.0/images/partitionawareness01.png)

partition awareness를 이용하면 다음 그림처럼 특정 노드를 통하지 않고 데이터가 저장된 특정 노드로 바로 쿼리를 실행할 수 있습니다.

![https://ignite.apache.org/docs/2.16.0/images/partitionawareness02.png](https://ignite.apache.org/docs/2.16.0/images/partitionawareness02.png)

