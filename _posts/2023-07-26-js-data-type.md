---
title:  "자바스크립트의 데이터 타입"
categories: 
  - Javascript
  - Books
tags:
  - 데이터 타입
  - 원시 타입
  - 참조 타입
  - 불변성
  - 스택
  - 힙
  - 가비지 컬렉션
  - 코어 자바스크립트
  - 리액트 불변성
toc: true
toc_sticky: true
toc_label: "자바스크립트의 데이터 타입"
share: true

---

`정재남` 님의 `코어 자바스크립트` 1장 데이터 타입을 읽고, 개인적으로 궁금했던 내용을 함께 정리한 글입니다.  

# 📍 자바스크립트의 데이터 타입

자바스크립트의 데이터 타입은 크게 원시 타입, 참조 타입으로 나뉜다.  

원시 타입에는 String, Number, Boolean, Null, Undefined, Symbol 등이 있다.  

참조 타입으로는 Object, Array, Function, Date 등이 있다.   

원시 타입과 참조 타입은 `데이터를 할당하는 과정`에서 차이가 있다.  

데이터를 할당한다는 것은 우리가 원하는 값을 메모리 공간에 저장하고, 해당 값을 변수에 할당하는 과정을 말한다.  

<br/>
<br/>
<br/>


# 💡 원시 타입 데이터의 할당

원시 타입 데이터는 `Stack (이하 스택)` 영역에 저장된다.  

스택 영역에는 `크기가 고정적인 값`이 저장되므로 불변성을 가진 원시 타입 데이터가 저장되기에 알맞다.  

> 불변성이란 한번 메모리 공간에 저장된 데이터의 `값` 이 변할 수 없는 성질을 의미한다.

불변성을 가지는 데이터는 값이 변하지 않아서 크기가 고정적이므로 정적인 데이터를 다루는 스택 영역에 저장되는 것이 유리하다.  

```js
var a = 23;
a = 3;
```

자바스크립트 엔진이 위 코드를 평가하고 실행하는 과정을 간략하게 따라가 보자! 👀

자바스크립트 엔진은 스택 메모리 영역에 23이라는 값이 있는지 검색하고, 없다면 23을 저장할 공간을 확보한다.    

해당 공간의 `주솟값`을 식별자 a와 연결한다.  

즉, 식별자 a는 23이라는 데이터가 저장된 스택 영역의 주솟값을 참조한다.  

<br/>
<br/>

## 🧩 데이터 재할당

식별자 a 에 3이라는 값을 재할당할때 23이라는 숫자가 3으로 변하는 것이 아니다. 

자바스크립트 엔진은 스택 메모리 영역에 3이라는 값이 있는지 확인하고, 없다면 3을 저장할 공간을 확보한다.  

이후에 식별자 a 의 주솟값을 3을 저장한 주솟값으로 변경한다.  

즉, 23이라는 값 자체가 3으로 변경된 것이 아니라 식별자 a 가 참조하는 `주솟값`만 달라졌다.  

따라서, 원시 타입 데이터는 **불변하다**.    

<br/>


## ⚡️ 데이터의 생명 주기

한번 메모리 공간에 할당된 원시 타입의 값은 값 자체가 불변하므로 고정적인 크기를 가진다.  

함수의 호출이 종료되면 스택 영역에 저장된 값도 제거된다.  

<br/>
<br/>
<br/>



# 💡 참조 타입 데이터의 할당

반면에 참조 타입 데이터는 `Heap (이하 힙)` 영역에 저장된다.  

그리고, 이렇게 힙 메모리에 저장된 값을 참조할 수 있는 `주솟값` 이 스택 메모리에 저장된다. 우리가 선언한 변수는 스택 메모리 영역의 주솟값을 참조한다.  

힙 메모리는 동적인 데이터를 저장한다. 이는 힙 메모리에 저장되는 데이터는 그 크기가 가변적이라는 의미이다.  

예를 들어, 참조 타입 데이터인 배열은 요소를 추가하거나 제거함으로써 배열의 크기가 변한다.  

<br/>
<br/>


## 🧩 데이터 재할당

```js

var obj = {
	name: 'hj',
	age: 1234
};

var copyObj = obj;

copyObj.age = 2;

console.log(obj === copyObj); // true
```

copyObj 는 obj 와 같은 주솟값을 참조한다.  

그리고, copyObj 객체의 age 프로퍼티 값을 수정할 때 자바스크립트는 obj 가 참조하는 힙 메모리에서 age 프로퍼티의 값만 수정한다.  

이때 스택 메모리에 저장된 obj 가 참조하고 있던 주솟값은 변하지 않았다.  

어떤 객체를 복사한 후에 복사한 객체에서 `프로퍼티 값`을 수정했을 때 원본 객체의 프로퍼티 값도 같이 변경되는 이유가 이 때문이다.  

참조 타입 데이터는 내부의 프로퍼티를 변경하는 경우 가변적이다.  

