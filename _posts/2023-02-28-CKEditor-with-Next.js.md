---
title:  "Next.js에서 CKEditor5 사용하기"
categories: 
  - Blog
tags:
  - CKEditor5
  - Next.js
  - Global CSS cannot be imported from within node_modules.
  - Online Builder
  - React-Quill
toc: true
toc_sticky: true
toc_label: "Next.js와 CKEditor5 🫥"
share: true
published: false,

---

호옥시 Next.js에 CKEditor를 설치하시다가 고통 받고 들어오셨나요 !!  
저도 그 마음을 잘 압니다 😇   
썰을 풀어보겠읍니다. . .

---

## 왜 CKEditor를 선택했는가?

### 시작에는 `React-Quill` 이 있었다
우리 프로젝트는 Next.js를 기반으로 한다.  
CMS를 개발하면서 에디터가 필요해서 [`React-Quill(이하 리액트 퀼)`](https://github.com/zenoamaro/react-quill) 을 설치했었다.    

간단한 글 작성 및 편집을 할 수 있으면서 Next.js와의 호환성도 좋고, 타입스크립트도 지원되어서 아주 편리하게 사용 중이었다.  

이후 CMS 고도화가 필요해지면서 에디터에 조금 더 풍부한 기능들이 요구되었다.  
예를 들어서, 구분 선을 보여준다거나 테이블, 이미지 업로드 시 자동으로 캡션이 추가되는 기능 등등이었다.  

처음에는 객기(?)로 모든 기능들을 커스터마이징하려고 했다. 🐶  
그러다 보니 에디터 개발 그 자체가 태스크가 되어버린 듯했다.  


사실 이번 스프린트의 목적은 개발자를 통해서 JSON으로 관리되던 콘텐츠를, 개발자가 아닌 팀원분들도 에디터를 통해서 발행하고, 발행된 콘텐츠를 우리 디자인 시스템에 맞게 스타일을 적용하는 것이었다.   
에디터를 잘 만드는 것도 중요했지만, 에디터 자체를 개발하는 것이 스프린트의 주된 목적은 아니었다.  


### `라이브러리를 라이브러리답게 사용하자`

리액트 퀼을 커스터마이징하기 위해 사투를 벌이던 중에  
`라이브러리를 라이브러리답게 사용하자` 라는 짱짱 팀원분의 이야기를 듣고 '라이브러리를 사용하는 목적은 이미 잘 만들어진 소프트웨어를 편리하게 사용하는 것에 있다'고 다시금 인지했다.  


이에 따라 에디터와 관련한 요구 사항을 가장 빠르고 편리하게 구현할 수 있는 다른 라이브러리를 찾기 시작했다.  

### 검증 과정

CKEditor는 홈페이지에서 강조하는 것처럼 리치한 에디터이다.  
데모를 사용해보며 우리에게 필요한 기능들을 손쉽게 구현할 수 있다고 생각했다.  

또한, `Snyk Open Source Advisor`에서 확인한 종합 점수도 꽤 괜찮은 편이라서 어느 정도 검증받은 라이브러리라고 생각했다.  
[snyk Advisor: CKEditor5](https://snyk.io/advisor/npm-package/ckeditor5)  


## 이슈 1: Global CSS cannot be imported from within node_modules.

(두괄식 커뮤니케이션이 좋다고 해서 이제 정말 시작을 해볼게요 😇)

CKEditor는 Next.js를 공식적으로 지원하고 있지는 않다.  
만약 npm으로 CKEditor 패키지를 설치하고, 플러그인을 사용하려고 하면 이런 이슈가 발생할 수 있다.  

(나는 구분선을 추가하기 위해 `@ckeditor/ckeditor5-horizontal-line` 라는 패키지를 설치하고, 임포트하기만 했는데도 이러한 이슈가 발생했다.)

`Global CSS cannot be imported from within node_modules.`  

![image](https://user-images.githubusercontent.com/84058944/221815221-f85ff9a3-542e-4cf2-85df-d9ff5890f964.png)  


[`Read more`](https://nextjs.org/docs/messages/css-npm) 를 클릭해보면 메인테이너에게 요청해보라는 해결책중 하나로 제시된다 ㄴㅇㄱ (대충 '상상도 못한' 짤을 형상화)  


<br><br>

해당 이슈는 Next.js 레포에도 버그로 리프팅이 되어있다: 
[Global CSS cannot be imported from within node_modules.](https://github.com/vercel/next.js/issues/19936#issue-758781682)  

**Next.js는 node_modules를 더 이상 컴파일하지 않기 때문에 node_modules에서 전역 css 코드를 가져올 수 없다는 것이 메인테이너의 답변이었다.**  

<br>

가장 최근에 업데이트된 답변은 Next.js 버전 13의 릴리즈와 함께 `app` 디렉토리를 사용하는 것이었다: 
[이슈 관련 메인테이너의 최신 업데이트](https://github.com/vercel/next.js/discussions/27953#discussioncomment-3978605)  

<br>


`app` 디렉토리는 베타 버전인데다가 우리 프로젝트는 아직 Next.js를 버전 13으로 업그레이드하지 않은 상황이라서 확실한 해결책이 되지는 않았다.  

### 이슈 1 해결: Online Builder를 사용하자

울며 겨자 먹기로 CKEditor의 공식 문서의 React 파트를 다시 한번 찬찬히 살펴 보았다.  
나에게 필요한건 기본 클래식 에디터에 내가 원하는 툴바 옵션을 추가하는 것이었다. (구분선, 테이블 등등)  

툴바를 커스텀하기 위해서 [Online Builder(이하 온라인 빌더)](https://ckeditor.com/ckeditor-5/online-builder/)를 사용할 수 있다는 것을 확인했다.  
내가 원하는 플러그인을 모두 선택한 후에 패키지를 npm으로 설치하는 것이 아니라, zip파일을 직접 다운로드하고 프로젝트로 옮겨서 설치하는 방식이었다.  

나는 아주 의욕적으로(복선) 거의 모든 플러그인을 선택한 채로 zip 파일을 내려 받았다. 😶‍🌫️   
플러그인을 임포트하면 위 오류가 발생하니까 더 추가할 플러그인이 없을 정도로 꽉꽉 채워서 빌드된 에디터를 설치하고 싶었기 때문이다 👉👈

그리고, 새로운 이슈를 만났다.  

## 이슈 2: `yarn add` 가 되지 않는다!
CKEditor의 공식 문서에 따라서 진행했다.  

1. 내려 받은 폴더를 `src`와 같은 위계상에 둔다.  

예시 이미지: <img width="259" alt="image" src="https://user-images.githubusercontent.com/84058944/221824618-25298b39-1b63-46ce-8198-efe7ee41f4b4.png">

<br>

2. 다운받은 패키지를 `yarn add` 명령어로 프로젝트의 종속성으로 추가한다.  

공식 문서에서는 아래와 같이 `상대 경로`를 알려준다.  

```
yarn add file:./ckeditor5
``` 

터미널에 그대로 복붙한 결과는 아래와 같았다.  

![image](https://user-images.githubusercontent.com/84058944/221827163-5ebe2e4d-7ced-44e3-87c3-a6af983ef0b9.png)  

😇

저 상태에서 터미널에 `cd ./ckeditor5` 를 입력하면 해당 폴더로 정상적으로 이동되었다.  
뭐가 문제인지 한참 고민했었다.  

### 이슈 2 해결: 절대 경로로 설치하기
팀원분께서 절대 경로로 설치해보라고 알려주셨다!  
`pwd` 명령어로 현재 작업중인 디렉토리의 경로를 알아낸 후 뒤에 `ckeditor5`를 붙여주었다.  

```
yarn add pwd로 알아낸 절대 경로/ckeditor5
```

요렇게 하니까 드디어 설치가 되었다 두둥 ... !!  

이 이후에는 `yarn add file:./ckeditor5`, `yarn add ./ckeditor5` 를 실행했을 때도 정상적으로 설치되었다.  

(혹시 빌드 파일 다운로드 이후에 저처럼 설치에서 막히셨던 분들 꼬옥 절대 경로로 시도해보세요!)  

제대로 설치가 되었다면 `package.json` 에서 아래와 같이 `ckeditor5-custom-build`를 확인할 수 있다.

<img width="639" alt="image" src="https://user-images.githubusercontent.com/84058944/221833902-80544e45-f759-4cbb-a2ed-d3ae48189e5a.png">



그리고 설치한 에디터를 임포트하면서 타입스크립트 관련 새로운 이슈를 맞닥뜨렸다. (두둥)   

## 이슈 3: 설치된 파일이 있었는데 파일이 없었습니다 Could not find a declaration file for module '@ckeditor/ckeditor5-react'.
야심차게 에디터 관련 코드를 임포트했다.  

```
import { CKEditor } from '@ckeditor/ckeditor5-react';
```

바로 TS 컴파일 오류가 발생했다.    

`TS7016: Could not find a declaration file for module '@ckeditor/ckeditor5-react'. '작업 경로/.yarn/unplugged/@ckeditor-ckeditor5-react-virtual-bd87788f77/node_modules/@ckeditor/ckeditor5-react/dist/ckeditor.js' implicitly has an 'any' type.   Try `npm i --save-dev @types/ckeditor__ckeditor5-react` if it exists or add a new declaration (.d.ts) file containing `declare module '@ckeditor/ckeditor5-react';`

우여곡절 끝에 파일을 설치해주었는데, 해당 모듈을 찾지 못하는 것 같았다. 🤦🏻‍♀️  
(CKEditor5에서 타입스크립트를 공식적으로 지원하지 않아서 발생하는 이슈같다.)  

### 이슈 3 해결: `declare module`

그치만 친절하게 해결책도 알려주었다.  
`if it exists or add a new declaration (.d.ts) file containing `declare module '@ckeditor/ckeditor5-react'`   



CKEditor5 레포에서도 이와 같은 방법이 제안되었다: [react, typescript, ckeditor-duplicated-modules](https://github.com/ckeditor/ckeditor5-react/issues/140#issuecomment-655453105)  

요렇게 해주면 된다!  <br>
<img width="876" alt="image" src="https://user-images.githubusercontent.com/84058944/221833001-5264284b-235f-44bc-bd0b-0a1604551cd3.png">

<br>

TS 오류가 말끔히 사라졌다!  

## 이슈 4: 빌드 파일이 정상적으로 실행되지 않는다. Cannot read properties of null (reading 'model') 

이젠 에디터 구경이라도 할 수 있을 줄 알고 희망차게 에디터를 리턴하는 코드를 작성하고, 로컬 환경에서 실행해 보았다.  

예시 코드

- 동적으로 임포트한 에디터를 리턴하는 컴포넌트

```
// 에디터가 SSR을 지원하지 않기 때문에 dynamic import의 옵션을 `ssr:false`로 설정한다.  
// 서버 사이드에서 해당 모듈이 포함되지 않도록 하기 위함이다.  

import { useState, useEffect } from 'react';
import dynamic from 'next/dynamic';

const Test = () => {
  const DynamicEditor = dynamic(() => import('../components/EditorTest'), {
    ssr: false,
  });

  const [isEditorReady, setIsEditorReady] = useState(false);

  useEffect(() => {
    setIsEditorReady(true);
  }, []);

  return <>{isEditorReady ? <DynamicEditor isEditorReady={isEditorReady} /> : <p>로딩중입니다...</p>}</>;
};

export default Test;
```

- CKEditor를 임포트하는 컴포넌트


```
// 바로 아래 라인을 봐주세요! 여기에서 `yarn` 명령어로 설치한 에디터를 임포트하시면 됩니다!
import Editor from 'ckeditor5-custom-build/build/ckeditor';
import { CKEditor } from '@ckeditor/ckeditor5-react';

const EditorTest = ({ isEditorReady }) => {
  return <>{isEditorReady && <CKEditor data={'test'} editor={Editor} />}</>;
};

export default EditorTest;
```


이때 브라우저에서 확인되는 오류는 아래와 같았다.  

```
Unhandled Runtime Error
TypeError: Cannot read properties of null (reading 'model')

Call Stack
eval
node_modules/@ckeditor/ckeditor5-react/dist/ckeditor.js (5:21743)
```

느낌상 우리가 설치해준 빌드 파일이 정상적으로 읽혀지지 않는 것 같았다.    
찬물을 좀 마시고 공식 문서를 다시 찬찬히 훑어 보았다.    

> If you want to use the CKEditor 5 online builder, make sure that the watchdog feature is not selected. The React integration comes with the watchdog feature already integrated into the core.

[Using the CKEditor 5 online builder](https://ckeditor.com/docs/ckeditor5/latest/installation/frameworks/react.html#using-the-ckeditor-5-online-builder)  

리액트 인터그레이션은 이미 `watchdog` 기능이 제공되기 때문에 온라인 빌드시 `watchdog` 기능이 선택되지 않도록 하라는 내용이 문득 눈에 들어왔다.  
그리고 내가 아주 의욕적으로 최대한 많은 플러그인을 추가한 파일을 다운로드 받았다는게 생각났다.  

아래 경로에서 내가 선택한 플러그인 옵션을 확인할 수 있다.  
`./ckeditor/src/ckeditor.js`  

아니나 다를까 `watchdog` 이라는 옵션이 포함되어 있었다 .. 😗  

### 이슈 4 해결: `watchdog` 옵션 제거
`watchdog` 옵션을 포함하지 않은 채로 빌드 파일을 다시 다운로드 받아서 프로젝트에 설치해주었다.   
여기까지 하니까 드디어! 로컬 환경에서 CKEditor가 정상적으로 실행되었다. (짝짝짝 👏👏👏)  
