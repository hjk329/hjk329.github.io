---
title:  "[코어 웹 바이탈 최적화] 프로젝트에서 LCP 개선하기"
categories: 
  - 최적화
tags:
  - 리팩토링
  - 최적화
  - 코어 웹 바이탈
  - LCP
toc: true
toc_sticky: true
toc_label: "프로젝트에서 LCP 개선하기"
share: true
gallery:
  - url: /assets/images/lcp-bad-2022-10-25.png
    image_path: /assets/images/lcp-bad-2022-10-25.png
    alt: 'lcp-bad'
  - url: /assets/images/lcp-improved-2022-10-25.png
    image_path: /assets/images/lcp-improved-2022-10-25.png
    alt: 'lcp-improved'
---

이전 포스팅을 통해 LCP의 의미와 측정 방법, 최적화 방안에 대해 알아 보았다. <br>

> [[코어 웹 바이탈 최적화] LCP(Largest Contentful Paint, 최대 콘텐츠풀 페인트)](https://hjk329.github.io/%EC%B5%9C%EC%A0%81%ED%99%94/lcp/)

> [[코어 웹 바이탈 최적화] LCP 최적화하기](https://hjk329.github.io/%EC%B5%9C%EC%A0%81%ED%99%94/improve-lcp/)

<br>
우리 프로덕트에서 LCP 점수를 개선한 사례를 공유하려 한다. <br>

우리 프로덕트 메인 페이지의 LCP는 최상단에서 보여주는 이미지이다. <br>

LCP 향상을 위해 활용한 전략은 아래와 같다.
- ### 이미지를 압축해서 최적화된 이미지를 전달하자.
- ### CDN을 이용해서 사용자에게 물리적으로 더 가깝게 캐시하자.

 <br>
초기에는 로컬에서 저장된 이미지를 불러왔다. 라이트하우스로 확인해보았을때 PC 기준으로 LCP 소요 시간은 6.3초로 양호한 LCP 기준인 2.5초보다 훨씬 오래 걸렸다. <br>
이미지를 압축하고, S3 버킷에서 정적 리소스를 사용하도록 변경한 이후에는 LCP 소요 시간은 1.8초가 되었다. <br>
 
라이트 하우스의 성능 점수는 각 지표별로 가중치가 있는데 LCP는 가중치가 꽤 큰 메트릭이라서 여기에서 낮은 점수를 받으니 전체적인 성능에서도 낮은 점수를 받을 수 있다. <br>
 
{% include gallery id="gallery" caption="왼쪽: LCP 개선 이전, 오른쪽: LCP 개선 후" %}
