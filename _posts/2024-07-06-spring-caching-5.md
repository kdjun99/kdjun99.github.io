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

## 다양한 캐시 레벨

> 참고 자료 링크      
> [https://managewp.com/blog/types-of-web-cache](https://managewp.com/blog/types-of-web-cache)      
> [https://www.baeldung.com/spring-two-level-cache](https://www.baeldung.com/spring-two-level-cache)       
> [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)       
> [https://wp-rocket.me/wordpress-cache/different-types-of-caching/](https://wp-rocket.me/wordpress-cache/different-types-of-caching/)        

가장 크게 두가지 타입으로 캐시를 나눌 수 있습니다.
1. 서버 사이드 캐시
2. 클라이언트 사이드 캐시

두 캐시 모두 웹 사이트의 빠른 로딩을 위해 사용할 수 있습니다.

### 서버 사이드 캐시

서버 사이드 캐시는 웹 파일과 데이터를 서버에 일시적으로 저장하여 나중에 사용할 수 있게 함으로써, 서버의 부하와 대기 시간을 효과적으로 줄여줍니다.

작동 방식은 다음과 같습니다:
1. 사용자가 웹사이트를 방문하고 웹 페이지를 요청하면, 웹사이트는 서버에서 데이터를 가져와 해당 웹 페이지를 생성하고 사용자에게 표시합니다.
2. 응답이 사용자에게 반환된 후, 서버는 웹 페이지의 복사본을 생성하여 캐시로 저장합니다.

사용자가 다시 사이트를 방문한다면, 캐시된 응답을 받으므로 전보다 빠르게 웹 사이트를 받을 수 있습니다.

[https://wp-rocket.me/wp-content/uploads/2022/12/Illustration-of-how-server-side-caching-works.png](https://wp-rocket.me/wp-content/uploads/2022/12/Illustration-of-how-server-side-caching-works.png)
> 이미지 출처 [https://medium.com/@tmakhlay2/cache-me-outside-1b312e04ead6](https://medium.com/@tmakhlay2/cache-me-outside-1b312e04ead6)

### 클라이언트 사이드 캐시

클라이언트 사이드 캐싱, 흔히 브라우저 캐싱이라고 불리는 이 방법은 웹 페이지의 복사본을 브라우저의 메모리(브라우저가 생성한 폴더)에 일시적으로 저장합니다.

따라서 사용자가 클라이언트 사이드 캐싱이 활성화된 웹사이트를 다시 방문할 때, 서버에 데이터를 요청하지 않고 사용자 장치에 있는 브라우저 캐시 폴더에서 데이터를 가져옵니다.

[https://wp-rocket.me/wp-content/uploads/2022/12/Illustration-of-how-client-side-caching-works.png](https://wp-rocket.me/wp-content/uploads/2022/12/Illustration-of-how-client-side-caching-works.png)
> 이미지 출처 [https://medium.com/@tmakhlay2/cache-me-outside-1b312e04ead6](https://medium.com/@tmakhlay2/cache-me-outside-1b312e04ead6)

### 서버 사이드 vs 클라이언트 사이드

- 서버 사이드 캐시는 정적인 웹 사이트인 경우에 적합합니다.
    - 동적으로 변하는 사이트를 캐시해두면 원치 않은 사이트가 클라이언트에게 리턴될 수도 있습니다.
- 동적인 사이트인 경우 클라이언트 사이드 캐시를 활용하는 것이 사용자에게 적합한 데이터를 fetching하고 caching할 수 있습니다.

### 3가지 웹 cache

1. **page cache (site cache)**

정적인 웹 페이지 자체를 cache해두는 개념입니다. 페이지 자체를 캐시해두는 것이기에 정적인 페이지에 사용하는 것이 적합합니다.

2. **browser cache**

브라우저 캐시를 이용해 html, css, 이미지 파일 등을 캐시해둘 수 있습니다. 사이트를 재방문할 경우, 캐시해둔 파일들을 재요청하지 않고 브라우저 캐시에서 가져옵니다.

3. **server cache**

서버에서 특정 오브젝트들을 캐시해두는 개념입니다. 서버에서 관리되는 캐시이며 데이터베이스 쿼리 결과등을 캐시함으로써 데이터베이스 접근을 줄일 수 있습니다.

## 프로젝트 캐시 정책

### spring cache vs hibernate cache

우선 스프링 레벨 캐시와 hibernate 2차 캐시 중 hibernate 2차 캐시로 도입할 계획입니다.

스프링 레벨 캐시보다 hibernate 캐시를 사용했을 때, 엔티티 자체를 캐싱해두는 것으로 캐시와 디비 간 데이터 정합성 문제를 하이버네이트에서 관리해준다는 장점이 있어 hibernate 캐시 도입을 결정했습니다.

캐시 구현체로는 hazelcast를 사용할 예정입니다. 

hibernate 2차 캐시를 지원하는 cache 구현체에는 다양한 종류가 있습니다.
- redis
- ehcache
- hazelcast
- etc

이중에서 hazelcast는 imdg라는 기술을 사용해서 로컬 캐시를 사용하면서 다른 인스턴스에서 만드는 변경 사항들을 전파할 수 있습니다. remote cache를 사용하면 네트워크 비용이 발생하게 되므로 로컬 캐시로 사용가능하고 다른 인스턴스에서 만드는 변경 사항들을 전파할 수 있는 hazelcast가 적합한 구현체입니다.

> redis에서는 유료버전 redisson을 사용해야만 local cache를 사용 가능하고, ehcache는 다른 인스턴스를 생성해서 terracota 서버를 사용해야 가능합니다.

----

이번 포스팅을 작성하면서 다양한 웹 캐시 레벨에 대해서 알아보았고, 고민 끝에 프로젝트에 적용할 캐시 레벨과 구현체를 결정할 수 있었습니다. 다음 포스팅에서는 본격적으로 hazelcast를 도입하기 전에 간단하게 redis를 사용해서 hibernate 2차 캐시의 동작 그리고 2차 캐시를 최대한 활용할 수 있게 코드 리팩토링 작업에 대해서 작성해보겠습니다.
