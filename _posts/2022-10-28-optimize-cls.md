---
title:  "[코어 웹 바이탈 최적화] 프로젝트에서 CLS 개선하기"
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

## CLS 메트릭에 좋지 않은 징조가 포착되었다. <br>
크롬 개발자 도구의 Performance 패널에서 확인해보면 Experience 섹션이 생기면서 Layout Shift 레이아웃 이동이 발생하는 구간에 빨간색 블록이 생겼다. <br><br>

![[layout shift](/assets/images/performance-2022-10-20.png)](/assets/images/performance-2022-10-20.png)


빨간색 블록을 클릭해보면 관련있는 요소를 알려준다. <br>
네비게이션에서 보여주는 메뉴 때문에 레이아웃 이동이 발생하는 것을 확인할 수 있었다. <br>

또한, 직접 사이트를 이용할때 육안으로 보더라도 렌더링이 완료되었을때 메뉴 옆의 화살표 이미지가 자리를 이동하면서 부자연스럽게 레이아웃이 이동되는 것을 확인할 수 있었다. <br>

### 이미지 리소스에 기본값을 설정해주자
해당 화살표 이미지는 path와 스크롤 여부에 따라서 흰색 혹은 검은색 화살표로 렌더링 되어야 했다. <br>
따라서, `useState` 를 사용해서 path와 스크롤 여부에 따라서 해당 화살표 이미지의 src를 업데이트해주었다. <br>
이때 화살표 이미지의 초깃값을 빈 문자열로 설정한 것이 원인이었다. <br>

```javascript
  const [caretSrc, setCaretSrc] = useState('');
```

이미지의 src가 초기에는 빈 문자열로 설정되었다가 렌더링이 완료된 후 path 및 스크롤 여부에 따라서 src가 변경되면서 화면이 버벅이는 것처럼 보였던 것이다. <br>

그래서 lazy initialization으로 화살표 이미지의 초깃값으로 하얀색 화살표를 가져올 수 있도록 설정해주었다. <br>

```javascript
  const [caretSrc, setCaretSrc] = useState(() => {
    return chooseCaretSrc(asPath, false);
  });
```

useState를 사용할때 initialState를 인자로 넘기면 초기 렌더링 시에만 사용하는 state를 설정할 수 있다. <br>
위와 같이 초기 렌더링 시에만 실행될 수 있는 함수를 제공하면 된다.
<br>

이렇게 초깃값을 제대로 설정해주고 나니까 실제로 사이트를 이용하면서도 메뉴바가 덜컹거리는 움직임이 없었고, 개발자 도구 Performance 패널에서도 빨간 블록이 사라졌다! <br>

![[cls-improved](/assets/images/cls-improved-1-2022-10-28.png)](/assets/images/cls-improved-1-2022-10-28.png)

### 프리 로드를 통한 FOUT 개선
우리 프로젝트는 구글 웹 폰트를 사용하고 있다.  <br>
웹 폰트 다운로드가 완료되기 전까지는 기본 글꼴을 보여주다가 다운로드를 마치면 지정된 웹 폰트로 글꼴이 바뀌면서 글자가 반짝거려 보이는 FOUT 현상이 발생했다. <br>

따라서, 아래와 같이 글꼴을 프리로드 할 수 있도록 변경해주었다. <br>
프리로드 속성은 CRP 초기에 폰트를 다운로드하기 때문에 글자의 반짝거림 없이 사용자에게 보다 안정적인 웹 폰트를 제공할 수 있게 되었다.

```javascript
<link
  rel="preload"
  href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR&display=swap"
  as="font"
  type="font/woff2"
  crossorigin
/>
```

