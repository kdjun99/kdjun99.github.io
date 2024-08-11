---
layout: post
title: Reactive Programming
date: 2024-08-11 14:01 +0900
description: 리액티브 프로그래밍이란
image:
category: ["reactive"]
tags:
published: true
sitemap: true
author: kim-dong-jun99
---

# Reactive Programming

리액티브 프로그래밍은 리액티브 시스템을 구축하는 데 필요한 프로그래밍 모델입니다. 리액티브 시스템에서는 **비동기 메시지 기반의 통신**을 통해서 구성 요소들 간의 느슨한 결합, 격리성, 투명성을 보장합니다.

> 비동기 메시지 통신은 Non-Blocking I/O 방식의 통신입니다. Blocking I/O 방식의 통신에서는 해당 스레드가 작업을 처리할 때까지 남아 있는 작업들은 해당 작업이 끝날 때까지 차단되어 대기합니다. Non-Blocking I/O 방식의 통신에서는 말 그대로 스레드가 차단되지 않습니다.

## Reactive Programming의 특징

위키피디아에서는 리액티브 프로그래밍을 다음과 같이 정의했습니다.

> In computing, reactive programming is a declarative programming paradigm concerned with data streams and the propagation of change.

### 선언형 프로그래밍

흔히들 사용하는 전통적인 프로그래밍 방식은 C 언어나 자바 같은 명령형 프로그래밍 방식입니다. 실행할 동작을 구체적으로 명시하는 프로그래밍 코드 형태라 할 수 있습니다.

선언형 프로그래밍 방식은 명령형 프로그래밍 방식과 달리 실행할 동작을 구체적으로 명시하지 않고 이러이러한 동작을 하겠다는 목표만 선언합니다.

명령형 프로그래밍 코드는 다음과 같습니다.

```java
public class Example1 {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 3, 21, 10, 8, 11);
        int sum = 0;
        for (int number : numbers) {
            if (number > 6 && (number % 2 != 0)) {
                sum += number;
            }
        }
        System.out.println(sum);
    }
}
```

같은 동작을 하는 코드는 선언형 프로그래밍으로는 다음과 같이 작성할 수 있습니다.

```java
public class Example2 {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 3, 21, 10, 8, 11);
        int sum = numbers.stream().filter(number -> number > 6 && (number % 2 != 0)).mapToInt(number -> number).sum();
        System.out.println(sum);
    }
}
```

코드를 보면 먼저 numbers 리스트 각 숫자에 접근하는 for문이 사라진 것을 확인할 수 있습니다. for문에서 하는 구체적인 동작을 java의 스트림이 내부에서 직접 해줍니다.
그리고 filter, sum 과 같은 메서드를 이용해 명령형 프로그래밍의 if문, 합연산을 처리하는 것을 알 수 있습니다.

선언형 프로그래밍 방식의 특징은 다음과 같습니다.
1. 동작을 구체적으로 명시하지 않고 목표만 선언한다
2. 여러가지 동작을 각각 별도의 코드로 분리하지 않고, 각 동작에 대해서 메서드 체인을 형성해서 한 문장으로 된 코드로 구성한다.
3. 함수형 프로그래밍으로 구성된다.

### data streams and propagation of change

- data streams
    - 데이터 흐름
    - 데이터가 지속적으로 발생한다는 의미
- propagation of change
    - 지속적으로 데이터가 발생 == 변화하는 이벤트
    - 이벤트를 발생시키면서 데이터를 계속적으로 전달하는 것을 의미

### Reactive Programming 코드 구성
작성하는 리액티브 프로그래밍 코드는 크게 publisher, subscriber, data source, operator 등으로 구성됩니다.

- publisher
    - 입력으로 들어오는 데이터를 제공하는 역할
- subscriber
    - publisher가 제공한 데이터를 전달받아서 사용하는 주체
- data source
    - publisher의 입력으로 들어오는 데이터를 대표하는 용어
- operator
    - publisher와 subscriber 사이에서 적절한 가공 처리를 담당

