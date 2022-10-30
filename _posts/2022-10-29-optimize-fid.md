---
title:  "[코어 웹 바이탈 최적화] 프로젝트에서 FID 개선하기"
categories: 
  - 최적화
tags:
  - 리팩토링
  - 최적화
  - 코어 웹 바이탈
  - FID
toc: true
toc_sticky: true
toc_label: "FID 개선하기"
share: true

gallery1:
  - url: /assets/images/FID-by-extension-2022-10-29.png
    image_path: /assets/images/FID-by-extension-2022-10-29.png
    alt: 'fid-before'
  - url: /assets/images/FID-improved-by-extension-2022-10-29.png
    image_path: /assets/images/FID-improved-by-extension-2022-10-29.png
    alt: 'fid-improved'
    
gallery2:
  - url: /assets/images/mp4-download-2022-10-29.png
    image_path: /assets/images/mp4-download-2022-10-29.png
    alt: 'mp4'
  - url: /assets/images/webm-download-2022-10-29.png
    image_path: /assets/images/webm-download-2022-10-29.png
    alt: 'webm'
---


## TBT 향상을 통해 FID 개선하기

FID를 개선하는 일은 꽤나 까다롭게 느껴졌다. <br>
코어 웹 바이탈 도큐먼트에서 추천해준 방식은 JS 번들을 여러 청크로 코드 분할하거나, Polyfills를 최소화하는 방법 등이었는데 이에 대한 이해가 선행되어야할 것 같아서 지금 당장 시도해볼 엄두가 나지 않았다. <br>
위 방법은 공부하면서 차차 시도해볼 예정이다. <br> <br>

그렇다면 FID를 개선할 수 있는 다른 방법은 무엇이 있을까 고민하다가 FID가 TBT와 관련이 있다는 것이 생각났다 <br>
TBT(Total Blocking Time, 총 차단 시간) 메트릭은 코어 웹 바이탈에 포함되는 메트릭은 아니지만, 메인 스레드가 입력 응답을 막는 시간과 관련있기 때문에 이를 개선하면 FID도 자연스레 개선할 수 있을 것이라 생각했다. <br>

TBT에 대한 자세한 내용은 여기를 참고해주세요! 
> [Total Blocking Time(총 차단 시간, TBT)](https://web.dev/tbt/)


우선 우리 프로젝트의 TBT는 아래와 같다. <br> <br>
![tbt](/assets/images/tbt-as-is-2022-10-29.png)

총 차단 시간이 330ms로 좋은 TBT 점수로 평가되는 300ms를 조금 넘었다. <br>


### 최적화된 이미지를 제공하자

우리 메인 페이지에서 TBT에 영향을 미치는 요인은 메인 이미지라고 생각했다. <br>
모바일로 웹 페이지에 접속했을때 메인 이미지의 자리가 비어있는데 1-2초 뒤쯤에 이미지 렌더링이 완료되었기 때문이다. <br>

엄밀히 말해서 이미지는 자바스크립트나 CSS 파일처럼 렌더링을 블로킹하는 요소는 아니다. <br>
하지만, 이미지가 로드되기 시작했지만 완료하는 데 시간이 걸려서 사용자에게 제대로된 이미지를 보여주지 못하는 것은 로드 성능에 굉장히 크게 관여한다고 생각했다. <br>

그래서 이미지 최적화를 해보기로 했다. <br>
현재 이미지는 CDN을 이용해서 S3에 저장된 정적 이미지를 가져오고 있다. <br>
해당 이미지 파일은 확장자가 png로 크기가 435KB이다. <br>

web.dev 도큐먼트에서 추천하는 방법대로 webp 이미지를 사용해보기로 했다! <br>

자세한 내용은 여기에서 확인하세요!
>[WebP 이미지 사용](https://web.dev/serve-images-webp/)

같은 이미지인데도 webp로 변환한 파일은 그 용량이 고작 9.4KB밖에 되지 않았다.

메인 페이지에서 보여주는 이미지를 webp로 변경하자 TBT도 절반 가량 줄어들었다! 두둥..
![tbt](/assets/images/tbt-improved-2022-10-29.png)

<br><br>

직접 사용해보면서도 사이트를 접속하자마자 이미지가 바로 보여진다는 느낌을 받았다! <br>

FID 메트릭에도 유의미한 변화가 있었다.

{% include gallery id="gallery1" caption="왼쪽: FID 개선 이전, 오른쪽: FID 개선 후" %}


### 최적화된 비디오를 제공하자
위와 같은 맥락으로 비디오 파일은 webm으로 변환해보기로 했다. <br>
기존에는 비디오 파일의 확장자가 mp4로 25MB가 넘는 파일도 있었다.  <br>

다행히도(?) 비디오는 페이지 하단쯤에 보여지므로 웹 사이트에 처음 접근했을때 뷰포트에서는 보여지지 않기 때문에 뷰포트에 관여하는 LCP, CLS과 같은 메트릭 평가에서는 벗어날 수 있었다. <br>

하지만, 비디오가 완전히 로드되고 보여지기까지 오랜 시간이 걸린다면 사용자에게 나이스한 경험을 제공하기 어렵기 때문에 개선해야겠다고 생각했다. <br>
webm의 브라우저 호환성은 아래와 같다.

[can i use?](https://caniuse.com/?search=webm)


비디오 파일의 최적화는 아래 문서를 참고하실 수 있습니다!
> [애니메이션 GIF를 비디오로 대체하여 페이지를 더 빠르게 로드](https://web.dev/replace-gifs-with-videos/)


결과는 아주 아주 놀라웠다! <br>
일단 webm 파일은 그 용량이 아주 가벼웠다. <br>
mp4로는 25.6MB 이었던 용량이 webm으로 변환했을때는 2.7MB밖에 되지 않았다. <br>
다른 파일도 대부분 절반 이상씩 용량이 줄어들었다. <br>

또한, 크롭 개발자 도구의 네트워크 패널에서 콘텐츠 다운로드 시간만 보더라도 차이가 어마어마했다. <br>

{% include gallery id="gallery2" caption="왼쪽: MP4 파일의 콘텐츠 다운로드 시간, 오른쪽: WEBM 파일의 콘텐츠 다운로드 시간" %}

s 단위가 ms 단위로 바뀌었다. <br>

혹시나 특정 브라우저에서 호환되지 않을 가능성을 고려해서 source 태그를 이용해서 webm 혹은 mp4 파일로 보여주도록 설정해주었다. <br>

여기까지 했을때 TBT는 이전보다 절반 가량 더 줄었다 <br>

![tbt](/assets/images/tbt-more-improved-2022-10-29.png)

너무 재밌다.. 💗
