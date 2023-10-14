---
title:  "자바스크립트의 실행 컨텍스트"
categories: 
  - Books
  - Javascript
tags:
  - 실행 컨텍스트
  - 렉시컬 환경
  - 호이스팅
  - 스코프 체이닝
  - Lexical Environment
  - environmentRecord
  - outerEnvironmentReference
toc: true
toc_sticky: true
toc_label: "자바스크립트의 실행 컨텍스트"
share: true
---

📖 [F-Lab 프론트엔드 멘토링](https://f-lab.kr/mentoring-courses/frontend)을 통해 정재남 님의 코어 자바스크립트 2장 실행 컨텍스트를 읽고 정리한 글입니다.  

# 실행 컨텍스트란

자바스크립트 엔진은 코드를 평가한 뒤에 실행한다.  

실행 컨텍스트는 코드가 실행할 때에 필요한 정보를 담고 있는 객체이다.  

실행 컨텍스트를 콜 스택에 쌓아올렸다가 가장 위에 쌓여있는 콘텍스트의 코드들을 실행하여 전체 코드의 환경과 순서를 보장한다.


# 실행 컨텍스트의 생성

실행 컨텍스트를 생성하는 방법은 아래와 같다.  

1. 전역 컨텍스트: 자바스크립트가 로드되면 생성되고 스크립트가 종료될때까지 유지된다.
2. 함수 컨텍스트: 함수를 호출할때 생성된다.
3. eval 컨텍스트: eval 함수를 호출할때 생성된다.
4. 모듈 컨텍스트: 모듈 코드가 실행될때 생성된다.


# 실행 컨텍스트의 구성

실행 컨텍스트는 아래와 같이 구성된다.

- VariableEnvironment
- LexicalEnvironment
    - environmentRecord
    - outerEnvironmentReference
- ThisBinding  


VariableEnvironment 와 LexicalEnvironment 모두 현재 컨텍스트 내의 식별자에 대한 정보, 외부 환경 정보 등을 담고 있지만 LexicalEnvironment 는 실시간으로 변경 사항이 반영된다는 점이 다르다.  



# 호이스팅과 environmentRecord

코드가 평가되는 시점에 LexicalEnvironment 의 envorinmentRecord 에 현재 컨텍스트와 관련있는 식별자의 정보가 수집된다.  

이는 식별자에 값이 할당되기 전이나, 함수가 호출되기 전에 이미 식별자나 함수의 이름을 알고 있는 것이다.  

마치 코드를 실행하기도 전에 해당 코드가 최상단으로 끌어올려진 것처럼 동작하는 것을 호이스팅이라고 한다.  

- var 키워드로 선언한 식별자는 선언과 동시에 undefined 로 초기화된다.
- let, const 키워드로 선언한 식별자의 정보도 envorinmentRecord 에 수집되지만, 실제 선언한 위치에 도달하기 전까지는 해당 식별자를 실질적으로 사용할 수 없는 TDZ(Temporarily Dead Zone) 이 발생한다.
- 함수 선언문은 함수 표현식과 다르게 선언문 전체가 호이스팅된다. 만약 같은 컨텍스트에서 같은 이름으로 함수 선언문을 작성하고, 선언부 이전에 호출한다면 코드는 동작하는데 내가 짠 함수는 아닌.. 그런 디버깅을 어렵게 하는 상황이 있을 수 있으니 함수 표현식을 사용하는게 보다 안전하겠다!


# 스코프 체이닝과 outerEnvironmentReference

자바스크립트 엔진이 function 키워드를 만나서 함수 인스턴스를 만들때 함수가 선언된 위치 (시점) 에서 참조할 수 있는 외부 스코프의 식별자 정보를 [[Scopes]] 라는 내부 슬롯에 수집한다. 이때 설정된 스코프는 변경되지 않는다.  

이를 정적 스코핑이라고 한다.  

> 정적 스코핑: 변수가 선언된 위치에 따라 그 범위가 결정되는 방식, 렉시컬 스코핑(Lexical Scoping) 이라고도 한다.

![image](https://github.com/hjk329/hjk329.github.io/assets/84058944/8a8ea47b-c03a-43ea-bab3-c711fe3d4127)


그리고 이는 LexicalEnvironment의 outerEnvironmentRecord 에 저장된다.  

즉, 함수 내부 스코프에서 바로 직전 스코프의 LexicalEnvironment 를 참조할 수 있게 된다. 

```js
let globalVar = '전역 변수';

function outerFunc() {
    let outerVar = '외부 함수 변수';
    
    function innerFunction() {
        let innerVar = '내부 함수 변수';

        console.log(innerVar);  // 내부 함수 변수
        console.log(outerVar);  // 외부 함수 변수
        console.log(globalVar); // 전역 변수
    }
    
    innerFunction();
}

outerFunc();
```

이처럼 스코프를 안에서부터 바깥으로 차례로 검색해가는 것을 스코프 체인이라고 한다.  

스코프 체이닝을 잘 이해하면 클로저의 개념도 자연스레 해결된다!  

클로저는 외부 함수가 종료되었음에도 내부 함수에서 외부함수에 선언된 변수를 여전히 참조할 수 있는 현상이다.  

이는 함수가 선언될 당시에 내부 슬롯 [[Scopes]] 에 해당 함수가 참조할 수 있는 변수의 렉시컬 환경 정보가 저장되는 것이기 때문이다.  

(그럼 이만 총총총 ..)

<img width="200" alt="스크린샷 2023-09-02 오전 12 12 42" src="https://github.com/hjk329/hjk329.github.io/assets/84058944/8d26095c-07e1-470e-9c37-5f0a345935ef">   

이미지 출처: [@namsee.jpg](https://www.instagram.com/p/CsDYlY8vGCo/?utm_source=ig_web_copy_link&igshid=MzRlODBiNWFlZA==)

