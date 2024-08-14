---
layout: post
title: Reactive Programming 사전지식
date: 2024-08-12 15:45 +0900
description: 리액티브 프로그래밍 사전지식
image:
category: ["reactive"]
tags:
published: false
sitemap: true
author: kim-dong-jun99
---

# Reactive Programming 사전 지식

리액티브 프로그래밍을 잘 사용하기 위해서 기본적으로 알아야 되는 최소한의 사전 지식은 바로 함수형 프로그래밍 기법입니다. 자바에서는 자바 8 이전까지는 함수형 프로그래밍이라는 개념이 없었으나 자바 8에 람다 표현식이 도입됨으로써 자바에서도 함수형 프로그래밍 기법을 사용할 수 있게 되었습니다.

## 함수형 인터페이스

함수형 인터페이스 역시 인터페이스지만, 기존 인터페이스 와의 차이점은 함수형 인터페이스는 단 하나의 추상 메서드만 정의되어 있습니다. 

Comparator 인터페이스가 함수형 인터페이스의 대표적 예시입니다.

```java
public class Example4 {
    public static void main(String[] args) {
        List<CryptoCurrency> cryptoCurrencies = SampleData.cryptoCurrencies;

        Collections.sort(cryptoCurrencies, new Comparator<CryptoCurren y>() {
            @Override
            public int compare(CryptoCurrency cc1, CryptoCurrency cc2) {
                return cc1.getUnit().name().compareTo(cc2.getUnit().name());
            }
        });
    }
}
```

위 코드에서는 Comparator 인터페이스를 익명 구현 객체의 향태로 Collections.sort() 파라미터로 전달하는데, 사실 객체를 메서드의 파라미터로 전달하는 것은 자바에서 매우 흔하게 사용하는 방식입니다.

그리고 인터페이스 자체를 익명 구현 객체로 전달하는 방식은 코드를 지저분하게 합니다. 그래서 자바 8 버전부터 기존에 이미 사용하던 인터페이스의 익명 구현 객체를 전달하던 방식을 조금 더 함수형 프로르개밍 방식에 맞게 표현할 수 있는 방법을 추가했는데 그것이 바로 람다 표현식입니다.

## 람다 표현식

람다 표현식의 구조 예시는 다음과 같습니다.      
`(String a, String b) -> a.equals(b)`       

람다 표현식은 단 하나의 추상 메서드를 가지는 인터페이스, 즉 함수형 인터페이스를 구현한 클래스의 메서드 구현을 단순화한 표현식입니다.

따라서 함수형 인터페이스의 메서드를 람다 표현식으로 작성해서 다른 메서드의 파라미터로 전달할 수 있습니다. 람다 표현식은 가운데 화살표를 기준으로 좌측의 람다 파라미터와 우측의 람다 몸체로 구성됩니다. 람다 파라미터를 함수형 인터페이스에 정의된 추상 메서드의 파라미터를 의미합니다. 그리고 람다 몸체는 이 추상 메서드에서 구현되는 메서드 몸체를 의미합니다. 

```java
public class Example5 {
    public static void main(String[] args) {
        List<CryptoCurrency> cryptoCurrencies = SampleData.cryptoCurrencies;
        Collections.sort(crypttoCurrencies, (cc1, cc2) -> cc1.getUnit().name().compareTo(cc2.getUnit().name()));
    }
}
```

람다 표현식으로 코드가 깔끔해졌습니다. 한 가지 혼동하지 않아야 될 부분은 함수형 인터페이스의 추상 메서드를 람다 표현식으로 작성해서 메서드의 파라미터로 전달한다는 의미는 메서드 자체를 파라미터로 전달하는 것이 아니라 **함수형 인터페이스를 구현한 클래스의 인스턴스를 람다 표현식으로 작성해서 전달한다**는 것입니다.

> Exmaple4에서 Collections.sort()의 파마리터로 Comparator 함수형 인터페이스의 익명 구현 객체를 전달했으므로, 람다 표현식 역시 익명 구현 객체와 마찬가지로 함수형 인터페이스를 구현한 클래스의 객체입니다.

