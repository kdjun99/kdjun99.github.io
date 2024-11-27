---
layout: post
title: spring cloud stream
date: 2024-11-07 19:17 +0900
description: spring cloud stream에 대하여
image: https://velog.velcdn.com/images/jinhxxxxkim/post/626d05c5-392d-42ba-9705-40f7350ddb0e/image.png
category: ["spring"]
tags:
published: false
sitemap: true
author: kim-dong-jun99
---

> 공식 문서를 통해 학습한 내용을 정리한 글입니다.

## Main concepts and abstractions

![https://docs.spring.io/spring-cloud-stream/reference/_images/SCSt-with-binder.png](https://docs.spring.io/spring-cloud-stream/reference/_images/SCSt-with-binder.png)

Spring Cloud Stream 애플리케이션은 미들웨어에 종속되지 않는 코어로 구성됩니다. <br/>
이 애플리케이션은 외부 브로커가 노출하는 대상과 코드 내 입력/출력 인수 사이에 바인딩을 설정함으로써 외부세계와 통신합니다. 바인딩을 설정하는 데 필요한 브로커 관련 세부 사항은 미들웨어별 바인더 구현이 처리합니다.

### binder abstraction

Spring Cloud Stream은 Kafka와 RabbitMQ에 대한 바인더 구현을 제공합니다.
또한 
