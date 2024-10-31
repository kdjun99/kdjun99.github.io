---
layout: post
title: javascript-asynchronous
date: 2024-10-31 14:09 +0900
description: 자바 스크립트 비동기 프로그래밍
image:  https://modulabs.co.kr/wp-content/uploads/2023/11/image-1536x864.jpeg
category: ["javascript"]
tags:
published: true
sitemap: true
author: kim-dong-jun99
---

> [mdn web docs](https://developer.mozilla.org/en-US)를 참고해 학습한 내용을 정리한 포스팅입니다.

Node.js의 비동기 동작을 이해하기 위해서는 다음과 같은 개념들을 이해하고 있어야합니다.

- callbacks
- timers
- promises
- async and await
- closures
- event loop

## 자바스크립트 비동기 프로그래밍

비동기 프로그래밍은 오래 걸리는 태스크를 실행하고 대기하는 대신 태스크를 실행하는 중에 다른 이벤트에 반응 가능하도록 해주는 기술입니다.

브라우저에서 제공되는 다수의 함수들은 오랜 시간이 걸릴 수 있기에 비동기적으로 실행됩니다.
- `fetch()`를 이용해서 HTTP 요청을 만드는 경우
- `getUserMedia()`를 이용해 사용자의 카메라나 마이크에 접근하려는 경우
- `showOpenFilePicker()`를 이용해 유저로 하여금 파일을 선택하게 할때

비동기 함수를 구현할 일은 그리 많지 않지만, 비동기 함수를 종종 사용하게 될 것이기에 정확하게 아는 것이 중요합니다.

비동기 함수에 대해 살펴보기 전에, 동기 함수의 문제에 대해서 살펴보겠습니다.

### synchronous programming

다음과 같은 코드가 있습니다.
```javascript
const name = "Miriam";
const greeting = `Hello, my name is ${name}!`;
console.log(greeting);
// "Hello, my name is Miriam!"
```

1. `name`이라는 문자열을 선언합니다.
2. `greeting`이라는 문자열을 선언하고, 이 문자열은 `name`을 이용해 선언합니다.
3. 콘솔에 생성한 변수를 출력합니다.

브라우저는 작성한 코드 순서대로 순차적으로 코드를 실행합니다.
브라우저는 현재 실행 중인 코드 라인의 작업이 끝나야 다음 코드 라인으로 이동합니다. 각 라인이 이전에 작성된 코드에 의존하기에 이런 방식으로 동작합니다.

이렇게 동작하는 프로그램이 **synchronous program**입니다. 별개의 함수를 통해 작성해도 똑같이 동작합니다.
```javascript
function makeGreeting(name) {
  return `Hello, my name is ${name}!`;
}

const name = "Miriam";
const greeting = makeGreeting(name);
console.log(greeting);
// "Hello, my name is Miriam!"
```

`makeGreeting`은 함수를 호출하고 함수의 종료까지 대기하기에 **synchronous function** 입니다.

다음 프로그램은 많은 수의 소수를 만드는 비효율적인 알고리즘을 사용합니다.
`Generate primes` 버튼을 누르면 소수들이 생성되고 큰 숫자가 입력될 수록 실행 시간은 오래걸립니다.
```html
<label for="quota">Number of primes:</label>
<input type="text" id="quota" name="quota" value="1000000" />

<button id="generate">Generate primes</button>
<button id="reload">Reload</button>

<div id="output"></div>
<script src="./longSync.js"></script>
```

```javascript
// longSync.js
const MAX_PRIME = 1000000;

function isPrime(n) {
  for (let i = 2; i <= Math.sqrt(n); i++) {
    if (n % i === 0) {
      return false;
    }
  }
  return n > 1;
}

const random = (max) => Math.floor(Math.random() * max);

function generatePrimes(quota) {
  const primes = [];
  while (primes.length < quota) {
    const candidate = random(MAX_PRIME);
    if (isPrime(candidate)) {
      primes.push(candidate);
    }
  }
  return primes;
}

const quota = document.querySelector("#quota");
const output = document.querySelector("#output");

document.querySelector("#generate").addEventListener("click", () => {
  const primes = generatePrimes(quota.value);
  output.textContent = `Finished generating ${quota.value} primes!`;
});

document.querySelector("#reload").addEventListener("click", () => {
  document.location.reload();
});
```

버튼을 눌러 소수를 생성해보면, 몇 초후에 완료 메세지가 출력되는 것을 확인할 수 있습니다.

**오래 걸리는 동기 함수의 문제점**

위 html 파일에 사용자 입력을 받을 수 있는 텍스트 박스를 추가하고, 소수 생성 버튼이 눌렸을 때 곧 바로 입력을 시도해 보겠습니다.
```html
<label for="quota">Number of primes:</label>
<input type="text" id="quota" name="quota" value="1000000" />

<button id="generate">Generate primes</button>
<button id="reload">Reload</button>
<textarea id="user-input" rows="5" cols="62">
Try typing in here immediately after pressing "Generate primes"
</textarea>
<div id="output"></div>
<script src="./longSync.js"></script>
```

입력을 시도해보면, 소수가 생성되는 와중에는 프로그램이 응답하지 않는 것을 확인할 수 있습니다.

이렇게 프로그램이 응답하지 않게되는 원인은 자바스크립트 프로그램은 단일 스레드이기 때문입니다.
공식 문서에는 스레드를 다음과 같이 설명합니다.
> Thread is a sequence of instructions that a program follows

프로그램은 단일 스레드로 구성되기에, 한번에 하나의 작업만 수행할 수 있습니다.
그렇기에 오래 걸리는 작업의 완료를 기다리는 동안 다른 어떤 작업을 수행할 수 없습니다.

이런 동작을 피해 프로그램은 다음과 같이 동작해야합니다.
1. 오래 걸리는 작업을 함수를 통해 호출합니다.
2. 함수가 작업을 시작하고 곧바로 리턴하게합니다. 이렇게 함으로서 프로그램은 다른 이벤트에 대응할 수 있습니다.
3. 스레드를 block하지 않고 (새로운 스레드를 만들어서 사용한다거나 등) 작업을 이어나가게합니다.
4. 작업이 완료되면 결과를 반환합니다.

위와 같이 동작하는 것을 자바스크립트를 이용해 구현 가능합니다.

### event handler

이벤트 핸들러는 비동기 프로그래밍의 한 형식입니다. 이벤트 핸들러에 이벤트가 발생했을 때 호출될 함수를 명시합니다. 
만약 그 이벤트가 비동기 함수의 작업 완료를 의미한다면, 그 이벤트를 사용해 비동기 함수의 작업 종료를 호출자에게 알릴 수 있습니다.

몇몇 초기 비동기 API들은 이런 이벤트를 사용했습니다.
`XMLHttpRequest` API는 자바스크립트를 이용해 HTTP 요청을 원격 서버로 보내는 것을 가능하게 해줍니다.
원격 서버 호출은 오래 걸리는 작업이기에, 비동기 API이고, `XMLHttpRequest` 오브젝트 이벤트 핸들러를 사용해서 이벤트 완료 처리를 할 수 있습니다.

다음 코드는 `XMLHttpRequest`를 이용해서 `loadend`이벤트를 대기하는 예시입니다.
```html
<button id="xhr">Click to start request</button>
<button id="reload">Reload</button>

<pre readonly class="event-log"></pre>
<script src=./event.js></script>
```

```javascript
// event.js
const log = document.querySelector(".event-log");

document.querySelector("#xhr").addEventListener("click", () => {
  log.textContent = "";

  const xhr = new XMLHttpRequest();

  xhr.addEventListener("loadend", () => {
    log.textContent = `${log.textContent}Finished with status: ${xhr.status}`;
  });

  xhr.open(
    "GET",
    "https://raw.githubusercontent.com/mdn/content/main/files/en-us/_wikihistory.json",
  );
  xhr.send();
  log.textContent = `${log.textContent}Started XHR request\n`;
});

document.querySelector("#reload").addEventListener("click", () => {
  log.textContent = "";
  document.location.reload();
});
```

loadend 이벤트가 발생하면 완료 로그가 찍힙니다.

이벤트 리스너에 이벤트를 등록하고 API를 이용해 요청을 전송합니다. 요청을 전송한 후에, 요청 시작 로그 메세지를 남깁니다.

이전과는 다르게 화면에 요청 시작 로그가 먼저 남겨지는 것을 확인할 수 있습니다.

### callbacks

이벤트 핸들러는 콜백의 한 종류입니다. 공식 문서에서는 콜백을 다음과 같이 정의합니다.
> Callback is just a function that's passed into another function, with the expectation that the callback will be called at the appropriate time.

콜백 기반 코드는 콜백 함수가 중첩되는 경우 이해하기 어려워진다는 단점이 존재합니다.

아래 두 코드는 같은 동작을 하는 코드입니다.
```javascript
function doStep1(init) {
  return init + 1;
}

function doStep2(init) {
  return init + 2;
}

function doStep3(init) {
  return init + 3;
}

function doOperation() {
  let result = 0;
  result = doStep1(result);
  result = doStep2(result);
  result = doStep3(result);
  console.log(`result: ${result}`);
}

doOperation();
```

```javascript
function doStep1(init, callback) {
  const result = init + 1;
  callback(result);
}

function doStep2(init, callback) {
  const result = init + 2;
  callback(result);
}

function doStep3(init, callback) {
  const result = init + 3;
  callback(result);
}

function doOperation() {
  doStep1(0, (result1) => {
    doStep2(result1, (result2) => {
      doStep3(result2, (result3) => {
        console.log(`result: ${result3}`);
      });
    });
  });
}

doOperation();
```

콜백 방식으로 작성된 코드가 콜백이 중첩되었기에, 가독성이 무척이나 안 좋아진 것을 확인할 수 있습니다.

그렇기에, 현대 자바스크립트 API들은 `Promise`를 이용해 비동기 처리를 구현합니다.

## setTimeOut

setTimeOut 메서드를 이용해 일정 딜레이 후에, 특정 함수나 코드를 실행할 수 있습니다.

```javascript
setTimeout(code)
setTimeout(code, delay)

setTimeout(functionRef)
setTimeout(functionRef, delay)
setTimeout(functionRef, delay, param1)
setTimeout(functionRef, delay, param1, param2)
setTimeout(functionRef, delay, param1, param2, /* …, */ paramN)
```

delay 이후에 functionRef에 전달될 파라미터를 넘길 수 있습니다.

메서드는 `timeoutID` 값을 리턴하고, 이 값을 `clearTimeout()` 메서드로 전달해 타이머를 취소할 수도 있습니다.
> 타이머가 유효한 동안 이후에 호출되는 `setTimeout` 메서드로 동일한 아이디 값이 리턴되지 않는 것이 보장됩니다.

`setTimeout`은 비동기 함수입니다.
```javascript
setTimeout(() => {
  console.log("this is the first message");
}, 5000);
setTimeout(() => {
  console.log("this is the second message");
}, 3000);
setTimeout(() => {
  console.log("this is the third message");
}, 1000);

// Output:

// this is the third message
// this is the second message
// this is the first message
```

위 함수의 출력 결과를 확인하면, setTimeout을 호출한 순서대로 로그가 찍히지 않은 것을 알 수 있습니다.

어떤 함수가 종료된 이후 다른 함수가 호출되기를 원한다면, `Promise` 같은 것을 사용해야 합니다.

> setTimeout으로 메소드를 전달할 때, 해당 메소드 내부에 `this`를 사용하는 구문이 있으면 정상적으로 동작하지 않을 수 있습니다.

## promises

`Promise`는 비동기 작업의 결과적 성공 혹은 실패를 나타내는 오브젝트입니다.

기본적으로 promise는 리턴되는 오브젝트입니다.
아래 코드는 같은 동작를 하는 코드를 콜백 방식으로 그리고 promise를 이용해 작성한 코드입니다.

```javascript
function successCallback(result) {
  console.log(`Audio file ready at URL: ${result}`);
}

function failureCallback(error) {
  console.error(`Error generating audio file: ${error}`);
}

createAudioFileAsync(audioSettings, successCallback, failureCallback);
```

```javascript
createAudioFileAsync(audioSettings).then(successCallback, failureCallback);
```

여러 비동기 오퍼레이션의 순차적 실행을 콜백을 이용해 구현하려면 다음과 같이 복잡한 중첩 콜백을 구현해야합니다.
```javascript
doSomething(function (result) {
  doSomethingElse(result, function (newResult) {
    doThirdThing(newResult, function (finalResult) {
      console.log(`Got the final result: ${finalResult}`);
    }, failureCallback);
  }, failureCallback);
}, failureCallback);
```

promise는 체이닝을 이용해 순차적인 실행을 할 수 있습니다.
```javascript
const promise = doSomething();
const promise2 = promise.then(successCallback, failureCallback);
```
`then()` 함수는 기존과 다른 promise를 리턴합니다. 
`promise2`는 기존 doSomething()의 종료 뿐만 아니라 promise를 리턴할 수 있는 콜백 함수의 종료를 표현합니다.
이런 경우, `promise2`에 추가된 콜백은 `promise`의 콜백의 실행이 종되기 전까지 실행되지 않습니다.

이런 패턴을 사용해서 다음과 같은 보다 더 긴 체이닝을 작성할 수 있습니다.
> `catch(failureCheck)` 는 `then(null, failureCheck)`와 동일합니다.
```javascript
doSomething()
  .then(function (result) {
    return doSomethingElse(result);
  })
  .then(function (newResult) {
    return doThirdThing(newResult);
  })
  .then(function (finalResult) {
    console.log(`Got the final result: ${finalResult}`);
  })
  .catch(failureCallback);
```

다음과 같이 arrow function 형태로 작성할 수도 있습니다.
```javascript
doSomething()
  .then((result) => doSomethingElse(result))
  .then((newResult) => doThirdThing(newResult))
  .then((finalResult) => {
    console.log(`Got the final result: ${finalResult}`);
  })
  .catch(failureCallback);
```

`then`의 콜백은 undefined같은 값을 리턴하더라도 `promise`를 리턴하는 것이 중요합니다.
promise를 시작한 이전 핸들러가 아무런 값도 리턴하지 않는다면, floating promise가 발생합니다.

```javascript
doSomething()
  .then((url) => {
    // Missing `return` keyword in front of fetch(url).
    fetch(url);
  })
  .then((result) => {
    // result is undefined, because nothing is returned from the previous
    // handler. There's no way to know the return value of the fetch()
    // call anymore, or whether it succeeded at all.
  });
```

`fetch`의 결과를 리턴해야 이후 핸들러가 fetch의 결과를 처리할 수 있습니다.
```javascript
doSomething()
  .then((url) => {
    // `return` keyword added
    return fetch(url);
  })
  .then((result) => {
    // result is a Response object
  });
```

## async

`async` 키워드를 이용해 비동기 함수를 선언할 수 있습니다.
`await` 키워드를 이용해서 명시적으로 promise chain을 사용할 수 있습니다.
```javascript
function resolveAfter2Seconds() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve('resolved');
    }, 2000);
  });
}

async function asyncCall() {
  console.log('calling');
  // const result = await resolveAfter2Seconds();
  // console.log(result);
  // 위 주석 코드는 아래 코드와 동일하게 동작합니다.  
  resolveAfter2Seconds().then((result) => console.log(result));
}

asyncCall();
console.log("hello");
```

## event loop

자바 스크립트는 코드 실행의 책임을 가지고, 이벤트를 처리하고, 큐에 있는 서브 태스크들을 처리하는 **event loop** 런타임 모델에 기반합니다.
이 모델은 C나 자바에서 사용하는 모델과는 사뭇 다릅니다.

### 런타임 컨셉

![https://developer.mozilla.org/en-US/docs/Web/JavaScript/Event_loop/the_javascript_runtime_environment_example.svg](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Event_loop/the_javascript_runtime_environment_example.svg)

**스택**
함수를 호출하면, 프레임들이 스택에 쌓입니다.
```javascript
function foo(b) {
  const a = 10;
  return a + b + 11;
}

function bar(x) {
  const y = 3;
  return foo(x * y);
}

const baz = bar(7); // assigns 42 to baz
```

위 코드가 실행되면 다음과 같은 일이 벌어집니다.
1. `bar`를 호출할 때, `bar`의 argument와 지역 변수들의 레퍼런스를 포함하는 첫번째 프레임이 생성됩니다.
2. `bar`이 `foo`를 호출하면, 두번째 프레임이 생성되고, 스택에 push됩니다.
3. `foo`가 리턴하면, 스택 가장 위 프레임이 pop됩니다.
4. `bar`가 리턴하면, 스택은 비워집니다.

**힙**

오브젝트들은 힙에 할당됩니다. heap은 구조화되지 않은 큰 규모의 메모리를 의미합니다.

**큐**

자바 스크립트 런타임은 처리되야할 메세지 리스트인 메세지 큐를 사용합니다.
각 메세지는 메세지를 처리하기 위한 연관된 함수를 가집니다.

event loop 도중 특정 시점에 런타임은 큐에 있는 메세지를 오래된 순으로 처리합니다.
그러기 위해서 메세지는 큐에서 제거되고, 메세지와 대응되는 함수가 호출됩니다. 함수 호출은 해당 함수가 사용할 새로운 스택 프레임을 생성합니다.

스택이 비워질때까지 함수를 처리합니다. 그 후에 이벤트 큐에 메세지가 있다면 다음 메세지를 처리합니다.

**event loop**는 다음과 같은 방식으로 구현되었기에 이벤트 루프라고 불립니다.
```javascript
while (queue.waitForMessage()) {
  queue.processNextMessage();
}
```
`queue.waitForMessage()` 는 동기적으로 메세지를 대기합니다.

각 메세지는 완벽하게 처리된 후에 다른 메세지로 넘어갑니다.

이 방식은 프로그램을 이해할때 편리한 특성을 제공합니다. 함수가 실행될 때 중간에 방해받지 않으며, 함수가 완료될 때까지 다른 코드가 실행되지 않아 함수가 다루는 데이터를 안전하게 수정할 수 있습니다.
이는 C언어와는 다른 점인데, C에서는 함수가 스레드에서 실행될 경우, 런타임 시스템이 어느 순간에든 해당 스레드를 멈추고 다른 스레드에서 코드를 실행할 수 있기 때문입니다.

이런 모델의 단점은 메세지 처리가 너무 오래 걸릴 경우, 웹 애플리케이션이 클릭이나 스크롤 같은 사용자 상호작용을 처리할 수 없다는 점입니다.
브라우저는 "스크립트가 너무 오래 실행중입니다"라는 대화 상자를 통해 이를 완화합니다. 
좋은 대처 방법으로는 메세지 처리를 짧게 유지하고, 가능하다면 하나의 메세지를 여러 메세지로 나누는 것 입니다.

**메세지 추가하기**

웹 브라우저에서 이벤트가 발생하고, 해당 이벤트에 이벤트 리스너가 존재하는 경우 메세지는 추가됩니다.
리스너가 없다면 이벤트는 유실됩니다.
클릭 이벤트 핸들러가 있는 엘리멘트를 클릭하면, 메세지가 추가됩니다. 
그러나 `click` 메서드를 통한 시뮬레이션 클릭처럼 일부 이벤트는 메세지 없이 동기적으로 발생하기도 합니다.

`setTimeout()` 함수의 첫번째 두번째 인자는 큐에 추가할 메세지와 시간 값(선택 사항이며 기본값은 0)입니다. 시간 값은 메세지가 큐에 추가되기 까지의 최소 지연 시간을 나타냅니다.
만약 큐에 다른 메세지가 없고 스택이 비어 있다면, 지연 시간 후에 바로 메세지가 처리됩니다. 
그러나 큐에 다른 메세지가 있다면, `setTimeout()` 메세지는 다른 메세지들이 처리될 때까지 대기해야 합니다. 이 때문에 두번째 인자는 최소 시간을 의미하며, 보장된 시간이 아닙니다.

```javascript
const seconds = new Date().getTime() / 1000;

setTimeout(() => {
  // prints out "2", meaning that the callback is not called immediately after 500 milliseconds.
  console.log(`Ran after ${new Date().getTime() / 1000 - seconds} seconds`);
}, 500);

while (true) {
  if (new Date().getTime() / 1000 - seconds >= 2) {
    console.log("Good, looped for 2 seconds");
    break;
  }
}
```

**zero delays**

zero delay는 콜백이 곧 바로 호출될 것임을 의미하지는 않습니다.
`setTimeout()`을 아무런 딜레이 없이 호출해도 주어진 딜레이 이후 곧바로 실행하진 않습니다.

콜백의 실행은 큐에 존재하는 태스크에 따라 달라집니다.

```javascript
(() => {
  console.log("this is the start");

  setTimeout(() => {
    console.log("Callback 1: this is a msg from call back");
  }); // has a default time value of 0

  console.log("this is just a message");

  setTimeout(() => {
    console.log("Callback 2: this is a msg from call back");
  }, 0);

  console.log("this is the end");
})();

// "this is the start"
// "this is just a message"
// "this is the end"
// "Callback 1: this is a msg from call back"
// "Callback 2: this is a msg from call back"
```

`this is just a message`는 콜백 로그가 찍히기 전에 먼저 로그가 찍힙니다.
딜레이에 0을 전달하는 것은 함수 0초 딜레이에 실행된다는 의미가 아니라, 함수가 실행되는데, 최소 0초만큼의 시간이 필요하다는 의미입니다.

> 웹 워커나 교차 출처의 iframe은 각각 고유한 스택, 힙, 그리고 메시지 큐를 가지고 있습니다. 서로 다른 두 런타임은 postMessage 메서드를 통해 메시지를 보내는 방식으로만 통신할 수 있습니다. 이 메서드는 메시지 이벤트를 수신하는 경우, 다른 런타임의 메시지 큐에 메시지를 추가합니다.

**never blocking**

자바 스크립트의 event loop 모델의 굉장히 흥미로운 특징은 절대로 block되지 않는 다는 것입니다.
일반적으로 I/O 처리는 이벤트와 콜백을 통해 수행되므로, 애플리케이션이 IndexedDB 쿼리나 fetch() 요청의 반환을 기다릴 때에도 사용자 입력 같은 다른 작업을 계속 처리할 수 있습니다.

> 예외적으로 alert이나 동기적 XHR 요청과 같은 레거시 블로킹 동작이 존재하지만, 이러한 사용은 피하는 것이 좋은 실천 방법으로 여겨집니다. 주의할 점은, 예외의 예외(보통 구현 버그로 발생)가 존재할 수 있다는 것입니다.
