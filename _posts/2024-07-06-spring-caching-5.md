---
layout: post
title: Spring - Caching.5
date: 2024-07-06 16:10 +0900
description: 프로젝트에 적용할 캐시 정책 수립하기
image:
category: ["spring-cache", "project"]
tags: ["cache-level"]
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

![https://wp-rocket.me/wp-content/uploads/2022/12/Illustration-of-how-server-side-caching-works.png](https://wp-rocket.me/wp-content/uploads/2022/12/Illustration-of-how-server-side-caching-works.png)
> 이미지 출처 [https://medium.com/@tmakhlay2/cache-me-outside-1b312e04ead6](https://medium.com/@tmakhlay2/cache-me-outside-1b312e04ead6)

### 클라이언트 사이드 캐시

클라이언트 사이드 캐싱, 흔히 브라우저 캐싱이라고 불리는 이 방법은 웹 페이지의 복사본을 브라우저의 메모리(브라우저가 생성한 폴더)에 일시적으로 저장합니다.

따라서 사용자가 클라이언트 사이드 캐싱이 활성화된 웹사이트를 다시 방문할 때, 서버에 데이터를 요청하지 않고 사용자 장치에 있는 브라우저 캐시 폴더에서 데이터를 가져옵니다.

![https://wp-rocket.me/wp-content/uploads/2022/12/Illustration-of-how-client-side-caching-works.png](https://wp-rocket.me/wp-content/uploads/2022/12/Illustration-of-how-client-side-caching-works.png)
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

