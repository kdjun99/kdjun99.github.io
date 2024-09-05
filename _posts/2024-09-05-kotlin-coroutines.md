---
layout: post
title: Kotlin - Coroutines
date: 2024-09-05 17:13 +0900
description: Coroutine 개념 정리 & 코틀린에서 Coroutine 사용하기
image:
category: ["kotlin", "coroutine"]
tags:
published: true
sitemap: true
author: kim-dong-jun99
---

> [공식 문서](https://kotlinlang.org/docs/coroutines-basics.html) 를 참고하여 학습하며 작성한 글입니다.

## Coroutine basics

코루틴은 중단 가능한 연산의 인스턴스입니다. 코드 블럭을 나머지 코드들과 동시에 실행 가능하다는 점에서 스레드와 컨셉적으로 유사합니다. 하지만, 코루틴은 특정 스레드에 묶여 있지 않습니다. 하나의 스레드에서 실행을 중단할 수 있으며, 다른 스레드에서 다시 실행을 재개할 수 있습니다.

코루틴은 가벼운 스레드처럼 생각할 수 있지만, 스레드와 아주 다른 점들이 존재합니다.

다음 코드를 통해 코루틴을 사용해볼 수 있습니다.
```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking { // coroutineScope
  launch { // 새로운 coroutine을 실행 
    delay(1000L) // 1초의 non-blocking delay
    println("world") // delay 이후 출력
  }
  println("hello") // main coroutine은 이전 coroutine이 지연되어도 계속 실행된다
}

/*
hello
world
*/
```

위 코드를 하나씩 뜯어보면 

- `launch` : 코루틴 빌더입니다. 나머지 코드와 동시에 실행되는 새로운 코루틴을 실행합니다. `launch`로 새로운 코루틴을 실행했기에, hello가 먼저 출력된 것을 확인할 수 있습니다.
- `delay` : 중단하는 함수입니다. 특정 시간 동안 코루틴을 중지시킵니다. 코루틴을 중단하면, 해당 코루틴이 사용 중인 기저 스레드가 차단되지 않고, 다른 코루틴들이 실행되면서 그 스레드를 사용할 수 있게 됩니다.
- `runBlocking` : 코루틴 빌더로 일반적인 `fun main()`과 같은 비코루틴 환경과 코루틴이 포함된 코드를 연결해주는 코루틴 빌더입니다. `runBlocking {...}` 중괄호 내부에 코루틴 코드를 작성할 수 있으며, 이는 IDE에서 `runBlocking`의 여는 중괄호 바로 뒤에 표시되는 `CoroutineScope` 힌트를 통해 강조됩니다.
  - 이 방식은 코루틴 코드가 실행될 때 `main`함수가 완료될 때까지 blocking 되는 특성을 가지고 있습니다. 이를 통해 비동기 코드와 동기적인 메인 함수 간의 상호작용이 가능합니다.

> 만약 `runBlocking`블럭을 작성하지 않았다면, `launch`을 작성할 때 에러가 발생하게 됩니다. `launch`는 `CoroutineScope`에서만 사용가능합니다.

`runBlocking`이라는 이름은 이 함수를 실행하는 스레드(이 경우 메인 스레드)가 호출되는 동안 차단된다는 의미입니다. `runBlocking {...}` 내부의 모든 코루틴이 실행을 완료할 때 까지 스레드는 차단됩니다. `runBlocking`은 주로 애플리케이션의 최상위 레벨에서 사용되며, 실제 코드 내부에서는 자주 사용되지 않습니다. 
> 스레드가 값비싼 리소스이기 때문에, 스레드를 차단하는 방식은 비효율적이며 대부분 원치 않기 때문입니다.

**구조적 동시성**

코루틴은 구조적 동시성 원칙을 따릅니다. 이는 새로운 코루틴이 특정 **Coroutine Scope** 내에서만 실행될 수 있음을 의미합니다. 이 스코프는 코루틴의 생명 주기를 한정합니다. 위 예제에서는 `runBlocking`이 해당 스코프를 설정하므로, 1초 후에 "World"가 출력될 때까지 기다린 후 프로그램이 종료됩니다.

실제 애플리케이션에서는 많은 코루틴을 실행하게 되며, 구조적 동시성은 이러한 코루틴들이 유실되거나 메모리 누수가 발생하지 않도록 보장해줍니다. **외부 스코프**는 그 안에 있는 모든 자식 코루틴들이 완료되기 전까지는 종료되지 않으며, 이 원칙은 코드에서 발생하는 오류가 제대로 보고되고 절대 유실되지 않도록 보장합니다.

**Extract function refactoring**

`launch{}`블럭 내부 코드를 함수로 추출해봅시다. 이 코드를 리팩토링하면서 "함수 추출"을 진행할 때, `suspend`키워드를 포함해 새로운 함수를 작성하게됩니다. 
이렇게 작성한 함수는 **suspending function** 입니다. suspending function은 코루틴 내부에서 보통 함수처럼 쓰일 수 있습니다. 한가지 차이점은 코루틴의 진행을 멈추는 (`delay`와 같은) 함수들을 실행할 수 있다는 것입니다.

```kotlin
fun main() = runBlocking {
  launch { doWorld() }
  println("Hello")
}

suspend fun doWorld() {
  delay(1000L)
  println("World!")
}
```

**Scope builder**

`coroutineScope`빌더를 사용해 코루틴 스코프를 지정할 수 있습니다. `coroutineScope` 빌더는 새로운 코루틴 스코프를 생성하고 모든 자녀의 실행이 완료될 때까지 종료되지 않습니다.

`runBlocking`과 `coroutineScope` 빌더는 유사하게 보입니다. 왜냐하면 둘다 자녀의 실행이 완료될때까지 대기하기 때문입니다. 가장 큰 차이점은 `runBlocking`은 현재 스레드를 `block`하고 `coroutineScope`는 suspend한다는 차이점입니다. 그렇기에 `runBlocking`은 일반 함수이고, `coroutineScope`는 suspending function입니다. 

**Scope builder and concurrency**

`coroutineScope` 빌더는 suspending function 내에서 여러 개의 **동시 작업**을 수행하기 위해 사용할 수 있습니다. 

```kotlin
// Sequentially executes doWorld followed by "Done"
fun main() = runBlocking {
    doWorld()
    println("Done")
}

// Concurrently executes both sections
suspend fun doWorld() = coroutineScope { // this: CoroutineScope
    launch {
        delay(2000L)
        println("World 2")
    }
    launch {
        delay(1000L)
        println("World 1")
    }
    println("Hello")
}
```

`launch { ... }` 블록 내의 두 코드 조각은 동시에 실행되며, `World 1`은 시작 후 1초 후에 먼저 출력되고, `World 2`는 2초 후에 출력됩니다. `doWorld` 함수 내의 `coroutineScope`는 두 코루틴이 모두 완료된 후에만 완료되므로, `doWorld` 함수는 두 코루틴이 끝난 후에 반환되며, 그제서야 `Done` 문자열이 출력됩니다.

**explicit job**

`launch` 코루틴 빌더는 `Job` 오브젝트를 리턴합니다. `Job` 오브젝트를 이용해 명시적으로 해당 작업이 끝날 때까지 대기할 수 있습니다.
```kotlin
val job = launch { // launch a new coroutine and keep a reference to its Job
    delay(1000L)
    println("World!")
}
println("Hello")
job.join() // wait until child coroutine completes
println("Done") 
```

**light-weight coroutine**

코루틴은 JVM 스레드보다 자원이 덜 필요합니다. JVM 여유 메모리를 고갈시키는 작업을 코루틴을 이용해서 처리했을 때는 문제없이 동작할 수 있습니다. 다음 코드는 50,000개의 코루틴을 실행하고 '.'을 출력하는 코드인데, 매우 적은 메모리를 이용해 처리할 수 있습니다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    repeat(50_000) { // launch a lot of coroutines
        launch {
            delay(5000L)
            print(".")
        }
    }
}
```

만약 위 코드를 스레드를 이용해서 처리하려면, 훨씬 더 많은 메모리를 사용하게 됩니다. 
