---
layout: post
title: Reactive Streams
date: 2024-08-11 15:15 +0900
description: 리액티브 스트림이란
image:
category: ["reactive"]
tags:
published: true
sitemap: true
author: kim-dong-jun99
---

# Reactive Streams

리액티브 라이브러리를 어떻게 구현할지 정의해 놓은 별도의 표준 사양을 리액티브 스트림즈라고 부릅니다. 

## 구성요소
리액티브 스트림즈를 통해 구현해야 되는 api 컴포넌트에는 publisher, subscriber, subscription, processor 가 있습니다.

| 컴포넌트 | 설명 |
|----------|------|
|publisher| 데이터를 생성하고 발행하는 역할 |
|subscriber| 구독한 publisher로부터 발행된 데이터를 전달받아서 처리하는 역할 |
|subscription| publisher에 요청할 데이터의 개수를 지정하고, 데이터의 구독을 취소하는 역할 |
|processor| publisher와 subscriber의 기능을 모두 가지고 있다 |

리액티브 스트림즈 컴포넌트의 동작 과정은 다음과 같습니다.

1. Subscriber는 전달받을 데이터를 구독합니다. (subscribe)
2. 다음으로 Publisher는 데이터를 발행할 준비가 되었음을 Subscriber에게 알립니다. (onSubscribe)
3. Publisher가 데이터를 통지할 준비가 되었다는 알림을 받은 Subscriber는 전달받기를 원하는 데이터의 개수를 Publisher에게 요청합니다. (Subscription.request)
4. 다음으로 Publisher는 Subscriber로부터 요청받은 만큼의 데이터를 통지합니다. (onNext)
5. 데이터 발행, 데이터 수신, 데이터 요청의 과정을 반복하다가 Publisher가 모든 데이터를 발행하게 되면 마지막으로 데이터 전송이 완료되었음을 Subscriber에게 알립니다. (onComplete) 만약 에러가 발생한다면 에러가 발생했음을 Subscriber에게 알립니다. (onError)

## Reactive Streams Component Code

### Publisher

```java
public interface Publisher<T> {
    public void subscribe(Subscriber<? super T> s);
}
```

subscribe 메서드는 파라미터로 전달받은 Subscriber를 등록하는 역할을 합니다.
> subscribe 메서드는 Subscriber가 실행해야하는 것이 아닌가??        
> 개념상으로는 Subscriber가 구독하는 것이 맞는데, Publisher가 subscribe 메서드로 Subscriber를 등록해야한다.

### Subscriber

```java
public interface Subscriber<T> {
    public void onSubscribe(Subscription s);
    public void onNext(T t);
    public void onError(Throwable t);
    public void onComplete();
}
```

- onSubscribe
    - 구독 시작 지점에 어떤 처리를 하는 역할
    - Subscription 객체를 통해 Publisher에게 요청할 데이터의 개수를 지정하거나, 구독을 해지하는 것을 의미
- onNext
    - Publisher가 통지한 데이터를 처리하는 역할
- onError
    - Publisher가 데이터 통지를 위한 처리 과정에서 에러가 발생했을 때, 에러를 처리하는 역할
- onComplete
    - 데이터 통지를 완료했음을 알릴 때

### Subscription

```java
public interface Subscription {
    public void request(long n); // 구독한 데이터의 개수를 요청
    public void cancel(); // 구독을 해지
}
```