‼️ 아예 새로운 객체를 할당하면 복사된 원본 객체가 변하지 않는다. 서로 다른 주솟값을 바라보기 때문이다.  

```js
var obj1 = {
	name: 'hj',
	mbti: 'esfp'
};

var obj2 = obj1;

obj2 = {
	name: 'eden',
	mbti: 'intj'
}

console.log(obj === obj2); // false
```

따라서, 참조 타입 데이터를 불변성을 유지하며 사용하려면 새로운 객체로 반환하는 것이 중요하겠다.  

교재에서는 함수의 프로퍼티 내부까지 완전히 새로운 객체로 복제하는 `깊은 복사` 의 예제 코드를 아래와 같이 제공한다.  

```js
var copyObjDeep = function(target){
	var newObj = {};

	if(typeof target === 'object' && target !== null){
		for(var prop in target){
			newObj[prop] = copyObjDeep(target[prop]);
  }
 } else {
		newObj = target;
	}
		return newObj;
};
```

<br/>


## ⚡️ 데이터의 생명 주기

스택에 쌓인 원시 타입 데이터는 관련 함수 호출이 종료되면서 제거되지만, 참조 타입 데이터는 다른 곳에서 참조하고 있다면 힙 메모리에 계속해서 존재할 수 있다.  

참조 카운트가 0이 된 데이터는 가비지 컬렉터의 회수 대상이 된다.  

> 참조 카운트란 특정 객체가 얼마나 많은 객체나 변수에 의해 참조되고 있는지를 나타낸다. 즉, “동일한 메모리 주소”를 참조하는지가 기준이 된다.

만약 해당 메모리 주소를 참조하고 있는 객체가 하나도 없는 참조 카운트 0인 상태가 된다면 가비지 컬렉터의 회수 대상이 된다. 

참조 카운트에 따라 가비지를 회수하는 것은 가비지 컬렉션을 하는 알고리즘 중에 하나이고, 표시하고-쓸기 Mark-and-sweep 라는 알고리즘도 있다.  

참조 카운트 알고리즘은 아래와 같은 허점이 있다.  

```js
function foo() {
  var a = {};
  var b = {};

  a.bar = b;
  b.baz = a;

  return 'clear';
}
```

변수 a와 변수 b는 함수 foo 가 종료되면 불필요하므로 회수되어야 한다.

하지만, 두 객체가 서로 참조하고 있기 때문에 가비지 컬렉션의 대상으로 판단되지 못하여, 메모리 누수의 원인이 될 수 있다 👀

반면에 표시하고-쓸기 Mark-and-sweep 알고리즘은 root 라는 전역 오브젝트에서 시작해서 root 에 닿지 않는 모든 오브젝트를 가비지로 콜렉트하는 방식이다.  

실질적으로 전역 오브젝트에 닿을 수 있는지의 여부를 판단하므로 조금 더 효율적인 알고리즘이라고 할 수 있다.  

(자바스크립트의 메모리 관리 및 가비지 컬렉션에 대해 아래 아티클을 더 참고하실 수 있습니다 😇 )

[자바스크립트의 메모리 관리
](https://developer.mozilla.org/ko/docs/Web/JavaScript/Memory_Management#%EA%B0%80%EB%B9%84%EC%A7%80_%EC%BD%9C%EB%A0%89%EC%85%98)

[그래서 자바스크립트는 어떻게 쓰레기를 수거하나요?
](https://www.hyesungoh.xyz/how-to-collect-garbage-in-js)

<br/>
<br/>
<br/>


# 🙋🏻‍♀️ 리액트에서 불변성을 강조하는 이유가 뭘까?

불변성이란 한번 생성된 값이 변경되지 않는 것을 의미한다.  

리액트에서는 원시 타입 데이터뿐만 아니라 객체도 불변한 것처럼 다뤄야 한다!

> However, although objects in React state are technically mutable, you should treat them as if they were immutable—like numbers, booleans, and strings. Instead of mutating them, you should always replace them.  
[https://react.dev/learn/updating-objects-in-state#whats-a-mutation](https://react.dev/learn/updating-objects-in-state#whats-a-mutation)

리액트는 상태를 업데이트할 때 얕은 비교를 수행한다.  

이는 데이터가 바라보는 주솟값이 동일한지의 여부를 확인하는 것이다.  

> React compares old and new props by shallow equality: that is, it considers whether each new prop is reference-equal to the old prop.  
[https://react.dev/reference/react/memo#troubleshooting](https://react.dev/reference/react/memo#troubleshooting)

그리고 리액트는 상태가 변경되면 리렌더링한다.  

따라서, 상태에 따라 적절한 UI 를 보여주려면 불변성을 지켜야 한다.  

만약 객체 내부의 프로퍼티 값이 변경됨에 따라서 상태를 업데이트하고 싶은데 가변적인 객체의 상태를 업데이트한다면 얕은 비교를 수행하는 리액트는 상태가 변경되었다고 감지할 수 없기 때문이다.  
