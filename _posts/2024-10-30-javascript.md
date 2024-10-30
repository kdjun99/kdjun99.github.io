---
layout: post
title: javascript
date: 2024-10-30 14:34 +0900
description: 자바스크립트 개념 & 문법 정리
image: https://modulabs.co.kr/wp-content/uploads/2023/11/image-1536x864.jpeg
category: ["javascript"]
tags:
published: true
sitemap: true
author: kim-dong-jun99
---

## Intro

### definition

웹 페이지에 생동감을 불어넣기 위해 만들어진 프로그래밍 언어로, 자바 스크립트로 작성한 프로그램을 스크립트라고 부릅니다.
> 스크립트는 웹 페이지의 HTML 안에 작성할 수 있고, 웹 페이지를 불러올 때 스크립트가 자동으로 실행됩니다.
> 스크립트는 컴파일 없이 보통의 문자 형태로 작성 가능하고, 실행 가능합니다.

자바스크립트는 자바스크립트 엔진이 들어있는 모든 디바이스에서 동작합니다.
브라우저에는 **자바스크립트 가상 머신**이라 불리는 엔진이 내장되어 있고, 엔진의 종류는 다음과 같습니다.
- V8 : 크롬과 오페라에서 쓰임
- SpiderMonkey : 파이어폭스에서 쓰임

> 자바스크립트 엔진의 동작 방식은 꽤 복잡하지만, 기본 원리는 다음과 같습니다.
> 1. 엔진이 스크립트를 읽습니다.
> 2. 읽어 들인 스크립트를 기계어로 전환합니다.
> 3. 기계어로 전환된 코드가 실행됩니다.

> 자바스크립트는 브라우저 환경과 node.js 환경에서 사용 가능합니다. 다양한 환경에서 사용 가능하기에, 자바스크립트를 기반으로 한 여러 언어가 개발되었습니다. (typescript, dart 등) 각 언어마다 고유한 기능을 제공하고, 자바 스크립트에 충분한 개념 학습을 진행한 후, 이런 언어들을 알아보는 것이 좋습니다.

### basic

자바스크립트는 브라우저 환경과 서버 사이드 환경(node.js)에서 실행할 수 있습니다.
브라우저에서 자바스크립트를 싱행하는 방법은 다음과 같습니다.
```html
<!DOCTYPE HTML>
<html>
<body>
  <p>스크립트 전</p>
  <script>
    alert( 'Hello, world!' );
  </script>
  <p>스크립트 후</p>
</body>
</html>
```
브라우저는 `<script>` 태그를 만나면 안의 코드를 자동으로 실행합니다.

코드의 양이 많은 경우, 파일에 코드를 저장하고 파일 경로를 html에 삽입할 수 있습니다.
```html
<script src="/path/to/script.js"></script>
```

> html 안에 직접 스크립트를 작성하는 경우는 스크립트가 아주 간단할 때만 사용합니다. 
> 스크립트가 길어지면, 별개의 분리된 파일로 만들어 저장하는 것이 좋습니다.
> 스크립트를 별도의 파일에 작성하면 브라우저가 스크립트를 다운받아 캐시에 저장하기 때문에, 성능상의 이점이 있습니다.

### variables and constants

자바스크립트에선 `let`키워드를 사용해 변수를 생성합니다.
```javascript
let message;

message = 'hello';
```

한 줄에 여러 변수를 작성할 수도 있습니다.
```javascript
let user = 'john', message = 'hello';
```
> 오래전에 작성된 스크립트는 `var`키워드를 사용해 변수를 정의하기도 합니다. 동작 방식은 `let`과 거의 유사합니다.

상수는 `const` 키워드로 선언할 수 있습니다.
```javascript
const myBirthday = '18.04.1982';
```

### types

자바스크립트는 동적 타입 언어로 자료의 타입은 있지만, 변수에 저장되는 값의 타입은 언제든지 바꿀 수 있는 언어입니다.
```javascript
let message = "hello";
message = 123;
```

### 숫자형

숫자형은 정수 및 부동소수점 숫자를 나타냅니다. 그리고 자바스크립트에서 숫자 연산은 안전합니다.
```javascript
alert(1 / 0);
alert("숫자가 아님"/2);
```
위 연산들은 에러를 발생하지 않고 정상 실행되며, `Infinity`, `Nan`같은 값을 리턴하며 정상 종료됩니다.

### 문자형

자바스크립트에서는 문자열은 큰 따옴표, 작은 따옴표, 역 따옴표를 이용해 작성할 수 있습니다.
```javascript
let str = "Hello";
let str2 = 'Single quotes are ok too';
let phrase =`can embed another ${str}`
```

또한 자바스크립트에서는 글자형이라는 자료형은 따로 없고 문자형 자료형만 존재합니다.

