---
title:  "[코어 웹 바이탈 최적화] LCP 최적화하기"
categories: 
  - 최적화
tags:
  - 리팩토링
  - 최적화
  - 코어 웹 바이탈
  - LCP
toc: true
toc_sticky: true
toc_label: "LCP 최적화하기"
share: true
---

LCP 점수를 개선하기 위한 방법을 알아보자!

> LCP 에 관한 자세항 사항은 아래 포스팅을 참고해주세요! <br>
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

- Resource load delay(리소스 로드 지연): TTFB와 브라우저가 LCP 리소스를 로딩하기 시작하는 사이의 값을 의미한다.

- Resource load time(리소스 로드 타임): LPC 리소스를 로드하는 데에 소요되는 시간을 의미한다.

- Element render delay(요소 렌더 지연): LCP 리소스가 로딩을 끝내고 LCP 요소가 완전히 렌더링 되는때까지의 사이의 값을 의미한다.

<br>
LCP를 향상시키기 위해서 위 모든 요소를 최적화하는 것이 중요하다. <br>
LCP 향상을 위해 기억해야할 것은 아래와 같다.
- 대부분의 LCP 시간은 HTML과 LCP 리소스를 로딩하는 데에 사용된다.
- HTML과 LCP리소스 중 하나가 로딩중이 아닌 LCP 이전에는 언제든지 개선할 수 있다.

<br>
위 요소들을 각각 최적화할 수 있는 방법을 알아보자.

## 각 요소들을 최적화하는 방법

### 리소스 로드 지연 제거하기
이 방법은 LCP 요소가 최대한 빨리 로딩을 시작하도록 하는 것이다. TTFB 직후에 바로 LCP 리소스의 로딩을 시작하면 좋겠지만 실질적으로는 항상 어느 정도의 딜레이가 발생한다. <br>
그래서 페이지에서 가장 처음으로 로드되는 리소스와 LCP 요소가 동시에 로딩되기 시작하는 것이 이상적이다. <br>
따라서, 만약 LCP 요소가 로딩되는 첫번째 리소스가 아니라면 LCP 향상을 위한 여지가 있다는 의미이다. <br> <br>

LCP 요소가 빠르게 로딩되게 하는 데에는 아래 두가지 요소가 영향을 미친다.
- LCP 요소가 언제 발견되는지
- LCP 요소가 부여받은 우선순위가 어떠한지


<br>
#### LCP 요소가 발견되는 때를 최적화하자
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
위 예시들은 공통적으로 브라우저가 LCP 요소를 찾고 로딩을 시작하기 위해서 스크립트를 실행하거나 스타일시트를 적용해야 한다. <br> 스크립트를 실행하거나 스타일시트를 적용하는 동작은 대개 네트워크 요청이 끝날때까지 기다려야 한다. <br> 최적이 로딩이 아닌 셈이다.

따라서, 불필요한 리로스 로드 지연을 제거하기 위해서는 LCP 요소가 항상 HTML 마크업에서부터 감지되는 것이 좋다. <br>
만약에 CSS나 자바스크립트 파일처럼 외부 파일에서만 LCP 요소가 참조된다면 LCP 요소는 fetchpriority 어트리뷰트의 값이 'high'로 프리로드 되게 해서 LCP 요소의 로딩을 최적화할 수 있다. <br>

예시 코드는 아래와 같다.

```
<!-- Load the stylesheet that will reference the LCP image. -->
<link rel="stylesheet" href="/path/to/styles.css">

<!-- Preload the LCP image with a high fetchpriority so it starts loading with the stylesheet. -->
<link rel="preload" fetchpriority="high" as="image" href="/path/to/hero-image.webp" type="image/webp">
```


<br>
#### LCP 요소에 부여된 우선순위를 최적화하자
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



### 렌더링 지연 제거하기
이번 스텝은 LCP 요소가 LCP 요소의 리소스 로딩이 끝난 후에 바로 렌더링되는 것을 보장하기 위한 방법이다. <br>

