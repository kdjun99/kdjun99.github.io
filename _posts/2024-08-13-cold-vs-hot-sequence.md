---
layout: post
title: Cold VS Hot Sequence
date: 2024-08-13 13:21 +0900
description: reactor에서 지원하는 cold sequence, hot sequence 알아보기
image:
category: ["reactive"]
tags:
published: true
sitemap: true
author: kim-dong-jun99
---

# Cold & Hot Sequence

cold는 차가운, 추운 이라는 뜻이고, hot은 뜨거운, 더운 이라는 뜻입니다. 이런 쉬운 단어가 컴퓨터 시스템에서도 종종 사용되는데, 일반적으로 알고 있는 단어의 의미를 컴퓨터 시스템에 그대로 적용하면 이해하기 힘든 측면이 있습니다.

간단하게 정의해보면, **cold는 무언가를 새로 시작하고, hot은 무언가를 새로 시작하지 않는다**로 정의할 수 있습니다.

## Cold Sequence

![https://velog.velcdn.com/images/shdrnrhd113/post/1fcf8abb-860e-42ec-952f-055c3b8e1b6e/image.png](https://velog.velcdn.com/images/shdrnrhd113/post/1fcf8abb-860e-42ec-952f-055c3b8e1b6e/image.png)

Cold Sequence는 Subscriber가 구독할 때마다 데이터 흐름이 처음부터 다시 시작되는 Sequence입니다.

Subscriber A, B의 구독 시점이 다르지만, 모두 1번 데이터부터 데이터 흐름이 시작되는 것을 확인할 수 있습니다.

```java
public class Example1 {
    public static void main(String[] args) {
        Flux<String> coldFlux = Flux.fromIterable(Arrays.asList("KOREA", "JAPAN", "CHINESE")).map(String::toLowerCase);
        coldFlux.subscribe(c -> log.info("sub 1 : {}", c));
        System.out.println("--------------------")
        Thread.sleep(2000L);
        coldFlux.subscribe(c -> log.info("sub 2 : {}", c));
    }
}
```

위 코드를 실행해보면, 두 Subscriber 모두 처음부터 emit된 데이터를 전달 받고 있음을 알 수 있습니다.

## Hot Sequence

![https://velog.velcdn.com/images/shdrnrhd113/post/bcf40459-c60f-4c47-be58-e6101dd66a2d/image.png](https://velog.velcdn.com/images/shdrnrhd113/post/bcf40459-c60f-4c47-be58-e6101dd66a2d/image.png)

Hot Sequence의 경우 구독이 발생한 시점 이전에 Publisher로부터 emit 된 데이터는 Subscriber가 전달받지 못하고 구독이 발생한 시점 이후에 emit 된 데이터만 전달 받을 수 있습니다.

```java
public class Example2 {
    public static void main(String[] args) {
        String[] singers = {"A", "B", "C", "D", "E"};

        Flux<String> concertFlux = Flux.fromArray(singers).delayElements(Duration.ofSeconds(1)).share();

        concertFlux.subscribe(s -> log.info("s1 : {}", s));

        Thread.sleep(2500);

        concertFlux.subscribe(s -> log.info("s2 : {}", s));

        Thread.sleep(3000);
    }
}
```

`delayElements()` 는 데이터 소스로 입력된 각 데이터의 emit을 일정시간 동안 지연시키는 Operator입니다. `share()` 는 coldSequence를 hotSequence로 동작하게 해주는 Operator입니다.

> Mono에서는 cache() 오퍼레이터를 사용해서 hot sequence로 전환할 수 있습니다.
