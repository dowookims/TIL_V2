# 자바스크립트에서 변수는 어떻게 저장이 될까?

## 2019.12.16

자바스크립트는 객체가 생성되었을 때 자동으로 메모리를 할당하고 쓸모 없어졌을 때 자동으로 해제한다. 이는 가비지 컬렉션이 자동적으로 처리 해 주기에, 자바스크립트 개발자는 메모리 관리를 하지 않아도 된다는 인상을 줄 수도 있다. 그러나 이는 잘못된 것이다.

## 1. 메모리 생존 주기

대부분의 프로그래밍 언어는 다음과 같은 생존 주기를 가진다.

1. 필요할 때 할당
2. 사용한다(읽기, 쓰기)
3. 필요 없어지면 해제

2는 모든 언어에서 개발자들이 코딩을 할 때 작성한다. 그리고 대부분 고수준 언어에서는 1,3 이 자동적으로 처리되나, 저수준 언어의 경우 이를 개발자가 직접 할당 해 주어야 한다.

## 2. 자바스크립트에서 메모리 할당

### 1) 값 초기화

자바스크립트는 값을 선언 할 때, 자동으로 메모리를 할당한다.

```js
var n = 123; // 정수를 담기 위한 메모리 할당
var s = 'azerty'; // 문자열을 담기 위한 메모리 할당

var o = {
  a: 1,
  b: null
}; // 오브젝트와 그 오브젝트에 포함된 값들을 담기 위한 메모리 할당

// (오브젝트처럼) 배열과 배열에 담긴 값들을 위한 메모리 할당
var a = [1, null, 'abra'];

function f(a){
  return a + 2;
} // 함수를 위한 할당(함수는 호출 가능한 오브젝트이다)

// 함수식 또한 오브젝트를 담기위한 메모리를 할당한다. 
someElement.addEventListener('click', function(){
  someElement.style.backgroundColor = 'blue';
}, false);
```

### 2) 함수 호출을 통한 할당

함수 호출의 결과로 메모리 할당이 되기도 한다.

```js
var d = new Date(); // Date 개체를 위해 메모리를 할당

var e = document.createElement('div'); // DOM 엘리먼트를 위해 메모리를 할당한다.
```

또한, 메소드가 새로운 값이나 오브젝트를 할당하기도 한다.

```js
var s = 'azerty';
var s2 = s.substr(0, 3); // s2는 새로운 문자열
// 자바스크립트에서 문자열은 immutable 값이기 때문에,
// 메모리를 새로 할당하지 않고 단순히 [0, 3] 이라는 범위만 저장한다.

var a = ['ouais ouais', 'nan nan'];
var a2 = ['generation', 'nan nan'];
var a3 = a.concat(a2);
// a 와 a2 를 이어붙여, 4개의 원소를 가진 새로운 배열
```

### 3) 값의 사용

변수나 오브젝트 속성 값을 읽고 쓸때 값 사용이 일어난다. 또 함수 호출시 함수에 인수를 넘길때도 일어난다.

### 4) 할당된 메모리가 사용되지 않을 때 해제

저수준 언어에서는 메모리가 필요없어질 때를 개발자가 직접 결정하고 해제하는 방식을 사용한다.
자바스크립트와 같은 고수준 언어들은 가비지 콜렉션(GC)이라는 자동 메모리 관리 방법을 사용한다. 가비지 콜렉터의 목적은 메모리 할당을 추적하고 할당된 메모리 블록이 더 이상 필요하지 않게게 되었는지를 판단하여 회수하는 것이다.  여기서 판단하는 것은 비 결정적이기 때문에, 여기서 많은 문제들이 발생하게 된다.

## 3. Garbage Collector

"더 이상 필요하지 않은" 모든 메모리를 찾는건 비결정적 문제이고, 이를 가비지 컬렉터가 판단하는데에는 다양한 알고리즘이 존재 할 수 있다.

### 1) Reference

가비지 콜렉션 알고리즘의 핵심 개념은 참조이다. A라는 메모리를 통해 (명시적이든 암시적이든) B라는 메모리에 접근할 수 있다면 "B는 A에 참조된다" 라고 한다. 예를 들어 모든 자바스크립트 오브젝트는 prototype 을 암시적으로 참조하고 그 오브젝트의 속성을 명시적으로 참조한다.

