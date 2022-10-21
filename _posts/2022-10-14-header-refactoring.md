---
title:  "스크롤값에 따라 계속해서 리렌더링되는 헤더 컴포넌트 리팩토링하기"
categories: 
  - React
tags:
  - React.memo
  - 리액트 메모
  - 리렌더링 방지
  - 리팩토링
toc: true
toc_sticky: true
toc_label: "헤더 컴포넌트 리팩토링"
share: true

---


## 리팩토링을 하게 된 배경
프로젝트를 진행하며 헤더에 대한 디자인 요구 사항은 아래와 같았다.
- 헤더의 기본적인 디자인은
	- 로고 이미지는 하얀색이다.
	- 헤더 전체의 배경색은 투명색이다.
	- 헤더 내부의 글자는 하얀색이다.
	- 헤더 내부의 caret(화살표) 이미지는 하얀색이다.
	
- 하지만, 1px 이라도 스크롤이 되거나, 혹은 특정 path 에서의 헤더의 디자인은
	- 로고 이미지는 컬러이다.
	- 헤더 전체의 배경색은 하얀색이다.
	- 헤더 내부의 글자는 검은색이다.
	- 헤더 내부의 caret(화살표) 이미지는 검은색이다.

위의 요구 사항을 해결하기 위해 헤더 컴포넌트는 다양한 로직을 가지고 있었다.
- 브라우저의 스크롤Y값이 1보다 큰지를 확인하는 훅을 불러온다. (전달한 props 만큼,예를 들어 1px 스크롤 되었는지의 여부를 boolean 으로 리턴하는 커스텀 훅을 생성했다.)
- 지금 위치한 path 를 알아야 한다.

위 두가지 결과값에 따라서 로고 이미지, caret 이미지의 src 및 헤더의 배경색 및 글자색이 변경되도록 구현했다.

결과적으로 디자인 요구사항대로 동작은 했지만, react-dev-tools 로 확인해보았을때 스크롤을 내릴때마다 헤더가 계속해서 리렌더되는 점을 확인했다.

불필요한 리렌더링을 방지하기 위해 리팩토링을 결정했고, 헤더가 리렌더링 되는 데에는 두 가지 문제점이 있었다.
1. 스크롤값을 확인하는 커스텀 훅이 계속해서 상태를 업데이트하면서 리렌더링을 발생시키고 있었다.
2. 헤더는 그냥 뷰만 보여주는 컴포넌트가 되어야 했는데 너무 많은 관심사를 가지고 있었다.


## 리팩토링 과정
1. 스크롤값을 확인하는 커스텀 훅이 계속해서 상태를 업데이트하면서 리렌더링을 발생시키고 있었다.

=> 헤더에서 필요하지 않은 상태값이 리렌더링을 유발하고 있었다. 따라서 해당 훅을 두 개의 훅으로 분리해주었다.

스크롤 값을 확인하기 위해 생성했던 커스텀 훅은 아래와 같다.

```
import { useEffect, useState } from 'react';
import { throttle } from 'throttle-debounce';

interface useDetectScrollReturn {
  scrolled: boolean;
  scrollHeight: number;
}

export const useDetectScroll = (detectScrollTop: number, throttleDuration?: number): useDetectScrollReturn => {
  const [scrolled, setScrolled] = useState(false);
  const [scrollHeight, setScrollHeight] = useState(0);

  useEffect(() => {
    const handler = throttle(Number(throttleDuration), () => {
      const sct = window.scrollY || window.pageYOffset;
      setScrollHeight(sct);
      if (sct > detectScrollTop) {
        setScrolled(true);
      } else {
        setScrolled(false);
      }
    });
    window.addEventListener('scroll', handler);
    return (): void => {
      window.removeEventListener('scroll', handler);
    };
  }, []);
  return { scrolled, scrollHeight };
};
```

해당 훅은 detectScrollTop 에 해당하는 값만큼 스크롤 되었는지의 여부와, 스크롤된 값을 리턴하고 있었다.
사실 처음에는 scrolled 만을 리턴하는 훅이었는데 다른 컴포넌트에서 스크롤값이 필요해져서 관련 코드를 추가해준 것이었다.

스크롤됨에따라서 scrollHeight 는 계속해서 업데이트되기 때문에 상태가 바뀌면 당연히 리렌더링될 수 밖에 없었다.
따라서, 해당 훅을 아래와 같이 각각의 역할을 할 수 있도록 분리해주었다.

```
import { useEffect, useState } from 'react';
import { throttle } from 'throttle-debounce';

interface useDetectIsScrolledReturn {
  scrolled: boolean;
}

export const useDetectIsScrolled = (detectScrollTop: number, throttleDuration?: number): useDetectIsScrolledReturn => {
  const [scrolled, setScrolled] = useState(false);

  useEffect(() => {
    const handler = throttle(Number(throttleDuration), () => {
      const sct = window.scrollY || window.pageYOffset;
      if (sct > detectScrollTop) {
        setScrolled(true);
      } else {
        setScrolled(false);
      }
    });
    window.addEventListener('scroll', handler);
    return (): void => {
      window.removeEventListener('scroll', handler);
    };
  }, []);
  return { scrolled };
};
```