람다 표현식에서는 람다 캡쳐링을 통해 외부에서 정의한 변수를 사용할 수도 있습니다.

## 메서드 레퍼런스

메서드 레퍼런스를 표현한 예시는 다음과 같습니다.      
`(Car car) -> car.getCarName() == Car::getCarName`         
이처럼 메서드 레퍼런스 형태로 표현할 수 있는 유형은 4가지 정도입니다.

### ClassName::static method

메서드 레퍼런스로 표현할 수 있는 가장 흔한 유형은 클래스의 static 메서드입니다.

```java
public class Example6 {
    public static void main(String[] args) {
        List<CryptoCurrency> cryptoCurrencies = SampleData.cryptoCurrencies;
        cryptoCurrencies.stream()
            .map(cc -> cc.getName())
//            .map(name -> StringUtils.upperCase(name))
            .map(StringUtils::upperCase)
            .forEach(name -> System.out.println(name));
    }
}
```

`StringUtils::upperCase` 메서드가 실행되어 코인명이 대문자로 출력되는 것을 확인할 수 있습니다.

### ClassName::instance method

메서드 레퍼런스로 표현할 수 있는 두 번째 유형은 클래스에 정의된 인스턴스 메서드입니다.

```java
public class Example6 {
    public static void main(String[] args) {
        List<CryptoCurrency> cryptoCurrencies = SampleData.cryptoCurrencies;
        cryptoCurrencies.stream()
            .map(cc -> cc.getName())
//            .map(name -> name.toUpperCase())
            .map(String::toUpperCase)
            .forEach(name -> System.out.println(name));
    }
}
```

여기서는 String 클래스의 인스턴스 메서드인 toUpperCase를 사용했습니다.

### object::instance method

이런 메서드 레퍼런스 유형은 람다 표현식 외부에서 정의된 객체의 메서드를 호출할 때 사용합니다

```java
public class Example7 {
    public static void main(String[] args) {
        List<CryptoCurrency> cryptoCurrencies = SampleData.cryptoCurrencies;
        int btcPrice = cryptoCurrencies.stream()
            .filter(cc -> cc.getUnit() == CurrencyUnit.BTC)
            .findFirst()
            .get()
            .getPrice();

        int amount = 2;
        PaymentCalculator calculator = new PaymentCalculator();
        cryptoCurrencies.stream()
            .filter(cc -> cc.getUnit() == CurrencyUnit.BTC)
            .map(cc -> new ImmutablePair(cc.getPrice(), amount))
//            .map(pair -> calculator.getTotalPayment(pair))
            .map(calculator::getTotalPayment)
            .forEach(System.out::println);
    }
}
```

### ClassName::new

생성자도 메서드 레퍼런스로 사용 가능합니다.

```java
public class Example8 {
    public static void main(String[] args) {
        List<CryptoCurrency> cryptoCurrencies = SampleData.cryptoCurrencies;
        int amount = 2;
        Optional<PaymentCalculator> optional = cryptoCurrencies.stream()
            .filter(cc -> cc.getUnit() == CurrencyUnit.BTC)
            .map(cc -> new ImmutablePair(cc.getPrice(), amount))
//            .map(pair -> new PaymentCalculator(pair))
            .map(PaymentCalculator::new)
            .findFirst();
    }
}
```

## 함수 디스크립터

함수 디스크립터는 함수 서술자, 함수 설명자 정도로 이해할 수 있는데, 실제로는 일반화된 람다 표현식을 통해서 이 함수형 인터페이스가 어떤 파라미터를 가지고, 어떤 값을 리턴하는지 설명해 주는 역할을 합니다. 

|함수형 인터페이스|함수 디스크립터|
|-----------------|---------------|
|Predicate<T> | T -> boolean |
|Consumer<T> | T -> void |
|Function<T, R> | T -> R |
|Supplier<T> | () -> T |
|BiPredicate<L, R> | (L, R) -> boolean |
|BiConsumer<T, U> | (T, U) -> void |
|BiFunction<T, U, R> | (T, U) -> R |

