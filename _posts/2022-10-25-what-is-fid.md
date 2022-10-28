---
title:  "[코어 웹 바이탈 최적화] FID란?"
categories: 
  - 최적화
tags:
  - 리팩토링
  - 최적화
  - 코어 웹 바이탈
  - FID
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


아래 예시 이미지를 보면 FCP와 TTI 사이에 긴 시간이 소요되는 3개의 작업이 포함된 것을 확인할 수 있다. <br>

![timeline](https://web-dev.imgix.net/image/admin/krOoeuQ4TWCbt9t6v5Wf.svg)
***이미지 출처: [First Input Delay (FID)](https://web.dev/i18n/en/fid/)***


<br>

즉, 사용자가 입력을 했다고 하더라도 브라우저는 작업중이기 때문에 사용자의 입력에 대해 응답하려면 진행중인 작업이 완료될때까지 기다려야 한다. <br>

참고로, 위 예시 이미지는 메인 스레드가 가장 바쁘게 작업중일때 사용자의 입력 이벤트가 발생했다. <br> 만약 조금 더 일찍 페이지와 상호작용했다면 브라우저가 즉시 응답했을 수도 있겠다. <br>

따라서, FID를 분석할때는 그 분포값을 살펴보는 것이 중요하다. 이와 관련한 사항은 아래에서 더 자세히 알아보겠다. <br>


### 상호 작용에 이벤트 리스너가 없으면 FID를 어떻게 측정할까?
***이벤트 리스너가 등록되지 않은 경우에도 FID는 측정된다.***
FID는 입력 이벤트가 수신된 시점과 메인 스레드가 다음 유휴 상태에 들어간 시점 사이의 델타를 측정하기 때문이다. <br>
이벤트 리스너를 통해 FID를 측정하지 않는 이유는, 대다수의 사용자 상호작용에 이벤트 리스너가 항상 필요한 것은 아니지만, 해당 상호작용을 실행하기 위해서는 메인 스레드가 유휴 상태가 되어야 하기 때문이다.
<br>

예를 들어, 다음 HTML 요소는 모두 사용자 상호작용에 응답하기 전에 메인 스레드에서 진행중인 작업이 완료되어야 한다.
<br>

-  텍스트 필드, 테크박스, 라디오 버튼 (`<input>` , `<textarea>`)
-  선택 드롭다운(`<select>`)
-  링크 (`<a>`)


### 최초 입력만 고려하는 이유?
첫번째 입력 지연을 측정하는 것이 좋은 이유는 아래와 같다. <br>


- 최초 입력 지연은 사이트의 응답성에 대한 사용자의 첫인상이 되어 사이트의 품질과 신뢰성에 대한 인상을 형성하는 데 중요하다.
- 오늘날 웹에서 볼 수 있는 가장 큰 상호 작용 문제는 페이지 로드 중에 발생한다. 최초 사용자 상호작용을 개선하는 데 중점을 두면 이러한 문제를 자연스럽게 해결할 수 있을 것이다.



### 최초 입력으로 간주되는 것은 무엇인가?
클릭, 탭 및 키 누름과 같은 이벤트이다. <br>
스크롤 및 확대/축소는 제외된다. 이는 애니메이션에 가까운 연속 작업에 해당하며, 위의 입력 이벤트와 완전히 다른 성능 제약 조건을 가지기 때문이다. <br>


### 사용자가 사이트와 상호 작용하지 않으면 어떻게 될까?
모든 사용자가 방문할 때마다 사이트와 상호 작용하는 것은 아니다. <br>
또한, 타이밍에 따라서 (메인 스레드가 바쁘게 작업중이거나, 유휴 상태이거나) 사용자마다 FID 값이 없거나, 낮거나, 높을 수 있다. <br>

따라서, FID를 추적하는 방법은 다른 메트릭과는 차별점이 있어야 한다.


### 입력 지연만 고려하는 이유?
FID는 '이벤트 처리의 지연' 만 측정한다. <br> 이벤트 처리 시간 자체나 브라우저에서 이벤트 핸들러를 실행한 후 UI를 업데이트하는 데 걸리는 시간은 측정하지 않는다. <br> <br>

이벤트 처리 시간이나 UI를 업데이트하기까지의 시간도 물론 중요하지만 이러한 시간이 메트릭에 포함된다면 실제적으로 사용자 경험에는 악영향을 미치지만, 메트릭 점수만 개선할 수 있는 워크로드가 추가될 여지가 있기 때문이다. <br>

예를 들어 이벤트 핸들러 로직을 `setTimeout()` 또는 `requestAnimationFrame()` 과 같은 비동기화 콜백으로 감싼다면 이벤트와 관련된 작업은 분리되어 메트릭 점수는 높아질 수 있다. <br>
하지만, 사용자가 인지하는 응답 시간은 더 느려지게 된다.


## FID 측정 방법
FID는 실제로 사이트와 상호작용할 사용자가 필요하기 때문에 사이트에서만 측정할 수 있는 메트릭이다. 실험실에서 측정할 수 없다. <br>
아래와 같은 방법으로 FID를 측정해볼 수 있다.

- Chrome User Experience Report
- PageSpeed Insights
- Search Console(Core Web Vitals Report)
- web-vitals Javascript 라이브러리


### FID 데이터 분석 및 보고
코어 웹 바이탈 임계값은 모두 75번째 백분위수 값으로 성능을 분류하고 있다. <br>
그러나, FID는 특별히 95~99번째 백분위수를 확인하는 것이 좋다. <br>
왜냐하면 해당 부분이 사용자가 사이트에서 특별히 좋지 않다고 느끼는 '첫번째 경험'이기 때문이다.

앞으로는 FID를 개선하는 방안을 알아보겠다.