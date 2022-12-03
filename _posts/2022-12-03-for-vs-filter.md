---
title:  "for loop와 filter 메소드 비교"
categories: 
  - Javascript
tags:
  - 반복문
  - for
  - filter
toc: true
toc_sticky: true
toc_label: "for loop vs filter"
share: true

---

## for loop 와 filter 중에 무엇을 사용할까?

어떠한 기준으로 `for loop` 혹은 `filter` 메소드를 사용하는걸까?  


```javascript
const items = ['abc', 'def', 'ghi', 'jk'];
```

```javascript
const threes = items.filter(item => item.length === 3);
```


```javascript
const threes = [];
for(let i = 0; i < items.length; i++) {
  if (items[i].length === 3) {
    items.push(items[i]);
  }
}
```


고려해야할 점은 아래와 같다.

1. 어떤 방법이 더 빠른가?
2. 어떤 방법이 더 스케일러블한가?
3. 어떤 방법이 가독성과 유지보수 측면에서 더 나은가?

하나씩 살펴보겠다.  

## 1. 성능
### `for loop` 가 훨!씬 더 빠르다  
심지어, `filter` 메소드는 `for loop` 보다 77%나 느리다.  

이는 `for loop`는 동기적으로 실행되고, `filter` 메소드는 배열의 각 요소에 대해 1개의 새로운 함수를 생성하기 때문이다.  


새롭게 생성된 함수는 콜 스택에 배치되고 이벤트 루프에 의해 하나씩 실행된다.  
반면에 `for loop` 는 같은 함수 내에서 바로 작업을 실행하기 때문에 새로 생성된 함수가 콜 스택에 배치되는 시간은 아예 배제된다.  

성능면에서는 `for loop`의 압승이라고 볼 수 있겠다.  

## 2. 가독성

### 가독성면에서는 `filter` 메소드가 훨씬 우세하다.  

```javascript
const threes = []; 
for(let i = 0; i < items.length; i++) { 
 if (items[i].length === 3) {items.push(items[i]);
 }
}
```


- 첫번째 라인은 의미가 없다. 그저 필터된 새 배열을 저장할 변수를 선언할 뿐이며, 아무런 비즈니스 로직도 포함하지 않는다. 
- 두번째 라인 또한 의미 없다. 비즈니스 로직이 포함되지 않았고, 반복문을 돌리기 위한 코드이다.
- 세번째 라인에는 비즈니스 로직이 포함된다.
- 네번째 라인에는 비즈니스 로직이 포함되지 않는다. 그저 해당 함수를 종료하는 내용이다.  

```javascript
const threes = items.filter(item => item.length === 3);
```

그에 반해 `filter` 메소드를 사용했을때 단 한줄로도 같은 로직을 처리할 수 있다 !!

## 3. 스케일 확장성
자바스크립트는 싱글 스레드 언어이므로 런타임에 1개의 요청을 실행한다.  
여러 개의 요청이 동시에 들어올때는 어떻게 처리할까?  
이벤트 루프는 싱글 스레드를 공유할 수 있도록 만든다.  

콜백 큐는 FIFO(First In First Out) 방식으로 이벤트들을 쌓아간다.  
이벤트 루프에 의해 함수가 실행되면 LIFO(Last In First Out) 방식으로 콜스택에 쌓인다.  

`for loop` 혹은 `forEach`는 이벤트 루프를 사용하지 않고, 동기적으로 실행된다.
- `for loop`: 동기적으로 실행됨
- `forEach`: 콜백 함수가 비동기 처리를 하든 말든 기다려주지 않고 계속 반복문을 실행함

즉, `for loop` 혹은 `forEach`를 사용하면 스레드에서 실행되어야 하는 다른 작업이 블록되거나, 반복문을 통해 우리가 기대한 결과값을 얻지 못할 수도 있다.


## 결론
만약 배열이 비교적 작다면 런타임 성능상 `for loop` 를 사용하는 것이 나을 것이다.  
하지만, 확장성을 고려하고, 가독성 및 유지보수를 생각한다면 `filter` 메소드가 더 적절할 수 있겠다.


## 번외: 선언형 프로그래밍 vs 명령형 프로그래밍
```javascript

// for loop는 코드를 명령형으로 작성해야 한다
function calculate1(arr){
    let resultl = [];
    for(let i =0; i < arr.length; i++>){
        results.push(arr[i] + 2)
    }
    return results
}
// map, filter는 코드를 선언형으로 작성할 수 있다.
function calculate2(arr) {
    return arr.map(number => number + 2);
}

```

`map`, `filter` 메소드와 같이 선언형으로 프로그래밍하면 `어떻게` 해야 할지 작성하지 않아도 된다.   
반복문 순회시 `무엇을` 할지 작성해주면 된다.  


반면에 `for loop` 와 같은 명령형 프로그래밍은 반복문을 `어떻게` 순회해야 할지 직접 작성해줘야 한다.  
위 코드를 예로 들면 변수 i가 배열의 길이보다 작을때까지 증가하면서 등등 어떤식으로 순회해야하는지 직접 명령해줘야 한다.  


참고: [Are JavaScript for loops better than filter() and forEach?()](https://plainenglish.io/blog/are-for-loops-better-than-arrays-filter-or-foreach-methods)  