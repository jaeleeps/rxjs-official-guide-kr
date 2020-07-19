# Observable

Observables are lazy Push collections of multiple values. They fill the missing spot in the following table:

옵저버블은 다수의 값을 위한 lazy Push collection이다. 옵저버블은 하단 표에서 다음과 같이 표현될 수 있다.

| | Single | Multiple |
| --- | --- | --- |
| **Pull** | [`Function`](https://developer.mozilla.org/en-US/docs/Glossary/Function) | [`Iterator`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols) |
| **Push** | [`Promise`](https://developer.mozilla.org/en-US/docs/Mozilla/JavaScript_code_modules/Promise.jsm/Promise) | [`Observable`](/api/index/class/Observable) |



**예시.** 아래의 코드는 구독되는 즉시 (동기적으로) `1`, `2`, `3` 의 값을 밀어넣어주는 옵저저블입니다. 값 `4` 는 구독 요청 후 1초 뒤에 전달되며, 이후에 옵저버블은 완료됩니다.


```ts
import { Observable } from 'rxjs';

const observable = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});
```

이러한 옵저버블은 호출하여 값을 확인하기 위해서는 옵저저블은 *구독(subscribe)* 해야합니다:

```ts
import { Observable } from 'rxjs';

const observable = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});

console.log('just before subscribe');
observable.subscribe({
  next(x) { console.log('got value ' + x); },
  error(err) { console.error('something wrong occurred: ' + err); },
  complete() { console.log('done'); }
});
console.log('just after subscribe');
```

상단의 코드는 다음과 같은 결과를 콘솔에 출력합니다:

```none
just before subscribe
got value 1
got value 2
got value 3
just after subscribe
got value 4
done
```

## 풀(Pull)과 푸시(Push)

*풀-Pull*과 *푸시-Push*는 *데이터 제공자-Producer*가 *데이터 소비자-Consumer*와 소통하는 방식에 대한 일종의 규약(프로토콜)입니다.


**풀이란?** 풀 시스템에서, 데이터 소비자가 데이터 제공자로부터 데이터를 받을 시점을 결정합니다. 데이터 제공자는 언제 데이터가 소비자에게 전달되어야 하는지 알지 못합니다.

모든 자바스크립트 함수는 풀 시스템에 기반합니다. 함수는 데이터의 제공자이며, 해당 함수를 호출하는 코드는 호출로부터 *단일* 반환값을 "풀"함으로써 데이터를 소비하게됩니다.

ES2015는 풀 시스템의 또 다른 형태인 [generator functions and iterators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*) (`function*`) 을 소개합니다. `iterator.next()`를 호출하는 코드가 데이터의 소비자이며, 여기서 데이터 소비자는 이터레이터(iterator)로부터 다수의 값을 "풀"합니다.


| | Producer | Consumer |
| --- | --- | --- |
| **Pull** | **Passive:** 요청에 따라 데이터를 생성. | **Active:** 데이터 요청 시기를 결정. |
| **Push** | **Active:** 스스로 데이터 생성 시기를 경정. | **Passive:** 전달된 데이터에 따라 반응. |


**푸시란?** 푸시 시스템에서, 데이터 제공자가 데이터 소비자에게 데이터를 보낼 시기를 경정합니다. 소비자는 데이터가 어떠한 시점에 전달 될지 알 수 없습니다.

프로미스는 현재 자바스크립트 생태계에서 가장 널리 사용되는 푸시 시스템입니다. 프로미스(데이터 제공자)는 가공된 값을 등록된 콜백(소비자)들에게 전달하지만, 함수와는 다르게 생성된 값들이 어떠한 시점에 콜백들에게 "푸시"될 지 결정하는 권한은 프로미스에게 있습니다.

RxJS는 자바스크립트를 위한 새로운 푸시 시스템인 옵저버블을 제시합니다. 옵저버블은 다수의 값들을 옵저버(소비자)들에게 "푸시"하는 데이터 제공자입니다.

- **함수**는 호출에 따라 동기적으로 단일 값을 반환하는 게으른 연산입니다.
- **제너레이터**는 이터레이션에 따라 0개 부터 무한에 가까운 수의 값을 반환할 수 있는 게으른 연산입니다.
- **프로미스**는 단일을 값을 미래의 특정 시점에 반환하거나, 아예 반환하지 않을 수도 있는 연산입니다. 
- **옵저버블**은 호출되는 시점에 동기적으로, 혹은 미래의 특정 시점에 비동기적으로 0개 부터 무한에 가까운 수의 값을 반환할 수 있는 게으른 연산입니다.

## 함수의 일반화로써의 옵저버블

흔한 인식과는 달리, 옵저버블은 이벤트 에미터(EventEmitter)가 아니며 다수의 값을 반환하는 프로미스도 아닙니다. 옵저버블은 특정 환경에서 이벤트 에미터와 같이 *행동 할 수도 있지만*(RxJS Subject를 사용하여 멀티캐스팅 될 때), 보통의 경우에 이벤트 에미터와는 다른 행동을 취합니다.

<span class="informal">옵저버블은 인자가 없는 함수와 비슷하지만, 다수의 값을 위해 인자들을 일반화 하는 경향이 있습니다.</span>


하단의 코드를 보면:

```ts
function foo() {
  console.log('Hello');
  return 42;
}

const x = foo.call(); // foo()와 같은 동작을 한다.
console.log(x);
const y = foo.call(); // foo()와 같은 동작을 한다.
console.log(y);
```

우리는 다음과 같은 값을 반환 할 것이라고 생각합니다:

```none
"Hello"
42
"Hello"
42
```

상단의 코드와 똑같이 동작하는 코드를 옵저버블을 사용하여 작성할 수 있습니다.

```ts
import { Observable } from 'rxjs';

const foo = new Observable(subscriber => {
  console.log('Hello');
  subscriber.next(42);
});

foo.subscribe(x => {
  console.log(x);
});
foo.subscribe(y => {
  console.log(y);
});
```

예상한 것처럼, 결괏값은 동일합니다.:

```none
"Hello"
42
"Hello"
42
```

함수와 옵저버블 모두 게으른 연산이기에 발생하는 현상입니다. 함수를 호출하지 않으면 `console.log('Hello')`는 실행되지 않습니다. 옵저버블 또한 "호출(구독)" 없이는 `console.log('Hello')`를 실행할 수 없습니다. 한 가지 명확히 할 점은, "호출(calling)"과 "구독(subscribe)"가 서로 독립된 동작이라는 것입니다. 두 함수 호출은 각각 다른 독립된 영향을 미치고, 두 옵저버블의 구독은 이 것과는 별개의 영향을 미칩니다. 영향을 공유하고 구독자(subscriber)의 존재여부와는 관계없는 즉시 실행을 가지는 이벤트 에미터와는 달리, 옵저버블은 공유하는 실행영역이 없으며 즉각적이지도 앖습니다.

<span class="informal">옵저버블을 구독하는 것은 함수를 호출하는 것과 유사합니다.</span>

옵저버블이 비동기적이라는 주장이 있지만. 이러한 주장은 사실이 아닙니다. 하단의 코드처럼 함수를 로그로 감싼 경우를 생각해봅시다.

```js
console.log('before');
console.log(foo.call());
console.log('after');
```

다음과 같은 결괏값을 반환합니다.

```none
"before"
"Hello"
42
"after"
```

옵저버블에도 같이 적용해봅시다.

```js
console.log('before');
foo.subscribe(x => {
  console.log(x);
});
console.log('after');
```

결괏값은 다음과 같습니다.

```none
"before"
"Hello"
42
"after"
```

이를 통해서 `foo`에 대한 구독이 함수와 같이 완전하게 동기적임을 알 수 있습니다

<span class="informal">옵저버블은 결괏값을 동기적으로 혹은 비동기적으로 전달 할 수 있습니다.</span>

그렇다면 옵저버블과 함수의 차이는 무엇일까요? **옵저버블은 다수의 결괏값을 여러 시간에 걸쳐 "반환"할 수 있습니다**, 함수는 못 하는 것이지요. 일례로, 함수는 다음과 같이 사용될 수 없습니다.


```js
function foo() {
  console.log('Hello');
  return 42;
  return 100; // 실행되지 않는 코드입니다. dead code라고도 합니다.
}
```

함수는 오직 하나의 결괏값을 반환할 수 있습니다. 하지만 옵저저블은 다음과 같이 사용될 수 있습니다.

```ts
import { Observable } from 'rxjs';

const foo = new Observable(subscriber => {
  console.log('Hello');
  subscriber.next(42);
  subscriber.next(100); // 다른 결괏값을 반환합니다.
  subscriber.next(200); // 또 다른 결괏값을 반환합니다.
});

console.log('before');
foo.subscribe(x => {
  console.log(x);
});
console.log('after');
```

동기적으로 결괏값을 반환합니다:

```none
"before"
"Hello"
42
100
200
"after"
```

하지만 동시에 비동기적으로 결괏값을 반환 할 수도 있습니다.:

```ts
import { Observable } from 'rxjs';

const foo = new Observable(subscriber => {
  console.log('Hello');
  subscriber.next(42);
  subscriber.next(100);
  subscriber.next(200);
  setTimeout(() => {
    subscriber.next(300); // 비동기적으로 실행됩니다.
  }, 1000);
});

console.log('before');
foo.subscribe(x => {
  console.log(x);
});
console.log('after');
```

다음과 같은 값을 반환합니다.:

```none
"before"
"Hello"
42
100
200
"after"
300
```

결론:

- `func.call()`은 "*동기적으로 하나의 값을 요청*"하는 것을 의미합니다.
- `observable.subscribe()`은 "*동기적으로든 비동기적으로든 불특정한 개수의 값을 요청*"하는 것을 의미합니다.

## 옵저버블의 내부 구조

옵저버블은 `new Observable` 구문 혹은 생성자를 통하여 **생성-created**되고 옵저버에 의해 **구독-subscribed**되며, 옵저버에게 `next` / `error` / `complete` 알림을 전달하기 위해 **실행-execute** 됩니다. 그리고 이러한 실행은 **폐기-disposed** 될 수도 있습니다. 이러한 네가지 측면은 옵저버블 객체에 내장되어있습니다. 하지만 이러한 특성의 일부는 옵저버와 섭스크립션과 같은 다른 타입들과도 연관이 있습니다.


옵저버블에 대한 주요 고려 사항:
- 옵저버블을 **생성 - Creating**
- 옵저버블을 **구독 - Subscribing**
- 옵저버블을 **실행 - Executing**
- 옵저버블을 **Disposing**

### 옵저버블의 생성

`Observable` 생성자는 하나의 인자를 받습니다: `subscribe` 함수.

하단의 예시는 옵저버에게 매초마다 `'hi'`라는 스트링을 반출하는 옵저버블을 생성합니다


```ts
import { Observable } from 'rxjs';

const observable = new Observable(function subscribe(subscriber) {
  const id = setInterval(() => {
    subscriber.next('hi')
  }, 1000);
});
```

<span class="informal">옵저버블은 `new Observable`구문을 통해 생성될 수 있습니다. 대부분의 경우, 옵저버블은 `of`, `from`, `interval`와 같은 생성 함수를 통해 생성됩니다.</span>

위의 예시에서, `subscribe` 함수는 옵저버블을 표현하는 데에 있어서 가장 중요한 역할을 합니다. 이제 구독을 하는 것이 어떠한 의미를 가지는지 살펴보겠습니다. 

### 옵저버블의 구독

위 예시에서 옵저버블인 `observable` 은 다음과 같이 *구독-subscribed* 될 수 있습니다.:

```ts
observable.subscribe(x => console.log(x));
```

`new Observable(function subscribe(subscriber) {...})` 내부의 `observable.subscribe` 와 `subscribe`가 동일한 이름을 가지는 것이 우연은 아닙니다. 라이브러리 내부에서 다르게 동작하기는 하지만 행동의 측면에서 개념적으로 같다고 생각할 수 있습니다.

이러한 현상은 동일한 옵저버블을 구독하는 다수의 옵저버들의 `subscribe` 호출이 공유되지 않음을 보여줍니다. 옵저버를 사용하여 `observable.subscribe` 을 호출할 때,`new Observable(function subscribe(subscriber) {...})` 내부의 `subscribe`함수는 주어진 섭스크라이버를 위해 실행됩니다. 각각의 `observable.subscribe` 호출은 주어진 섭스크라이버를 위한 독립된 환경을 구성합니다.

<span class="informal">옵저버블의 구독은 데이터가 전달되어야 할 콜백을 제공하며, 함수의 호출처럼 동작합니다.</span>

이것은 `addEventListener` / `removeEventListener`와 같은 이벤트 핸들러 API와는 전적으로 다릅니다. `observable.subscribe`를 통해, 주어진 옵저버는 옵저버블의 리스너로 등록되는 것은 아닙니다. 심지어 옵저버블은 해당 옵저버블을 구독하는 옵저버들의 리스트 조차 관리하지 않습니다.

그렇기에 `subscribe` 호출은 단순히 "옵저버블 실행"과 해당 실행의 주체인 옵저버에게 데이터와 이벤트를 전달하기를 시작하기 위한 방법이라고 생각할 수 있습니다.


### 옵저버블의 실행

`new Observable(function subscribe(subscriber) {...})` 내부의 코드는 "옵저버블 실행"을 표현합니다. 그리고 이 실행은 해당 옵저버블을 구독하는 옵저버블 내에서만 일어나는 게으른 연산입니다. 동기적으로든, 비동기적으로든 해당 실행문은 여러 시간에 걸쳐 다수의 결괏값을 반환합니다.

옵저버블 실행문이 전달할 수 있는 결괏값에는 세가지 종류가 잇습니다.

- "Next" notification: Number, String, an Object 등과 같은 결과를 반환합니다.
- "Error" notification: JavaScript Error 혹은 exception을 전달합니다.
- "Complete" notification: 어떠한 결괏값도 반환하지 않습니다.

"Next"는 가장 중요한 동시에 가장 널리 사용되는 타입입니다. "Next"는 구족자에게 전달되는 실제 데이터를 표현합니다. "Error"와 "Complete"는 옵저버블 실행문 내에서 한 번 실행되거나 아예 실행되지 않습니다. 그리고 "Error"와 "Complete"가 실행문 내부에서 동시에 실행 될 수 없으며, 둘 중 한 종류만 1회 실행될 수 있습니다.

해당 제약은 소위 *Observable Grammar* 혹은 *Contract* 이라고 불릐우는 정규식으로 표현됩니다.

```none
next*(error|complete)?
```

<span class="informal">단일 옵저버블 실행문에서, 불특정한 수(0에서 무한)의 Next notification이 전달 될 수 있습니다. Error 혹은 Complete notification이 한번이라도 전달되면, 그 이후에는 어떠한 notification도 전달 될 수 없습니다.</span>

하단의 코드는 3개의 Next notification을 전달한 후 완료되는 옵저버블 실행문입니다.


```ts
import { Observable } from 'rxjs';

const observable = new Observable(function subscribe(subscriber) {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  subscriber.complete();
});
```

옵저버블은 옵저버블 컨트렉트를 엄격히 준수해야합니다. 따라서 하단의 코드는 Next notification `4`를 전달 할 수 없습니다.


```ts
import { Observable } from 'rxjs';

const observable = new Observable(function subscribe(subscriber) {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  subscriber.complete();
  subscriber.next(4); // 규약에 어긋나기에 해당 notification이 발생하지 않습니다.
});
```

예외 발생 시 Error notification을 전달할 수 있도록 `subscribe` 를 `try`/`catch`문으로 감싸는 것이 권장됩니다.

```ts
import { Observable } from 'rxjs';

const observable = new Observable(function subscribe(subscriber) {
  try {
    subscriber.next(1);
    subscriber.next(2);
    subscriber.next(3);
    subscriber.complete();
  } catch (err) {
    subscriber.error(err); // 에러 발생 시 해당 에러를 전달한다.
  }
});
```

### 옵저버블 실행문의 폐기

옵저버블 실행문은 무한할 수 있으며, 옵저버들은 옵저버블을 유한한 시간동안 필요로 하며 이후에는 해당 실행을 철회하기를 바랍니다. 그렇기에 옵저버블 실행을 취소할 수 있는 API가 필요합니다. 각각의 실행문은 단인 옵저버들에 국한되기 때문에, 옵저버가 데이터를 받는 것이 완료되면 연산 능력과 메모리 자원 사용의 낭비를 막기 위해 옵저버블 실행을 멈출 수 있는 방법이 있어야만 합니다.

`observable.subscribe`가 실행되면, 옵저버는 새롭게 생성된 옵저버블 실행문과 연결됩니다. 그리고 해당 호출은 `Subscription` 객체를 반환합니다.


```ts
const subscription = observable.subscribe(x => console.log(x));
```

구독(Subscription)은 진행중인 옵저버블 실행문을 표현합니다. 그리고 해당 실행문을 취소하도록 해주는 최소한의 API를 제공합니다. 자세한 정보는 [`Subscription` type here](./guide/subscription)를 참고하세요. `subscription.unsubscribe()`를 통해서 진행중인 실행문을 취소할 수 있습니다.

```ts
import { from } from 'rxjs';

const observable = from([10, 20, 30]);
const subscription = observable.subscribe(x => console.log(x));
// 이후:
subscription.unsubscribe();
```

<span class="informal">구독을 하게되면, 진행중인 실행을 표현하는 구독 객체를 받게됩니다. `unsubscribe`를 실행하는 것을 통해서 해당 실행문을 취소할 수 있습니다.</span>

옵저버블은 `create()`를 사용해서 옵저버블을 생성할 때, 실행 시 사용되어지는 자원들을 폐기하여 반환하는 방식에 대해 정의할 책임이 있습니다. 보통은 `function subscribe()` 내부에서 `unsubscribe` 함수를 반환하도록 합니다.

하단의 코드는 `setInterval`를 통해 생성된 인터벌 실행문을 취소시키는 코드입니다.:

```js
const observable = new Observable(function subscribe(subscriber) {
  // Keep track of the interval resource
  const intervalId = setInterval(() => {
    subscriber.next('hi');
  }, 1000);

  // Provide a way of canceling and disposing the interval resource
  return function unsubscribe() {
    clearInterval(intervalId);
  };
});
```

`observable.subscribe` 이 `new Observable(function subscribe() {...})`와 비슷하게 동작하는 것 처럼,  `subscribe`로부터 반환되는 `unsubscribe`는 개념적으로 `subscription.unsubscribe`와 비슷하게 동작합니다. 해당 개념에 대한 ReactiveX 타입들을 제외하고 생각하면, 조금 더 직관적인 JavaScript의 관점에서 생각할 수 있습니다.

```js
function subscribe(subscriber) {
  const intervalId = setInterval(() => {
    subscriber.next('hi');
  }, 1000);

  return function unsubscribe() {
    clearInterval(intervalId);
  };
}

const unsubscribe = subscribe({next: (x) => console.log(x)});

// 이후:
unsubscribe(); // 사용 자원을 폐기, 반환 합니다.
```
옵저버블, 옵저버, 그리고 구독과 같은 Rx 타입들을 사용하는 이유는 옵저버블 규약과 같은 안정성과 오퍼레이터에서의 결합성을 가지기 위해서입니다.