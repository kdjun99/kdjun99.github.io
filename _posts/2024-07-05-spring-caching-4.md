---
layout: post
title: Spring - Caching.4
date: 2024-07-05 19:26 +0900
description: 다양한 cache 읽기, 쓰기 전략 정리
image:
category: ["spring-cache", "project"]
tags: ["cache", "cache strategy"]
published: true
sitemap: true
author: kim-dong-jun99
---

## 캐시 읽기 전략

### look aside 패턴

![캐시 읽기 전략](https://learn.microsoft.com/en-us/azure/architecture/patterns/_images/cache-aside-diagram.png)
> 이미지 출처 https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside

데이터를 조회할 때 캐시에 데이터가 저장되어 있는지 확인하고 캐시에 데이터가 없으면 데이터베이스에서 조회하는 읽기 전략입니다. 

특징 
- 캐시와 데이터베이스가 분리되어 있기에 캐시 장애 발생시에도 서비스는 문제 없이 실행될 수 있습니다.
	- 캐시가 다운되더라도 데이터베이스에서 가져올 수 있기 때문에
- 캐시와 데이터베이스 간 정합성 문제가 발생할 수 있습니다.
- 초기 조회 시 무조건 데이터베이스에 접근해야 하므로 단건 호출 빈도가 높은 서비스에서는 적합하지 않습니다.
	- 반복적으로 동일 쿼리를 수행하는 서비스에 적합합니다.
	
### read through 패턴

![read through](https://codeahoy.com/img/read-through.png)
> 이미지 출처 https://codeahoy.com/2017/08/11/caching-strategies-and-how-to-choose-the-right-one/

캐시에서만 데이터를 읽어오는 전략입니다.

look aside와 비슷하지만, **데이터 동기화를 라이브러리 또는 캐시에 위임하는 방식입니다.**
데이터 조회를 전적으로 캐시에만 의존하므로 redis가 다운될 경우 서비스에 문제가 생길 수 있습니다. 대신 캐시와 데이터베이스 간 데이터 동기화가 항상 이루어져 데이터 정합성 문제에서 벗어날 수 있습니다.

- 캐시에 문제가 발생했을 경우 문제가 되기에 캐시의 replication, cluster 구성이 필요합니다.

## 캐시 쓰기 전략

### write back 패턴

![writeback](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fv5Y3R%2FbtrRBa5kwoz%2FWHclsCcEdxImBj0bLEfSc1%2Fimg.png)
> 이미지 출처 https://codeahoy.com/2017/08/11/caching-strategies-and-how-to-choose-the-right-one/

데이터를 저장할 때 데이터베이스에 바로 저장하지 않고, 캐시에 모아서 배치 작업을 통해 데이터베이스에 반영하는 패턴입니다.

캐시에 저장했다가 데이터베이스에 쓰기 때문에 데이터베이스 부하를 줄일 수 있다는 장점이 있고, 쓰기 작업이 빈번하게 발생하면서 조회를 하는데 많은 양의 리소스가 소모되는 서비스에 적합합니다.

데이터의 정합성을 보장한다는 장점이 있지만, 캐시에서 오류가 발생하면 데이터를 영구 소실하게 된다는 단점도 보유하고 있는 패턴입니다.

### write through 패턴

![writethrough](https://velog.velcdn.com/images/zenon8485/post/87269e11-2566-4742-8b0a-62be86923ec0/image.png)
> 이미지 출처 https://codeahoy.com/2017/08/11/caching-strategies-and-how-to-choose-the-right-one/

데이터베이스와 캐시에 동시에 데이터를 저장하는 전략입니다. 데이터를 저장할 때 먼저 캐시에 저장한 다음 바로 데이터베이스에 저장합니다. read through 패턴과 마찬가지로 데이터베이스 동기화 작업을 캐시에게 위임합니다.

데이터 일관성 유지에 장점이 있고 데이터 유실이 발생하면 안되는 상항에 적합합니다. 매 요청마다 캐시와 데이터베이스 모두 쓰기 작업을 해야하기에 성능 이슈가 발생할 가능성이 있습니다.

### write around 패턴

모든 데이터는 데이터베이스에 저장하고 캐시는 갱신하지 않습니다. cache miss가 발생하는 경우에만 데이터베이스와 캐시에도 데이터를 저장합니다.

따라서 **캐시와 데이터베이스 내의 데이터는 다를 수 있습니다.**

write through 패턴 보다 훨씬 빠르다는 장점이 있습니다. cache miss가 발생하기 전에 데이터베이스에 저장된 데이터가 수정된다면, cache와 데이터베이스 간의 데이터 불일치가 발생합니다. 데이터베이스에 저장된 데이터가 수정, 삭제 될때마다 cache 또한 삭제하거나 변경하고, 캐시의 만료 시간을 짧게 조정하는 식으로 대처할 수 있습니다.

## 캐시 읽기 + 쓰기 패턴 조합

- look aside + write around : 일반적으로 자주 쓰이는 조합
- read through + write around : 데이터 정합성 이슈에 대한 완벽한 안전 장치를 구성할 수 있습니다.
- read through + write through : 데이터를 쓸 때 항상 캐시에서 데이터베이스로 보내므로 데이터 정합성이 지켜집니다.

## 캐시 공유 방식 지침

캐시는 여러 인스턴스에서 공유됩니다. 따라서 한 애플리케이션이 캐시에 저장된 데이터가 수정될 때, 다른 애플리케이션이 만드는 업데이트에 의해 변경 내용이 덮어 쓰여지지 않도록 유의해야합니다.

데이터의 충돌을 방지하기 위해 다음과 같은 개발방식을 취해야 합니다.

- 캐시 데이터를 변경하기 직전에 데이터가 검색된 이후 변경되지 않았는지 확인하는 방법
	- 변경되지 않앗다면 즉시 업데이트, 변경되었다면 업데이트 여부를 애플리케이션 레벨에서 결정하도록 수정해야합니다
	- 업데이트가 드물고 충돌이 발생하지 않는 상황에 적용하기 좋습니다.
- 캐시 데이터를 업데이트 하기 전에 lock을 잡는 방식
	- 조회성 업무를 처리하는 서비스에 lock으로 인한 대기 현상이 발생합니다
	- 데이터의 사이즈가 작아 빠르게 업데이트가 가능한 업무와 업데이트가 자주 발생하는 상황에 적합합니다.
	
## 프로젝트에 적용할 캐시 패턴

관리자가 업데이트 작업을 빈번하게 하지 않고, 데이터 정합성이 매우 중요합니다
그렇기에 read through, write through 패턴을 사용해야할 것으로 보여집니다.

이번 글에서는 캐시 읽기, 쓰기 전략에 대해서 알아보았습니다. 다음 글에서는 적용 가능한 다양한 캐시 옵션에 대해서 정리하고, 프로젝트에 도입할 캐시 정책에 대해서 다뤄보겠습니다.

[참고 블로그 링크](https://inpa.tistory.com/entry/REDIS-📚-캐시Cache-설계-전략-지침-총정리#)