기본적으로 LCP 요소가 리소스 로딩이 끝난 이후에 바로 렌더링되지 않는 이유는 아래와 같은 이유로 렌더링이 정체되었기 떄문이다. <br>

- `<head>` 의 스타일 시트나 스크립트가 여전히 로딩중이다.
- LCP 리소스 로딩은 마쳤으나, LCP 요소가 DOM에 추가되지 않았다. (자바스크립트 코드가 로딩되기를 기다리는 경우)
- LCP 요소가 어떠한 코드에 가려져있다. (예를 들어 A/B 테스팅 라이브러리에서는 사용자가 어떠한 실험군에 포함될지 판단하고 있는 중일 수 있다.)
- 긴 자바스크립트 작업으로 인해 메인 스레드가 블락되었다. (자바스크립트 작업이  완료되기까지 렌더링이 블로킹된다.)

다음부터는 불필요하게 요소의 렌더링을 지연하는 요인들을 개선하는 방안을 알아보겠다. <br>

#### 스타일 시트를 줄이거나, 스타일시트를 인라인화하자
HTML 마크업에서 로드되는 스타일시트는 이후의 모든 콘텐츠의 렌더링을 막는다. <br>
만약에 스타일시트가 너무 커서 LCP 리소스보다 로드하는 데에 훨씬 더 오래걸린다면 LCP 요소가 로딩을 마쳤더라도 렌더링되는 것을 막을 수 있다. <br>
아래 이미지와 같은 상황이다.<br><br>
![rendering-block](https://web-dev.imgix.net/image/eqprBhZUGfb8WYnumQ9ljAxRrA72/A5XTlQxIdF9WsLdXiBXE.png?auto=format&w=845)

***이미지 출처: [Optimize Largest Contentful Paint](https://web.dev/i18n/en/optimize-lcp/)***

<br>


이를 개선하기 위한 옵션은 아래와 같다.
- 추가적인 네트워크 요청을 방지하기 위해 스타일시트를 HTML에 인라인화하거나,
- 스타일시트의 크기를 줄일 수 있다.

일반적으로 스타일시트를 인라인화하는 것은 스타일시트가 작은 경우에만 권장된다. <br> HTML의 인라인 콘텐츠는 이후에 페이지를 로드할때 캐싱의 이점을 얻을 수 없기 때문이다. <br>

스타일시트가 너무 커서 LCP 리소스보다 로드하는 데 시간이 더 오래 걸린다면 인라이닝에 적합하지 않다.

<br>

대부분의 경우에 LCP 요소의 렌더링이 블락되는 것을 방지하기 위한 가장 좋은 방법은 LCP 요소보다 스타일 시트의 크기가 작도록 조정하는 것이다. <br>
스타일시트의 크기를 줄이기 위해 아래와 같은 방법을 제안한다.
- 사용하지 않는 코드를 지운다
- 첫 페이지를 로드하는 데에 필요한 스타일과 그렇지 않은 스타일을 분리해서 lazy loading을 할 수도 있다
- transfer size를 가능한 작게 줄여서 CSS를 축소하거나 압축해볼 수 있다

#### 서버 사이드 렌더링을 하자
서버 사이드 렌더링(SSR)은 서버에서 클라이언트 어플리케이션 로직을 실행하고 응답으로 완전한 HTML 마크업을 내려주기 때문에 LCP 향상에 아래와 같은 이점이 있다. <br>
- 이미지 리소스가 HTML 마크업에서 발견될 것이다
- 페이지의 콘텐츠가 렌더링되기 위해서 추가적인 자바스크립트 요청이 완료되기까지 기다리지 않아도 된다.


SSR은 서버에서 처리할 추가 시간이 요구되는데 이는 TTFB를 늦출 수 있다. <br>
이러한 트레이드 오프(TTFB를 늦추는 대신에 리소스 렌더링 타임을 줄이는 것)는 고려해볼 만 하다. <br> 왜냐하면 서버에서의 처리 시간은 제어할 수 있는 반면에 사용자의 네트워크 및 장치 기능은 그렇지 않기 때문이다.
<br>

SSR과 유사한 옵션으로는 SSG(static site generation) 혹은 프리렌더링이 있다. <br>
HTML은 사용자의 요청 시점이 아닌 빌드 시점에 생성되므로 LCP 요소가 HTML 마크업에서부터 발견될 수 있겠다.


## 리소스 로드 타임을 줄이자
해당 스텝의 목적은 네트워크를 통해 리소스의 바이트를 사용자의 장치로 전송하는 데 소요되는 시간을 줄이는 것이다. <br>

- 리소스의 크기를 줄인다.
- 리소스가 이동해야하는 거리를 줄인다.
- 네트워크 대역폭에 대한 경합을 줄인다.
- 네트워크 타임을 완전히 제거한다.


<br> 각 요소를 간략히 알아보겠다.

#### 리소스의 크기를 줄인다
LCP 요소는 이미지 혹은 웹 폰트일 수 있다. 다음 가이드를 통해 사이즈를 어떻게 줄일 수 있는지 살펴보자.
- [최적의 이미지 크기를 제공하자](https://web.dev/uses-responsive-images/)
- [모던한 이미지 포맷을 제공하자](https://web.dev/uses-webp-images/)
- [이미지를 압축하자](https://web.dev/uses-optimized-images/)
- [웹폰트 크기를 줄이자](https://web.dev/reduce-webfont-size/)


#### 리소스가 이동해야 하는 거리를 줄인다
CDN을 이용해서 리소스가 이동해야하는 물리적인 거리를 줄일 수 있다. <br>
CDN을 이용하면 일반적으로 리소스의 크기를 줄여 이전의 모든 크기 축소 권장 사항을 자동으로 구현할 수 있으므로 리소스 로드 타임을 줄일 수 있는 좋은 선택이다. <br>


#### 네트워크 대역폭에 대한 경합을 줄인다
리소스의 크기와 이동해야 하는 거리를 줄인 경우에도 다른 많은 리소스를 동시에 로드하는 경우 리소스를 로드하는 데 시간이 오래 걸릴 수 있다. 이를 네트워크 경합이라고 한다. <br>

LCP 리소스에 high fetchpriority를 부여하고 가능한 빨리 로드를 시작했다면 브라우저는 우선 순위가 낮은 리소스가 경쟁하지 않도록 할 것이다. <br>
그러나 high fetchpriority로 많은 리소스를 로드하거나, 일반적으로 많은 리소스를 로드하는 경우 LCP 리소스 로드 속도에 영향을 줄 수 있다.


#### 네트워크 타임을 완전히 제거한다
효율적인 캐시 정책으로 정적 리소스들을 제공한다면 캐시에서 가져온 리소스를 보여줄 것이므로 리소스 로드 타임을 0으로 만들 수 있다. <br>


### TTFB를 줄이자
해당 스텝의 목표는 초기 HTML을 가능한 빨리 전달하는 것이다. 개발자가 제어할 수 있는 가장 적은 부분이면서도 모든 단계에 직접적인 영향을 미치는 중요한 단계이다. <br>
백엔드에서 첫번째 바이트를 제공할때까지 프론트엔드에서 할 수 있는 일이 없으므로, TTFB 속도를 높일 수 있다면 다른 모든 로드 메트릭도 개선할 수 있다. <br>
TTFB를 개선하는 방안은 아래 페이지를 참고하자.
[How to improve TTFB](https://web.dev/ttfb/#how-to-improve-ttfb)

### 정리
LCP는 로딩 성능과 관련된 메트릭이므로 이를 개선하면 로드와 관련된 다른 메트릭들도 향상될 수 있다. <br>

LCP를 최적화하는 방안은 아래와 같다.

1. LCP 리소스가 최대한 빠르게 로딩을 시작하도록 하자.
2. LCP 요소가 로딩을 마친 후 최대한 빠르게 렌더링되도록 하자.
3. LCP 리소스의 로드 타임을 줄이자.
4. 초기 HTML을 최대한 빠르게 전달하자.



---------
참고<br>
[Optimize Largest Contentful Paint](https://web.dev/i18n/en/optimize-lcp/)