### 2) Reference Counting

 이 알고리즘은 "더 이상 필요없는 오브젝트"를 "어떤 다른 오브젝트도 참조하지 않는 오브젝트"라고 정의한다. 이 오브젝트를 "가비지"라 부르며, 이를 참조하는 다른 오브젝트가 하나도 없는 경우, 수집이 가능하다.

 ```js
 var o = {
  a: {
    b:2
  }
}; // 2개의 오브젝트가 생성되었다. 하나의 오브젝트는 다른 오브젝트의 속성으로 참조된다.
// 나머지 하나는 'o' 변수에 할당되었다.
// 명백하게 가비지 콜렉션 수행될 메모리는 하나도 없다.


var o2 = o; // 'o2' 변수는 위의 오브젝트를 참조하는 두 번째 변수이다.
o = 1; // 이제 'o2' 변수가 위의 오브젝트를 참조하는 유일한 변수가 되었다.

var oa = o2.a; // 위의 오브젝트의 'a' 속성을 참조했다.
// 이제 'o2.a'는 두 개의 참조를 가진다. 'o2'가 속성으로 참조하고 'oa'라는 변수가 참조한다.

o2 = "yo"; // 이제 맨 처음 'o' 변수가 참조했던 오브젝트를 참조하는 오브젝트는 없다
// 그렇다고 o 오브젝트에 가비지 콜렉션이 수행될 수는 없다
// 오브젝트의 'a' 속성이 여전히 'oa' 변수에 의해 참조되므로 메모리를 해제할 수 없다.

oa = null; // 'oa' 변수에 다른 값을 할당했다.
//이제 맨 처음 'o' 변수가 참조했던 오브젝트를 참조하는 오브젝트가 없기에 가비지 콜렉션이 수행된다.
 ```

#### 한계 : 순환 참조

 두 객체가 서로를 참조하고 있으므로, Reference-Counting 알고리즘은 둘 다 가비지 컬렉션의 대상으로 표시하지 않는다. 이러한 순환 참조는 메모리 누수의 흔한 원인이다.

```js
 function f(){
  var o = {};
  var o2 = {};
  o.a = o2; // o는 o2를 참조한다.
  o2.a = o; // o2는 o를 참조한다.

  return "azerty";
f();
 ```

### 3) Mark-and-sweep 알고리즘

이 알고리즘은 "더 이상 필요없는 오브젝트"를 "닿을 수 없는 오브젝트"로 정의한다.

이 알고리즘은 *roots* 라는 오브젝트의 집합을 가지고 있다(자바스크립트에서는 전역 변수들을 의미한다). 주기적으로 가비지 콜렉터는 *roots*로 부터 시작하여 *roots*가 참조하는 오브젝트들, *roots*가 참조하는 오브젝트가 참조하는 오브젝트들을 닿을 수 있는 오브젝트라고 표시한다. 그리고 닿을 수 있는 오브젝트가 아닌 닿을 수 없는 오브젝트에 대해 가비지 콜렉션을 수행한다.

이 알고리즘은 위에서 설명한 Reference-Counting 알고리즘보다 효율적이다. 왜냐하면 "참조되지 않는 오브젝트"는 모두 "닿을 수 없는 오브젝트" 이지만 역은 성립하지 않기 때문이다. 위에서 반례인 순환 참조하는 오브젝트들을 설명했다.

2012년 기준으로 모든 최신 브라우저들은 가비지 콜렉션에서 표시하고-쓸기 알고리즘을 사용한다. 지난 몇 년간 연구된 자바스크립트 가비지 콜렉션 알고리즘의 개선들은 모두 이 알고리즘에 대한 것이다. 개선된 알고리즘도 여전히 "더 이상 필요없는 오브젝트"를 "닿을 수 없는 오브젝트"로 정의하고 있다.

#### 순환 참조는 문제가 되지 않는다

첫 번째 예제에서 함수가 리턴되고 나서 두 오브젝트는 닿을 수 없다. 따라서 가비지 콜렉션이 일어난다.

#### 한계: 수동 메모리 해제

***
참조

https://developer.mozilla.org/ko/docs/Web/JavaScript/Memory_Management