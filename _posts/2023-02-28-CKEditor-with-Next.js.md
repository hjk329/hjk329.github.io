---
title:  "Next.js에서 CKEditor5 사용하기"
categories: 
  - Blog
tags:
  - CKEditor5
  - Next.js
  - Global CSS cannot be imported from within node_modules.
  - Online Builder
  - React-Quill
toc: true
toc_sticky: true
toc_label: "Next.js와 CKEditor5 🫥"
share: true
published: false,

---

호옥시 Next.js에 CKEditor를 설치하시다가 고통 받고 들어오셨나요 !!  
저도 그 마음을 잘 압니다 😇   
썰을 풀어보겠읍니다. . .

---

## 왜 CKEditor를 선택했는가?

### 시작에는 `React-Quill` 이 있었다
우리 프로젝트는 Next.js를 기반으로 한다.  
CMS를 개발하면서 에디터가 필요해서 [`React-Quill(이하 리액트 퀼)`](https://github.com/zenoamaro/react-quill) 을 설치했었다.    

간단한 글 작성 및 편집을 할 수 있으면서 Next.js와의 호환성도 좋고, 타입스크립트도 지원되어서 아주 편리하게 사용 중이었다.  

이후 CMS 고도화가 필요해지면서 에디터에 조금 더 풍부한 기능들이 요구되었다.  
예를 들어서, 구분 선을 보여준다거나 테이블, 이미지 업로드 시 자동으로 캡션이 추가되는 기능 등등이었다.  

처음에는 객기(?)로 모든 기능들을 커스터마이징하려고 했다. 🐶  
그러다 보니 에디터 개발 그 자체가 태스크가 되어버린 듯했다.  


사실 이번 스프린트의 목적은 개발자를 통해서 JSON으로 관리되던 콘텐츠를, 개발자가 아닌 팀원분들도 에디터를 통해서 발행하고, 발행된 콘텐츠를 우리 디자인 시스템에 맞게 스타일을 적용하는 것이었다.   
에디터를 잘 만드는 것도 중요했지만, 에디터 자체를 개발하는 것이 스프린트의 주된 목적은 아니었다.  


### `라이브러리를 라이브러리답게 사용하자`

리액트 퀼을 커스터마이징하기 위해 사투를 벌이던 중에  
`라이브러리를 라이브러리답게 사용하자` 라는 짱짱 팀원분의 이야기를 듣고 '라이브러리를 사용하는 목적은 이미 잘 만들어진 소프트웨어를 편리하게 사용하는 것에 있다'고 다시금 인지했다.  


이에 따라 에디터와 관련한 요구 사항을 가장 빠르고 편리하게 구현할 수 있는 다른 라이브러리를 찾기 시작했다.  

### 검증 과정

CKEditor는 홈페이지에서 강조하는 것처럼 리치한 에디터이다.  
데모를 사용해보며 우리에게 필요한 기능들을 손쉽게 구현할 수 있다고 생각했다.  

또한, `Snyk Open Source Advisor`에서 확인한 종합 점수도 꽤 괜찮은 편이라서 어느 정도 검증받은 라이브러리라고 생각했다.  
[snyk Advisor: CKEditor5](https://snyk.io/advisor/npm-package/ckeditor5)  
