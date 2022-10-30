---
title:  "[코어 웹 바이탈 최적화] 코어 웹 바이탈이란?"
categories: 
  - 최적화
tags:
  - 리팩토링
  - 최적화
  - 코어 웹 바이탈
toc: true
toc_sticky: true
toc_label: "코어 웹 바이탈이란?"
share: true

---

사내 프로젝트는 구글 서치 콘솔 서비스로 구글 검색결과 및 사이트 사용 경험에 대한 결과를 모니터링하고 있다. <br>

구글 서치 콘솔에서는 코어 웹 바이탈이라는 섹션을 확인할 수 있는데 해당 섹션에서 제공되는 차트를 통해 느린 url, 개선이 필요한 url 등을 날짜별로 확인할 수 있었다. <br>

해당 차트를 통해 개선해야할 부분을 파악하여 프로젝트를 최적화하면 좋겠다고 생각했다. <br>

차트를 제대로 이해하기 위해서 코어 웹 바이탈이 무엇인지 먼저 알아보려 한다.


<br>
<br>


## 코어 웹 바이탈이란?

최적화된 사용자 경험을 제공하는 것은 웹 사이트를 성공적으로 유지하기 위해 필수적이다. <br>
구글에서는 웹 사이트의 성능을 측정하고 보고하는 다양한 지표와 도구를 제공해왔으며, '코어 웹 바이탈(Core Web Vitals)' 이라는 가장 중요한 메트릭을 소개했다.<br>
말 그대로 핵심이 되는 필수적인 지표들을 포함한다.<br>

> ***메트릭이란?*** <br>

성능은 상대적일 수 있다. <br>
왜냐하면 동일한 사이트이더라도 빠른 네트워크를 사용하는 사용자에게는 빠르고, 느린 네트워크를 사용하는 사용자에게는 느리게 동작할 수 있기 때문이다. <br>
따라서, 이러한 상대성을 줄이고 정확하고 정량적이며 객관적인 기준에서 성능을 비교할 수 있어야하는데 이러한 객관적인 기준을 '메트릭'이라고 한다.