### undefined & null
 
 다른 언어에서의 `null`은 존재하지 않는 객체에 대한 참조나 null 포인터를 의미하는데 자바스크립트에서는 존재하지 않는 값, 비어있는 값, 알 수 없는 값을 나타내는 데 사용합니다.

 `undefined` 값도 `null`값처럼 자신만의 자료형을 형성합니다. 변수는 선언했지만, 값을 할당하지 않았다면 해당 변수에 `undefined`가 자동으로 할당됩니다.

### 객체와 심볼

객체는 특수한 자료형입니다.
객체를 이용해 데이터 컬렉션이나 복잡합 entity를 표현할 수 있습니다.
심볼형은 객체의 고유한 식별자를 만들 때 사용됩니다.

`typeof` 연산자를 이용해 인수의 자료형을 확인할 수 있습니다.

```javascript
typeof undefined // "undefined"
typeof 0 // "number"
typeof 10n // "bigint"
typeof true // "boolean"
typeof "foo" // "string"
typeof Symbol("id") // "symbol"
typeof Math // "object" 
typeof null // "object" 
typeof alert // "function" 
```

### type conversion

### 문자열로 형변환

`String` 함수를 호출해 전달받은 값을 문자열로 변환할 수도 있습니다.

```javascript
let value = true;
alert(typeof value); // boolean
value = String(value);
alert(typeof value); // string
```

### 숫자형으로 변환

`Number` 함수를 호출해 주어진 값을 숫자형으로 명시해서 변환할 수 있습니다.
```javascript
let str = "123";
alert(typeof str); // string
let num = Number(str);
alert(typeof num); // number
```
### boolean형으로 변환

`Boolean` 함수로 변환가능하고, 0, 빈문자열, null, undefined, Nan 같은 값들은 false로 리턴되고 나머지 값들은 true로 리턴됩니다.

### 조건문

`if-else`조건문은 자바와 동일할 문법으로 사용가능합니다.
```javascript
let year = prompt('ECMAScript-2015 출시 년도는?', '');
if (year < 2015) {
  alert(' bigger ');
} else if (year > 2015) {
  alert(' smaller ');
} else {
  alert(' correct ');
}
```

### 반복문

반복문 또한 자바와 유사하게 사용 가능합니다.
```javascript
let i = 0;
while (i < 3) {
  alert(i);
  i++;
}

for (let i = 0; i < 3; i++) {
  alert(i);
}

```

### function

함수는 다음과 같이 선언할 수 있습니다.

```javascript
function showMessage() {
  alert("hello");
}
let userName = "kim"
function showMessage() {
  let msg = "hello"+userName; // 외부 변수에 접근 가능합니다.
  alert(msg);
}
```

다음과 같이 파라미터를 명시할 수도 있습니다.
```javascript
function showMessage(from, text="empty text") { // 파라미터 값이 지정되지 않았을 때, 기본 값 지정이 가능합니다.
  alert(from+" "+text);
}
```

다음과 같이 반환 값을 반환할 수도 있습니다.
```javascript
function sum(a, b) {
  return a+b;
}

let result = sum(1,2);
```

### 함수 표현식

자바스크립트는 함수를 특별한 종류의 값으로 취급합니다.
위에서 살펴본 함수 선언 방식 외에 함수 표현식을 사용해서 함수를 만들 수 있습니다.
```javascript
let sayHi = function() {
  alert("hello");
};

sayHi();
```
함수를 생성하고 변수에 할당하는 것처럼 함수가 변수에 할당되었습니다.

### 콜백 함수

함수를 값처럼 전달하는 예시를 좀 더 살펴보면, 다음과 같은 3가지 매개변수가 있는 함수를 작성할 수 있습니다.
```javascript
function ask(question, yes, no) {
  if (confirm(question)) {
    yes()
  } else {
    no()
  }
}

function showOk() {
  alert("ok");
}

function showCancel() {
  alert("cancel");
}

ask("agree?", showOk, showCancel);
```

함수 `ask`의 함수로 전달되는 인수를 콜백 함수라고 합니다.

위 예시를 익명 함수를 이용해 보다 더 간단하게 작성 가능합니다.
```javascript
function ask(question, yes, no) {
  if (confirm(question)) {
    yes()
  } else {
    no()
  }
}

ask(
  "agree?", 
  function() { alert("ok"); },
  function() { alert("cancel"); }
);
```

> 함수 표현식 vs 함수 선언문 <br/>
> 함수 선언문으로 정의한 함수는 선언문 이전에도 함수를 호출할 수 있지만, 함수 표현식으로 정의한 함수는 이전에 호출 불가합니다.


### 화살표 함수

```javascript
let func = (arg1, arg2, ...argN) => expression
```

위 와 같이 화살표 함수를 작성할 수 있고, 위 함수는 아래 함수와 동일하게 동작합니다.
```javascript
let func = function (arg1, arg2, ...argN) {
  return expression;
};
```



