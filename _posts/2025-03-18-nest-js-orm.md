---
layout: post
title: prsima
date: 2025-03-18 16:16 +0900
description: spring+jpa를 사용하다, nestjs + prisma를 사용하며 느낀 점 정리
image: https://media2.dev.to/dynamic/image/width=1000,height=420,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe8wccds3d6jqdrqssn2v.jpg
category: ["nestjs", "prisma"]
tags:
published: true
sitemap: true
author: kim-dong-jun99
---

# 들어가며

`spring + jpa`를 주로 사용하다 `nestjs + prisma`를 사용하면서 여러 장,단점을 느꼈습니다. 각 orm의 장점만 추린 라이브러리를 개발해보면 좋겠다는 생각이 들어, 본격적인 개발 이전에 느낀 장, 단점을 정리해보는 글을 작성해볼까 합니다.

# prisma의 단점

## 서비스 메소드에 너무 긴 prisma 로직이 작성된다.

prisma의 최대 단점으로는 서비스 메소드에 데이터베이스 쿼리 관련한 코드가 너무 많이 작성된다는 점을 꼽고 싶습니다.
```typescript
const objectList = await this.prisma.object.findMany({
      where: {
        컬럼1: 필드1,
        연관된 엔티티: {
          컬럼2: 조건1,
        },
      },
      include: {
        연관된 엔티티1: {
          include: {
            연관된 엔티티2: {
              include: {
                연관된 엔티티3: {
                  include: {
                    연관된 엔티티4: {
                      where: {
                        컬럼3: 조건2,
                      },
                    },
                  },
                },
              },
            },
            연관된 엔티티5: {
              where: {
                컬럼4: 조건3,
              },
            },
          },
        },
      },
    });
```

```java
SomeObject someObject = this.someObjectRepository.findBy컬럼2And컬럼3And컬럼4(조건1, 조건2, 조건3);
```

위와 같은 prisma를 이용한 조회 쿼리를 서비스 메소드에서 쉽게 발견할 수 있습니다. 아래 jpa 레포지토리를 이용하면, 한줄 혹은 훨씬 간단하게 같은 동작을 할 수 있습니다.
이와 같은 문제를 해결하기 위해서 prisma에 repository 패턴을 도입하여 사용하는 것이 한가지 방법이 될 수 있을 것 같긴하나, 개발자의 추가 공수가 필요하다는 점이 아쉬운 점으로 남게되는 것 같습니다.

## entity 클래스의 모호한 역할

prisma를 사용하면서, entity 클래스 역할의 모호함도 느끼게 되었습니다.

```java
    public void updateTransportationPriceSum(Integer oldVisitDateTransportationPriceSum, Integer newVisitDateTransportationPriceSum) {
        this.transportationPriceSum -= oldVisitDateTransportationPriceSum;
        this.transportationPriceSum += newVisitDateTransportationPriceSum;
    }

    public void updateTripItemPriceSum(Long oldTripItemPrice, Long newTripItemPrice) {
        this.tripItemPriceSum -= oldTripItemPrice;
        this.tripItemPriceSum += newTripItemPrice;
    }
```

이전 자바를 사용한 프로젝트에서 사용한 엔티티 코드의 일부입니다. 위와 같이, update를 하는 로직을 엔티티에 추출해 객체 지향적인 코드를 작성할 수 있었습니다. 
+ jpa의 변경 감지를 통해, 자동으로 데이터베이스 업데이트 쿼리가 실행되기에, 위 메소드를 실행하는 것으로 데이터베이스 업데이트까지 실행할 수 있었습니다.

prisma같은 경우, 
```typescript
await this.prisma.object.updateMany({
  data: {
    a,
    b,
    c,
  },
  where: {
    condition
  }
})
```

같이 쿼리를 작성해야되기에, 엔티티 클래스의 역할이 매우 모호하게 느껴졌습니다.
>  제가 prisma의 경험 부족으로 느낀 점일 수도 있습니다만, 맡은 프로젝트 코드에서, 엔티티에 어떤 코드가 작성된 경우를 찾기 어려웠습니다. prisma에서도 객체지향적으로 엔티티를 사용한 사례가 있으시면, 공유해주시면 감사하겠습니다.

## 트랜잭션을 사용하기 어렵다.

spring+jpa 에서는 `@Transactional` 어노테이션을 사용해서 정말 편리하고 깔끔하게 트랜잭션을 사용할 수 있습니다. spring에서는 중첩 트랜잭션도 지원하기에, `@Transactional` 어노테이션이 포함된 여러 서비스 메소드를 중첩해서 호출할때 기존 트랜잭션에 참여할지 혹은 새로운 트랜잭션을 생성할지 결정하기 수월합니다.

허나 prisma에서는 `prisma.$transaction` 메소드를 이용해서 트랜잭션을 사용해야되고, 한 트랜잭션 메소드 내부에서 다른 트랜잭션을 사용하는 서비스 메소드를 호출하면, 어떻게 트랜잭션이 처리될지 알기 어렵습니다.

> 서비스 메소드 파라미터로 tx를 받는 것으로 해결할 순 있으나, 좋아보이는 방법은 아닌 것 같습니다.

# prisma의 장점

## 어떤 쿼리가 실행될지 비교적 직관적으로 알 수 있다.

prisma는 동작을 개발자가 명시하기 때문에, jpa에 비해서 어떻게 코드가 동작할지 파악하기 쉽습니다. jpa는 내부적으로 실행되는 것들이 많기 때문에, jpa에 익숙하지 않은 개발자가 동작 방식을 파악하기 어려울 수도 있습니다.

> 음, 아직까지는 위 장점 말고는 파악하기 어려웠습니다. findUnique 배치처리, 중첩 create 등등 다양한 기능들을 지원하긴하는데 jpa에 비해서 특별한 장점으로 느껴지는 점들은 없었던 것 같습니다.    
> 그리고 jpa의 단점에 대해서는 devpill 님이 작성해주신 블로그 포스팅에 너무 잘 설명되어 있습니다.     
> [https://maily.so/devpill/posts/x1zg04jkrqg](https://maily.so/devpill/posts/x1zg04jkrqg)

# 어떻게 orm을 사용하는게 좋을까?

우선 repository를 사용하는 것이 좋을 것 같습니다. repository를 사용하면서, 데이터 접근 로직을 재활용할 수 있고, 서비스 메소드에 불필요하게 길게 작성된 데이터 접근 로직을 제거할 수 있습니다.
추가로 repository에 자주 사용되는 기본 메소드를 미리 정의되면 좋을 것 같습니다.

더불어서 스프링과 유사하게 Transactional을 사용할 수 있도록 지원해주는 라이브러리에 대한 개발 필요성을 느껴 앞으로 라이브러리를 개발하면서 공부한 점, 느낀 점들에 대한 포스팅을 작성해보겠습니다.
