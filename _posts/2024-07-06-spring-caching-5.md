---
layout: post
title: Spring - Caching.5
date: 2024-07-06 16:10 +0900
description: 프로젝트에 적용할 캐시 정책 수립하기
image:
category: [“spring”, “cache”]
tags: [“hibernate cache", "spring cache", "browser cache"]
published: true
sitemap: true
author: kim-dong-jun99
---

## redis를 이용해 write through 패턴 구현을 위한 사전 작업

### redis gears & redis cluser 구성

redis를 이용해 write through 패턴을 구현하기 위해 공식 문서를 찾아보았습니다. 공식 문서에서 마침 write through 전략을 redis-gears를 이용해 적용하는 문서를 발견할 수 있었습니다.

[How to use redis for write through caching strategy](https://redis.io/learn/howtos/solutions/caching-architecture/write-through)

> **redis gears란 무엇일까요?**    
> 공식 문서에서는 redis gears를 다음과 같이 설명하고 있습니다.     
> RedisGears is a programmable serverless engine for transaction, batch, and event-driven data processing allowing users to write and run their own functions on data stored in Redis.       
> redis gear에 대한 추가적인 설명은 [홈페이지 공식 문서](https://redis.io/docs/latest/operate/oss_and_stack/stack-with-enterprise/gears-v1/) 에서 확인할 수 있었습니다.

공식 문서에서 적힌 설명에 따르면 redis gears는 레디스 모듈로 구현되어서 있고, redis enterprise와 클러스터 구성을 갖춘 이후에 설치할 수 있습니다.

