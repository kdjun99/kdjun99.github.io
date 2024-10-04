--
layout: post
title: microservice pattern.02
date: 2024-10-04 10:14 +0900
description: microservice patterns 책을 통해 학습한 내용을 정리한 글입니다.
image:
category: ["msa"]
tags:
published: true
sitemap: true
author: kim-dong-jun99
---

## decomposition strategies

마이크로서비스 구조의 핵심 아이디어는 기능적 분리입니다. 하나의 큰 애플리케이션을 개발하는 대신, 애플리케이션을 여러 서비스의 집합으로 구성합니다.
애플리케이션을 이렇게 여러 서비스의 집합으로 구성하다보면, 몇몇 새로운 의문점들이 등장합니다.

이런 의문점들의 답을 찾기 위해서는 먼저 소프트웨어 아키텍처의 의미를 알아야합니다.

### 소프트웨어 아키텍처

소프트웨어 아키텍처는 다음과 같이 정의할 수 있습니다.
> 컴퓨팅 시스템의 소프트웨어 아키텍처는 시스템에 대해 논리적으로 이해하기 위해 필요한 구조들의 집합으로, 이 구조들은 소프트웨어 요소들, 그들 간의 관계, 그리고 이들 양쪽의 속성들로 구성된다.

다소 추상적인 정의인데, 애플리케이션 아키텍처의 본질은 여러 엘리멘트들로의 분해 그리고 그 엘리멘트들의 관계입니다.

좀 더 구체적으로 살펴보면, 애플리케이션 아키텍처는 다양한 관점에서 바라볼 수 있습니다. Phillip Krutchen은 [4 + 1 view model of software architecture](https://www.cs.ubc.ca/~gregor/teaching/papers/4+1view-architecture.pdf)라는 글을 작성했습니다.
4 + 1 모델에서는 소프트웨어 아키텍처의 4가지 다른 뷰를 정의합니다. 

<img width="758" alt="Screenshot 2024-10-04 at 15 06 49" src="https://github.com/user-attachments/assets/2f32a088-5305-47d9-8e3b-81c38f1e2343">


