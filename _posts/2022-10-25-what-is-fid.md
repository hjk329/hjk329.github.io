---
title:  "[코어 웹 바이탈 최적화] FID란?"
categories: 
  - 최적화
tags:
  - 리팩토링
  - 최적화
  - 코어 웹 바이탈
toc: true
toc_sticky: true
toc_label: "FID란?"
share: true
---

## FID란?

FID는 유저와 페이지가 처음으로 상호작용하는 순간부터 (예를 들어, 링크를 클릭하거나, 버튼을 탭하는 등) 브라우저가 해당 인터렉션에 대한 응답으로 실제로 이벤트 핸들러 처리를 시작하기까지의 시간을 측정한다.
<br>

> FID는 사용자가 응답하지 않는 페이지와 상호 작용할때 느끼는 경험을 수량화하기 때문에 중요한 사용자 중심 메트릭이다. <br>

### 좋은 FID 점수는?
우수한 사용자 경험을 제공하기 위해서는 사이트의 최초 입력 지연 값이 100밀리초 이하여야 한다.


## FID 상세 정보
일반적으로 입력 지연은 브라우저의 메인 스레드가 다른 작업을 하고 있어서 사용자에게 아직 응답할 수 없기 때문에 발생한다. <br>
대개 브라우저가 큰 자바스크립트 파일을 파싱하고 실행하느라 바쁜 경우에 이와 같은 상황이 발생할 수 있다.

<br>
<br>

웹 페이지가 로딩되는 타임라인을 아래 예시로 살펴보자.
<br>
![timeline](https://web-dev.imgix.net/image/admin/9tm3f6pwlHMqNKuFvaP0.svg)
***이미지 출처: [First Input Delay (FID)](https://web.dev/i18n/en/fid/)***

CSS, JS 파일 등의 리소스는 네트워크 요청으로 다운로드된 이후에 메인 스레드에서 처리된다. <br>

긴 입력 시간 지연은 FCP(First Contentful Paint, 최초 콘텐츠풀 페인트)와 TTI(Time to Interactive, 상호작용까지의 시간) 사이에 발생한다. <br>
