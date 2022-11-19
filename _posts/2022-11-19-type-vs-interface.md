---
title:  "Type과 Interface의 차이점: OCP(Open Closed Principle)"
categories: 
  - Typescript
tags:
  - Type
  - Interface
  - OCP
toc: true
toc_sticky: true
toc_label: "Type vs Interface"
share: true
published: false

---

개발팀 팀원분들과 깜퀴(깜짝 퀴즈)를 진행하고 있다 🤭  
슬랙에 `돌아온 깜퀴`라는 채널을 만들었는데 해당 채널에 개발과 관련된 퀴즈를 툭툭 던지는 형태이다.    


작년에 코어 자바스크립트라는 책을 팀원분들과 함께 읽으며 불시에 코어 자바스크립트 책을 펴고 서로에게 깜짝 질문을 던지곤 했는데 그 깜퀴가 부활했다.  

하여튼 돌아온 깜퀴 그 첫번째 문제는 타입과 인터페이스의 차이점이다.  

## 타입과 인터페이스는 어떻게 다를까?
1. OCP(Open Closed Principle, 개방 폐쇄 원칙)
확장의 측면에서 봤을때 인터페이스에서는 OCP를 적용할 수 있다.  

> OCP란?

