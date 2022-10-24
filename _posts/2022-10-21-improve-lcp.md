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

## 느린 서버 응답 시간
서버에서 응답을 받기까지 시간이 오래 걸린다면 응답 받아온 데이터를 화면에 렌더링 하는 데에도 시간이 많이 소요될 것은 당연하다. <br>
따라서 더 빠른 서버 응답 시간을 보장하는 것은 LCP 뿐만이 아니라 로딩 성능을 향상 시키기 위해서 중요하다. <br> <br>
서버 응답 시간은 Time To First Bye(최초 바이트까지의 시간, TTFB)로 측정할 수 있다. <br>
TTFB를 알 수 있는 가장 간단한 방법은 크롬 개발자 도구에서 네트워크 패널을 활용하는 것이다. <br>
네트워크 패널의 타이밍 탭을 보면 선택한 리소스에 대한 네트워크 활동의 분석이 표시되는데 이중에서 Wating for server response 섹션을 참고하면 된다. <br> <br>

우리 페이지는 아래와 같이 서버 응답 시간이 18.69ms 소요되는 것으로 확인되었다. <br> <br>

![[TTFB](/assets/images/TTFB-2022-10-24.png)](/assets/images/TTFB-2022-10-24.png)


좋은 TTFB의 기준은 0.8s 이하라고 한다. 참고로, TTFB는 코어 웹 바이탈에 포함되는 메트릭은 아니다.

참고: [Time to First Byte (TTFB)](https://web.dev/ttfb/#what-is-a-good-ttfb-score)

<br>

TTFB를 개선할 수 있는 방안은 아래와 같다.

- 서버 최적화
- 사용자를 가까운 CDN으로 라우팅
- 자산 캐시
- HTML 페이지 캐시 우선 제공
- 조기에 타사 연결 구축
- 서명된 교환 사용


<br>

각 항목을 간단히 살펴보겠다. <br>

### 서버 최적화
