# 동시성 모델과 이벤트 루프(Concurrency Model and Event loop)

**2019.12.17**

자바스크립트에서는 코드의 실행, 이벤트 수집 및 처리, 대기중인 sub tasks의 실행을 담당하는 이벤트 루프를 기반으로하는 동시성 모델이 있습니다.

## runtime concept

![concurrencymodel](https://developer.mozilla.org/files/4617/default.svg)

*concurrency model image on* [Mozila](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)

### stack

함수 호출은 frame 스택을 형성합니다.

```js
function foo(b) {
  let a = 10
  return a + b + 11
}

function bar(x) {
  let y = 3
  return foo(x * y)
}

console.log(bar(7)) //returns 42
```

1. bar를 호출하면 bar의 인수와 로컬 변수가 포함 된 첫 번째 프레임이 생성됩니다.
2. bar가 foo를 호출하면 두 번째 프레임이 작성되고 foo의 arguments 및 로컬 변수가 포함 된 첫 번째 프레임 위에 푸시됩니다.
3. foo가 반환되면 상단 프레임 요소가 스택에서 pop이 됩니다. (bar의 콜 프레임 만 남음).
4. bar가 반환되면 스택이 비게 됩니다.

### heap

객체는 힙 (heap)에 할당됩니다.

### Queue

자바스크립트 런타임은 처리되어야 할 메세지들의 리스트인 message queue를 사용합니다. 각 메시지에는 메시지를 처리하기 위해 호출되는 관련 함수가 있습니다.

1. 이벤트 루프는 어느 시점에서 런타임은 가장 오래된 메시지부터 큐에서 메시지 처리를 시작합니다.
2. 처리된 메시지는 큐에서 제거되고 이에 대항하는 함수가 메시지를 입력 매개 변수로 하여 호출됩니다.
3. 함수를 호출하면 해당 함수의 사용을위한 새로운 스택 프레임이 생성됩니다.
4. 스택이 다 비워 질 때까지 함수 처리가 계속됩니다.
5. 다음 이벤트 루프는 큐에있는 다음 메시지 있을 경우 처리합니다.
