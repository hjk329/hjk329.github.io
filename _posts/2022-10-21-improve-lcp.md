---
title:  "[코어 웹 바이탈 최적화]LCP 최적화하기"
categories: 
  - 최적화
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

최적화된 페이지를 제공하기 위해서는 LCP가 최대한 빠르게 로딩되고, 로딩된 이후에는 최대한 빠르게 렌더링되어야 한다. <br>
총 LCP 에 소요되는 시간은 아래 4가지 부분으로 나뉜다. <br><br>
![LCP sub-parts](https://web-dev.imgix.net/image/eqprBhZUGfb8WYnumQ9ljAxRrA72/4AriEgko87GR1iZSgOou.png?auto=format&w=845)

***이미지 출처: [Optimize Largest Contentful Paint](https://web.dev/i18n/en/optimize-lcp/)***

<br>

각 항목을 살펴보면 아래와 같다. <br>

- TTFB(Time to First Byte): 사용자가 페이지 로딩을 시작하고 브라우저가 HTML 응답으로 첫 바이트를 받아오기까지의 시간을 의미한다. <br> TTFB를 알 수 있는 가장 간단한 방법은 크롬 개발자 도구에서 네트워크 패널을 활용하는 것이다. <br> 네트워크 패널의 타이밍 탭을 보면 선택한 리소스에 대한 네트워크 활동의 분석이 표시되는데 이중에서 Wating for server response 섹션을 참고하면 된다. <br> <br>

우리 페이지는 아래와 같이 서버 응답 시간이 18.69ms 소요되는 것으로 확인되었다. <br> <br>

![[TTFB](/assets/images/TTFB-2022-10-24.png)](/assets/images/TTFB-2022-10-24.png)

좋은 TTFB의 기준은 0.8s 이하라고 한다. 참고로, TTFB는 코어 웹 바이탈에 포함되는 메트릭은 아니다.

참고: [Time to First Byte (TTFB)](https://web.dev/ttfb/#what-is-a-good-ttfb-score)

- Resource load delay: TTFB와 브라우저가 LCP 리소스를 로딩하기 시작하는 사이의 값을 의미한다.

- Resource load time: LPC 리소스를 로드하는 데에 소요되는 시간을 의미한다.

- Element render delay: LCP 리소스가 로딩을 끝내고 LCP 요소가 완전히 렌더링 되는때까지의 사이의 값을 의미한다.

<br>
LCP를 향상시키기 위해서 위 모든 요소를 최적화하는 것이 중요하다. <br>
LCP 향상을 위해 기억해야할 것은 아래와 같다.
- 대부분의 LCP 시간은 HTML과 LCP 리소스를 로딩하는 데에 사용된다.
- LCP 요소 이전에 HTML과 LCP리소스를 로딩하고 있지 않은 곳이 향상시킬 수 있는 기회이다. (예를 들어 Resource load delay)

<br>
그렇다면 위 요소들을 각각 최적화할 수 있는 방법을 알아보자.

## 각 요소들을 최적화하는 방법

### Resource load delay 
이 방법은 LCP 요소가 최대한 빨리 로딩을 시작하도록 하는 것이다. TTFB 직후에 바로 LCP 리소스의 로딩을 시작하면 좋겠지만 실질적으로는 항상 어느 정도의 딜레이가 발생한다. <br>
그래서 페이지에서 가장 처음으로 로드되는 리소스와 LCP 요소가 동시에 로딩되기 시작하는 것이 이상적이다. <br>
따라서, 만약 LCP 요소가 로딩되는 첫번째 리소스가 아니라면 LCP 향상을 위한 여지가 있다는 의미이다. <br> <br>

LCP 요소가 빠르게 로딩되게 하는 데에는 아래 두가지 요소가 영향을 미친다.
- LCP 요소가 언제 발견되는지
- LCP 요소가 부여받은 우선순위가 어떠한지


<br>
#### LCP 요소가 발견되는 떄를 최적화해보자
HTML에서 해당 요소가 LCP 요소임을 발견할 수 있도록 하는 것은 LCP 요소가 빠르게 로딩되는 것을 보장할 수 있다. <br>
예를 들어, 아래와 같은 경우 브라우저는 LCP 요소를 발견할 수 있다.

- LCP 요소가 `<img>` 요소이고, 해당 요소의 src나 srcset 어트리뷰트가 HTML 마크업에 존재한다.
- LCP 요소가 CSS 배경 이미지를 요구하는데 해당 배경 이미지가 HTML 마크업에서 `<link rel="preload">` 로 프리로드된다.
- LCP 요소가 웹 폰트를 요구하는 텍스트 노드인데 해당 폰트가 HTML 마크업에서 `<link rel="preload">` 로 프리로드된다.


<br>

아래의 경우에는 브라우저의 프리로드 스캐너가 LCP 요소를 감지하지 못한다. <br>
- LCP 요소가 `<img>` 요소이고, 자바스크립트를 통해 동적으로 추가된다.
- LCP 요소가 자바스크립트 라이브러리를 통해 lazy loading이 되어 src나 srcset 어트리뷰트가 가려진다.
- LCP 요소가 CSS 배경 이미지를 요구한다.

<br>
위 예시들을 통해 알 수 있는 점은 브라우저가 LCP 요소를 찾고 로딩을 시작하기 위해서는 스크립트를 실행하거나 스타일시트를 적용해야 한다는 것이다. <br> 그리고 스크립트를 실행하거나 스타일시트를 적용하는 동작은 대개 네트워크 요청이 끝날때까지 기다리는 것을 포함한다. <br>

따라서, 불필요한 Resource load delay 를 제거하기 위해서는 LCP 요소가 항상 HTML 마크업에서부터 감지되어야 하는 것이다. <br>
만약에 CSS나 자바스크립트 파일처럼 외부 파일에서만 LCP 요소가 참조된다면 LCP 요소는 fetchpriority 어트리뷰트의 값이 'high'로 프리로드 되어야 한다. <br>

예시 코드는 아래와 같다.

```
<!-- Load the stylesheet that will reference the LCP image. -->
<link rel="stylesheet" href="/path/to/styles.css">

<!-- Preload the LCP image with a high fetchpriority so it starts loading with the stylesheet. -->
<link rel="preload" fetchpriority="high" as="image" href="/path/to/hero-image.webp" type="image/webp">
```


<br>
#### LCP 요소에 부여된 우선순위를 최적화해보자
LCP 요소가 HTML 마크업에서부터 발견된다고 하더라도 우선순위가 높다고 판단되지 않는다면 로딩을 빠르게 시작하지 않을 수 있다. <br>
예를 들어서 LCP 요소인 이미지의 어트리뷰트를 `loading='lazy'` 로 설정했다면 뷰포트 안에서의 이미지를 확인한 이후에 로딩하기 때문에 로딩이 지연될 수 있다. <br>
lazy loading이 아니더라도 이미지는 가장 높은 우선순위를 가지고 로딩되는 요소는 아니기 때문에 로딩이 지연될 수 있다. <br>
해당 이미지 요소의 fetchpriority 어트리뷰트를 설정함으로써 더 빠르게 로딩을 시작할 수 있다. <br>


```
<img fetchpriority="high" src="/path/to/hero-image.webp">
```

<br>
위와 같이 LCP 요소가 HTML 마크업에서부터 발견되며, 프리로드 되거나 높은 우선순위로 로딩을 시작하게 한다면 아래와 같이 리소스 로딩 시간이 단축될 수 있다. <br><br>

![resource load lazy](https://web-dev.imgix.net/image/eqprBhZUGfb8WYnumQ9ljAxRrA72/f9s7SJSBNKcMm3VmcvtT.png?auto=format&w=845)

***이미지 출처: [Optimize Largest Contentful Paint](https://web.dev/i18n/en/optimize-lcp/)***



### Element Render Delay 제거하기
이번 스텝은 LCP 요소가 LCP 요소의 리소스 로딩이 끝난 직후에 바로 렌더링되는 것을 보장하기 위한 방법이다. <br>

기본적으로 LCP 요소가 리소스 로딩이 끝난 이후에 바로 렌더링되지 않는 이유는 아래와 같은 이유로 렌더링이 정체된 것이다. <br>

- `<head>` 의 스타일 시트나 스크립트가 여전히 로딩중이다.
- LCP 리소스 로딩은 마쳤으나, LCP 요소가 DOM에 추가되지 않았다. (자바스크립트 코드가 로딩되기를 기다리는 경우)
- LCP 요소가 어떠한 코드에 가려져있다. (예를 들어 A/B 테스팅 라이브러리에서는 사용자가 어떠한 실험군에 포함될지 판단하고 있는 중일 수 있다.)
- 긴 자바스크립트 작업으로 인해 메인 스레드가 블락되었다. (렌더링은 긴 자바스크립트 작업을 완료할때까지 기다려야 한다.)

다음부터는 불필요하게 요소의 렌더링을 지연하는 요인들을 개선하는 방안을 알아보겠다. <br>

#### 스타일 시트를 줄이거나, 스타일시트를 인라인화하자
HTML 마크업에서 로드되는 스타일시트는 이후의 모든 콘텐츠의 렌더링을 막는다. <br>
만약에 스타일시트가 너무 커서 LCP 리소스보다 로드하는 데에 훨씬 더 오래걸린다면 LCP 요소가 로딩을 마쳤더라도 렌더링되는 것을 막을 수 있다. <br>
아래 이미지와 같은 상황이다.<br><br>
![rendering-block](https://web-dev.imgix.net/image/eqprBhZUGfb8WYnumQ9ljAxRrA72/A5XTlQxIdF9WsLdXiBXE.png?auto=format&w=845)

***이미지 출처: [Optimize Largest Contentful Paint](https://web.dev/i18n/en/optimize-lcp/)***

<br>
<br>

이를 개선하기 위한 옵션은 아래와 같다.
- 추가적인 네트워크 요청을 방지하기 위해 스타일시트를 HTML에 인라인화하거나,
- 스타일시트의 크기를 줄일 수 있다.

<br>

일반적으로 스타일시트를 인라인화하는 것은 HTML의 인라인 콘텐츠는 이후에 페이지를 로드할때 캐싱의 이점을 얻을 수 없으므로 스타일시트가 작은 경우에만 권장된다. <br>

스타일시트가 너무 커서 LCP 리소스보다 로드하는 데 시간이 더 오래 걸린다면 인라이닝에 적합하지 않다.

대부분의 경우에 LCP 요소의 렌더링이 블락되는 것을 방지하기 위한 가장 좋은 방법은 LCP 요소보다 스타일 시트의 크기가 작도록 조정하는 것이다. <br>
스타일시트의 크기를 줄이기 위해 아래와 같은 방법을 제안한다.
- 사용하지 않는 코드를 지우자
- 첫 페이지를 로드하는 데에 필요한 스타일과 그렇지 않은 스타일을 분리해서 lazy loading을 할 수도 있다.
- transfer size를 가능한 작게 줄여서 CSS를 축소하거나 압축해볼 수 있다.

#### 렌더링을 블로킹하는 자바스크립트를 지연시키거나, 인라인화하자
