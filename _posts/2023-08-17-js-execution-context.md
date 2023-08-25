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
published: false
---

정재남 님의 코어 자바스크립트 2장 실행 컨텍스트를 읽고 정리한 글입니다.  

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
