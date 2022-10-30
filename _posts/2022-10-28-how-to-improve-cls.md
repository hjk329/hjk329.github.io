---
title:  "[코어 웹 바이탈 최적화] CLS 최적화하기"
categories: 
  - 최적화
tags:
  - 리팩토링
  - 최적화
  - 코어 웹 바이탈
  - CLS
toc: true
toc_sticky: true
toc_label: "CLS 최적화하기"
share: true
---

CLS는 뷰포트 내에서 발생한 예기치 않은 레이아웃 이동의 영향분율과 거리율분을 곱한 값이다. <br>

> [[코어 웹 바이탈 최적화] CLS(Cumulative Layout Shift, 누적 레이아웃 이동)란?](http://localhost:4000/%EC%B5%9C%EC%A0%81%ED%99%94/CLS/)

CLS 점수가 열악한 이유는 대개 아래와 같다.
- 크기가 정해지지 않은 이미지
- 크기가 정해지지 않은 광고, 임베드 및 iframe
- 동적으로 주입된 콘텐츠
- FOIT/FOUT을 유발하는 웹 폰트
- DOM을 업데이트하기 전에 네트워크 응답을 대기하는 작업

<br>
각 원인들을 자세히 알아보겠다.

## 크기가 정해지지 않은 이미지
이미지나 비디오 요소에 항상 `width` 및 `height` 크기 속성을 포함하자. <br>
또는 CSS 종횡비 상자를 사용하여 필요한 공간을 확보해 두는 방법도 있다. <br>

나도 이전에 개인적으로 프로젝트를 하면서 RatioBox 라는 컴포넌트를 만들어서 사용한 적이 있다. <br>
반응형 웹 사이트를 구현할때 이미지나 동영상이 정해진 비율에 맞게 확대 및 축소되기를 원했기 때문이다. <br>
아래는 RatioBox 컴포넌트의 코드 전문이다. <br>

```javascript
import React from 'react';
import styled from 'styled-components';

const RatioBox = ({
  children, width, height, contain,
}) => (
  <Container>
    <Inner>
      {children}
    </Inner>
  </Container>
)

const Container = styled.div`
  position: relative;
  padding-top: ${(p) => (p.width && p.height && p.height / p.width * 100)}%;
`;

const Inner = styled.div`
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #eee;

  img,
  video,
  iframe {
    width: 100%;
    height: 100%;
    object-fit: ${(p) => (p.contain ? 'contain' : 'cover')};
  }
`;
export default RatioBox;
```

<br>
이미지나 비디오 요소의 너비와 높이까지도 정해진 비율에 따라서 크기가 줄어들었다가 커지고, 아직 해당 요소가 렌더링되지 않았다면 배경색을 보여줘서 스켈레톤을 보여줬다. <br>
이렇게 하면 이미지나 비디오 요소가 차지할 영역이 이미 정해져있기 때문에 사용자에게 예기치 않은 레이아웃 이동이 발생하지 않는다.



## 크기가 정해지지 않은 광고, 임베드 및 iframe

### 광고
광고의 크기는 성능과 수익을 높이는 요소이지만, 사용자가 보고 있는 콘텐츠를 아래로 밀어낼 수 있기 때문에 최적의 사용자 경험과는 멀어질 수 있다. <br>
다음을 통해 레이아웃 이동을 경감시킬 수 있다. <br>

- 광고 슬롯을 위한 고정 공간을 확보한다.
  - 즉, 광고 태그 라이브러리가 로드되기 전에 요소의 스타일을 지정한다.
  - 콘텐츠 흐름에 따라 광고를 배치하는 경우 슬롯 크기를 미리 확보하여 이동이 없도록 한다.

- 뷰포트 상단 근처에 비고정 광고를 배치하는 경우 주의한다.
(뷰포트 상단에 비고정 광고가 등장하는 경우 전체적으로 기존 요소들의 시작 위치가 변경되므로 영향분율과 거리율분에 영향이 클 것이다.)

- placeholder를 표시하여 광고 슬롯이 표시될때 반환된 광고가 없는 경우 할당된 공간이 축소되지 않도록 한다.

- 광고 슬롯에 대해 가능한 가장 큰 크기를 할당하여 이동이 없도록 한다.

- 과거 데이터를 기반으로 광고 슬롯에 가장 적합한 크기를 선택한다.

### 임베드 및 iframe
임베드는 그 크기를 미리 알지 못할 수 있다. <br>
임베드될 충분한 공간의 크기를 파악해서 placeholder를 제공하는 것이 바람직하다.

### 동적 콘텐츠
동적 콘텐츠가 추가되어야 한다면 placeholder나 스켈레톤 UI 등을 사용해서 로드 시 페이지의 콘텐츠가 크게 이동하지 않도록 하는 것이 좋다. <br>
동적 콘텐츠를 예기치 않은 레이아웃 이동 없이 제공하는 예시로는 현재 보고 있는 상품 페이지에서 더 많은 상품을 불러오기 위해 더 보기 버튼을 클릭하면 더 많은 콘텐츠를 불러오는 것이 있다. <br>

![avoid unexpected layout shift](https://web-dev.imgix.net/image/OcYv93SYnIg1kfTihK6xqRDebvB2/TjsYVkcDf03ZOVCcsizv.png?auto=format&w=845)
***이미지 출처: [누적 레이아웃 이동 최적화](https://web.dev/i18n/ko/optimize-cls/#%ED%81%AC%EA%B8%B0%EA%B0%80-%EC%A0%95%ED%95%B4%EC%A7%80%EC%A7%80-%EC%95%8A%EC%9D%80-%EA%B4%91%EA%B3%A0,-%EC%9E%84%EB%B2%A0%EB%93%9C-%EB%B0%8F-iframe-%F0%9F%93%A2%F0%9F%98%B1)***


### FOUT/FOIT를 유발하는 웹 폰트
- FOUT(Flash Of Unstyled Text): 브라우저가 웹 폰트를 다운로드하기 전까지는 대체 글꼴로 렌더링되다가 해당 웹 폰트를 다운로드한 후에 대체해서 보여주는 현상. 글자가 깜빡거리는 느낌이다.
- FOIT(Flash Of Invisible Text): 브라우저가 웹 폰트를 다운로드하기 전까지 텍스트가 보이지 않는 현상

FOUT, FOIT 모두 웹 폰트가 다운로드되고 렌더링되면서 기존의 레이아웃을 이동시킬 수 있기 때문에 최소화하는 것이 바람직하다. <br>

- `font-display`를 사용하면 `auto`, `swap`, `block`, `fallback`, `optional` 같은 값으로 커스텀 글꼴의 렌더링 동작을 수정할 수 있다. 단, `optional` 이외의 모든 값은 레이아웃 이동을 발생시킬 수 있다.
- 핵심 웹 폰트에 `<link rel=preload>` 사용: CSSOM이 생성될 때까지 기다릴 필요 없이 CRP 초기에 웹 폰트에 대한 요청이 트리거되어 웹 폰트를 미리 로드하므로 레이아웃 이동이 발생하지 않는다.

### 애니메이션
레이아웃 이동을 유발하는 애니메이션보다 `transform` 애니메이션을 권장한다.
특정 CSS 속성은 레이아웃을 트리거한다. 즉, 레이아웃을 재구성하고 페인트 및 합성을 해야하기 때문에 비용이 많이 들기도 하고, 예기치 않은 레이아웃 이동을 유발할 수도 있다. <br>
아래 링크를 통해 레이아웃을 트리거하는 CSS 속성을 확인할 수 있다.
[csstriggers](https://csstriggers.com/)

---
참고  <br>
[누적 레이아웃 이동 최적화](https://web.dev/i18n/ko/optimize-cls/)