```
import { useEffect, useState } from 'react';
import { throttle } from 'throttle-debounce';

interface useDetectScrollHeightReturn {
  scrollHeight: number;
}

export const useDetectScrollHeight = (detectScrollTop: number, throttleDuration?: number): useDetectScrollHeightReturn => {
  const [scrollHeight, setScrollHeight] = useState(0);

  useEffect(() => {
    const handler = throttle(Number(throttleDuration), () => {
      const sct = window.scrollY || window.pageYOffset;
      setScrollHeight(sct);
    });
    window.addEventListener('scroll', handler);
    return (): void => {
      window.removeEventListener('scroll', handler);
    };
  }, []);
  return { scrollHeight };
};
```

2. 헤더는 그냥 뷰만 보여주는 컴포넌트가 되어야 했는데 너무 많은 관심사를 가지고 있었다.

=> 기존의 헤더 컴포넌트는 프레젠테이션 역할만 담당할 수 있도록 수정했다.
스크롤 여부, path 확인 등의 로직과 관련한 부분은 HeaderContainer 를 생성해서 해당 컴포넌트에서 헤더 컴포넌트로 props 로 주입해주는 방식으로 수정했다.

이외에도 컴포넌트의 props가 변경되었을때만 리렌더링해주도록 하는 방법이 없을까? 생각하다가 React.memo 를 공부해 보았다.

## React.memo
React.memo 는 리액트에서 제공하는 API 로 HOC 의 일종이다. 
React.memo 를 사용하는 컴포넌트는 props 의 값이 변경될때만 리렌더링 된다.

> 컴퍼넌트가 `React.memo()`로 래핑 될 때, React는 컴퍼넌트를 렌더링하고 결과를 메모이징(Memoizing)한다. 그리고 다음 렌더링이 일어날 때 `props`가 같다면, React는 메모이징(Memoizing)된 내용을 재사용한다.
> 출처: https://ui.toast.com/weekly-pick/ko_20190731

React.memo 는 props 가 변경된 사실을 어떻게 알아챌까?

> props가 갖는 복잡한 객체에 대하여 얕은 비교만을 수행하는 것이 기본 동작입니다. 다른 비교 동작을 원한다면, 두 번째 인자로 별도의 비교 함수를 제공하면 됩니다.
> 출처: https://ko.reactjs.org/docs/react-api.html#reactmemo

즉, 넘겨받는 props 가 원시 타입이라면 '값'이 같은지까지 비교한다. 하지만, 넘겨받는 props 가 참조 타입 (예를 들어 객체) 이라면 해당 객체의 참조 주소가 같은지만을 확인한다.

따라서, 객체 안의 값이 변하지 않고 동일함에도 불구하고 참조 주소가 다르다면 리렌더링을 발생시킬 수 있다.
리액트는 리렌더링될때마다 객체가 새로 생성되고, 이에 따라 참조 주소가 달라질 것이기 때문에 얕은 비교를 해주는 React.memo 를 사용한다면 객체 안의 값이 같더라도 계속해서 리렌더링이 발생할 수 있다.
이렇게 되면 React.memo 를 사용하는 이점을 제대로 누리지 못할 수 있다.
React.memo 의 얕은 비교 방식을 선택하지 않고, 커스텀한 비교 방식을 설정해주기 위해서 두번째 인자로 props 로 넘겨받는 객체의 프로퍼티를 비교해주는 함수를 추가할 수 있다.

```
function MyComponent(props) {
  /* props를 사용하여 렌더링 */
}
function areEqual(prevProps, nextProps) {
  /*
  nextProps가 prevProps와 동일한 값을 가지면 true를 반환하고, 그렇지 않다면 false를 반환
  */
}
export default React.memo(MyComponent, areEqual);
```

이와 관련해서 고민해본 내용은 아래와 같다.
해당 프로젝트에서 헤더 컴포넌트의 자식 컴포넌트중에서 item 이라는 객체와, caretSrc 라는 string 을 props 로 넘겨받는 컴포넌트가 있었다.
리렌더링이 유발되는 조건은 caretSrc 값이 달라지는 것이었다.

처음에는 객체(참조 타입)를 props 로 넘겨받았기 때문에 얕은 비교 방식으로는 리렌더링을 계속 유발할 수 있기 때문에 무조건 비교 함수를 작성해줘야 하는 것으로 생각했다.
하지만, item 이라는 객체는 단순히 메뉴바에 대한 데이터를 가지고 있는 정적인 객체였고, 변경시 리렌더링해줘야하는 조건도 아니었다.

해당 이슈와 관련하여 팀원분과 협의 후에 React.memo 에 대해 아래와 같은 컨벤션을 생성했다.
-   원시값만 props 로 넘겨 받는 컴포넌트에서 React.memo() 를 사용하는 경우 두번째 인자로 비교함수를 전달하지 않아도 된다.
-   참조 타입을 props 로 넘겨받으면서 해당 참조 타입이 리렌더링을 발생시키는 조건이라면 비교 함수를 명시적으로 생성해준다.


## 마무리
React.memo 가 메모이제이션을 위해 props 들을 어떻게 비교해주는지, 어떤 경우에 비교 함수를 생성할 것인지에 대해 생각해 볼 수 있어서 좋았다.