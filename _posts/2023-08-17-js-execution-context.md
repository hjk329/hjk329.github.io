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

## 실행 컨텍스트의 생성

실행 컨텍스트를 생성하는 방법은 아래와 같다.  

1. 전역 컨텍스트: 자바스크립트가 로드되면 생성되고 스크립트가 종료될때까지 유지된다.
2. 함수 컨텍스트: 함수를 호출할때 생성된다.
3. eval 컨텍스트: eval 함수를 호출할때 생성된다.
4. 모듈 컨텍스트: 모듈 코드가 실행될때 생성된다.