참고: [사용자 중심 성능 메트릭](https://web.dev/user-centric-performance-metrics/#how-metrics-are-measured)

<br>
코어 웹 바이탈을 구성하는 메트릭은 시간에 따라 그 구성이 달라질 수 있다. 2020년 기준으로는 사용자 경험의 세 가지 측면인 로딩, 상호 작용, 시각적 안정성에 중점을 두고 있으며 아래와 같은 메트릭이 포함한다. 각각의 지표에 대한 자세한 사항은 추후에 기술하겠다.


- ***LCP(Largest Contentful Paint, 최대 콘텐츠풀 페인트): 로딩 성능을 측정한다. 최적화된 사용자 경험을 제공하기 위해서는 페이지가 처음으로 로딩된 후 2.5초 이내에 LCP 가 발생해야 한다.***
- ***FID(First Input Delay, 최초 입력 지연): 상호 작용을 측정한다. 우수한 사용자 경험 제공을 위해 페이지의 FID가 100밀리초 이하여야 한다.***
- ***CLS(Cumulative Layout Shift, 누적 레이아웃 시프트): 시각적 안정성을 측정한다. 우수한 사용자 경험을 제공하려면 페이지에서 0.1 이하의 CLS를 유지해야 한다.***


<br>
<br>


## 코어 웹 바이탈 측정 기준

코어 웹 바이탈을 측정하는 도구는 위 세가지 메트릭이 모두 75번째 백분위수인지를 기준으로 권장 목표에 도달했는지 확인하고 있다.

> ***백분위수 선택***

사이트 방문의 대부분이 목표 수준의 성능을 도달했는지를 확인하기 위한 기준으로 75번째 백분위수를 확인하고 있다. <br> 페이지 조회수의 75% 이상이 양호한지, 나쁜지 등의 기준값으로 사용한다는 의미이다. <br> 구글은 75번째 백분위수가 합리적인 균형을 이룬다고 판단했다. 만약 95번째 백분위수를 사용한다면 목표 수준의 성능을 제공했는지 보다 더 확실하게 알 수 있겠지만, 이렇게 되면 중앙값에서 떨어져있는 다른 값들에 영향을 미칠 수 있다. <br> 만약 페이지 조회수 총 100개에 대해 95번째 백분위수를 기준점으로 잡는다면 이상값의 영향을 받는 샘플은 5개 뿐이기 때문이다.


참고: [Core Web Vitals 메트릭 및 임계값 정의](https://web.dev/defining-core-web-vitals-thresholds/)


<br>
<br>


## 코어 웹 바이탈 측정 도구

코어 웹 바이탈을 측정하는 도구에는 Lighthouse, PageSpeed, Insights, Chrome DevTools, Search Console, web.dev의 측정 도구, Web Vitals Chrome 확장 프로그램 및 새로운 Chrome UX Report API가 포함된다.


위 측정 도구들로 우리 프로덕트의 메트릭을 측정해보았다.
<br>
<br>


### Lighthouse
성능 점수를 측정할 수 있는 도구이다. LCP, CLS 를 측정할 수 있으며, Lighthouse에서 측정하고 있는 메트릭 중 Total Blocking Time(총 차단 시간, TBT)은 코어 웹 바이탈 메트릭 중 FID와 관련성이 크다.




[![lighthouse](/assets/images/lighthouse-2022-10-20.png)](/assets/images/lighthouse-2022-10-20.png)



<br>
<br>

### PageSpeed Insights
모바일 및 데스크톱 장치 모두에서의 성능을 측정한다. LCP, CLS, FID를 측정할 수 있으며, 코어 웹 바이탈 평가 합격 및 불합격 여부를 표시한다.




[![pagespeed insights](/assets/images/page-speed-insights-desktop-2022-10-20.png)](/assets/images/page-speed-insights-desktop-2022-10-20.png)<br>
[![pagespeed insights](/assets/images/page-speed-insights-mobile-2022-10-20.png)](/assets/images/page-speed-insights-mobile-2022-10-20.png)



<br>
<br>



### Chrome DevTools
Performance 패널의 Experience 섹션으로 CLS 등 불안정한 시각적 문제를 발견할 수 있다. 또한, 퍼포먼스 패널 하단의 Total Blocking Time 을 통해 FID를 파악하는 용도로 사용할 수 있다.




[![chrome devtools](/assets/images/performance-2022-10-20.png)](/assets/images/performance-2022-10-20.png)




<br>
<br>


### Search Console
사이트 소유자에게 주의가 필요한 페이지 그룹에 대한 개요를 제공한다.




[![search console](/assets/images/search-console-2022-10-20.png)](/assets/images/search-console-2022-10-20.png)




<br>
<br>


### web.dev
시간 경과에 따른 페이지 성능을 측정할 수 있다. 마찬가지로 코어 웹 바이탈 메트릭을 지원하고 있다.




[![web.dev](/assets/images/web.dev-2022-10-20.png)](/assets/images/web.dev-2022-10-20.png)



<br>
<br>


### Web Vitals 확장 프로그램
크롬 웹 스토어에서 설치하여 사용할 수 있는 웹 바이탈 확장 프로그램은 LCP, FID, CLS 메트릭을 실시간으로 측정하여 보여준다.




[![web vitals extension](/assets/images/web-vitals-extension-2022-10-20.png)](/assets/images/web-vitals-extension-2022-10-20.png)
<br>
<br>
> 참고: [Web Vitals](https://web.dev/vitals/),
[Core Web Vitals 측정 도구](https://web.dev/vitals-tools/) <br>

이외에도 [web-vitals](https://github.com/GoogleChrome/web-vitals)라는 라이브러리를 사용하여 코어 웹 바이탈을 측정하는 방법도 있었다.


<br>
<br>


### 기타 Web Vitals
코어 웹 바이탈에서 포함하는 CLS, FID, LCP 메트릭 이외에도 다른 중요한 메트릭이 있다.
- TTFB(Time to First Byte, 최초 바이트까지의 시간)
- FCP(First Contentful Paint, 최초 콘텐츠풀 페인트)
- TTI(Time to Interactive, 상호 작용까지의 시간)

우리는 코어 웹 바이탈을 개선할 목적이므로 기타 웹 바이탈에 대한 논의는 제외하도록 하겠다.

<br>
<br>


## 정리
구글에서 정의한 코어 웹 바이탈(CLS, LCP, FID) 메트릭을 사용자 경험을 개선할 수 있는 중요한 지표로 삼을 수 있다. <br>
각 메트릭을 직접 측정해보면서 우리 페이지에서 개선해야할 메트릭을 확인할 수도 있었다. <br>
앞으로는 CLS, LCP, FID 메트릭에 대해 더욱 자세히 알아보고, 이해한 것을 바탕으로 실제로 프로젝트를 최적화하는 내용까지 다뤄보려고 한다!
