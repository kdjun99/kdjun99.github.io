---
layout: post
title: Spring Caching.3
date: 2024-07-04 17:10 +0900
description: 도입한 caching 전략의 문제점 분석 & cache 읽기, 쓰기 전략 정리
image:
category: ["spring", "jpa", "hibernate"]
tags: ["cache"]
published: true
sitemap: true
author: kim-dong-jun99
---

## 도입한 caching 전략의 문제점

문제가 생길 경우는 다음과 같은 경우입니다.
1. 관리자가 상품 업데이트 API를 호출
2. 상품 변경을 위해 디비에서 조회하고 업데이트하는 서비스 로직 실행중 cache에 없는 데이터 조회 요청이 들어옴
3. 상품 변경을 마치고 데이터베이스에 값을 저장, 저장된 캐시 값 업데이트
4. 2번에서 들어온 조회 요청을 위해 데이터베이스에서 값을 조회, 이때 상품 업데이트 내용이 데이터베이스에 저장되지 않았다면, 예전 데이터 읽어옴
5. Cacheable 어노테이션으로 인해 예전 데이터로 cache 업데이트

## 문제 발생 원인

올바른 cache 쓰기 전략을 적용하지 않아서 발생하는 문제입니다.
캐싱 읽기 전략 패턴에는 look aside, read through 이 있고, 쓰기 전략 패턴에는 write back, write through, write around 패턴이 있습니다.

cacheable 어노테이션을 통해 look aside 전략을 이용해서 데이터를 읽었지만, 쓰기 작업을 위해 개발한 어노테이션이 올바른 패턴으로 동작하지 못했습니다.

개발한 어노테이션의 문제점을 해결하고, 올바른 캐시 패턴을 적용하기 위해 아래 블로그를 참고해서 cache 전략을 학습했습니다.

> [참고 블로그 링크](https://inpa.tistory.com/entry/REDIS-📚-캐시Cache-설계-전략-지침-총정리#write_through_패턴)

다음 포스팅에서 학습한 cache 패턴에 대해서 다뤄보겠습니다.

