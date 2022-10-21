---
title:  "[코어 웹 바이탈 최적화]LCP 최적화하기"
categories: 
  - 리팩토링
  - 최적화
  - 코어 웹 바이탈
tags:
  - 리팩토링
  - 최적화
  - 코어 웹 바이탈
toc: true
toc_sticky: true
toc_label: "LCP 최적화하기"
share: true

---

LCP 점수를 개선하기 위한 방안을 알아보자!

> LCP 에 관한 자세항 사항은 아래 포스팅을 참고해주세요!

[[코어 웹 바이탈 최적화] LCP(Largest Contentful Paint, 최대 콘텐츠풀 페인트)](https://hjk329.github.io/%EB%A6%AC%ED%8C%A9%ED%86%A0%EB%A7%81/%EC%B5%9C%EC%A0%81%ED%99%94/%EC%BD%94%EC%96%B4%20%EC%9B%B9%20%EB%B0%94%EC%9D%B4%ED%83%88/lcp/)

## 느린 서버 응답 시간
서버에서 응답을 받기까지 시간이 오래 걸린다면 응답 받아온 데이터를 화면에 렌더링 하는 데에도 시간이 많이 소요될 것은 당연하다. <br>
