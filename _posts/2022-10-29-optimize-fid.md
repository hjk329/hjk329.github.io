---
title:  "[코어 웹 바이탈 최적화] FID를 개선해보자!"
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

gallery:
  - url: /assets/images/FID-by-extension-2022-10-29.png
    image_path: /assets/images/FID-by-extension-2022-10-29.png
    alt: 'fid-before'
  - url: /assets/images/FID-improved-by-extension-2022-10-29.png
    image_path: /assets/images/FID-improved-by-extension-2022-10-29.png
    alt: 'fid-improved'
---


## TBT 향상을 통해 FID 개선하기

FID를 개선하는 일은 꽤나 까다롭게 느껴졌다. <br>
코어 웹 바이탈 도큐먼트에서 추천해준 방식은 JS 번들을 여러 청크로 코드 분할하거나, Polyfills를 최소화하는 방법 등이었는데 이에 대한 이해가 선행되어야할 것 같아서 지금 당장 시도해볼 엄두가 나지 않았다. <br>
위 방법은 공부하면서 차차 시도해볼 예정이다. <br> <br>

그렇다면 FID를 개선할 수 있는 다른 방법은 무엇이 있을까 고민하다가 FID가 TBT와 관련이 있다는 것이 생각났다 <br>
TBT(Total Blocking Time, 총 차단 시간) 메트릭은 코어 웹 바이탈에 포함되는 메트릭은 아니지만, 메인 스레드가 입력 응답을 막는 시간과 관련있기 때문에 이를 개선하면 FID도 자연스레 개선할 수 있을 것이라 생각했다. <br>

> TBT에 대한 자세한 내용은 여기를 참고해주세요! [Total Blocking Time(총 차단 시간, TBT)](https://web.dev/tbt/)

<br>

우선 우리 프로젝트의 TBT는 아래와 같다. <br> <br>
![tbt](/assets/images/tbt-as-is-2022-10-29.png)

<br><br>
총 차단 시간이 330ms로 좋은 TBT 점수로 평가되는 300ms를 조금 넘었다. <br>


우리 메인 페이지에서 TBT에 영향을 미치는 요인은 메인 이미지라고 생각했다. <br>
모바일로 웹 페이지에 접속했을때 메인 이미지의 자리가 비어있는데 1-2초 뒤쯤에 이미지 렌더링이 완료되었기 때문이다. <br>

그래서 이미지 최적화를 해보기로 했다. <br>
현재 이미지는 CDN을 이용해서 S3에 저장된 정적 이미지를 가져오고 있다. <br>
해당 이미지 파일은 확장자가 png로 크기가 435KB이다. <br>

web.dev 도큐먼트에서 추천하는 방법대로 webp 이미지를 사용해보기로 했다! <br>
같은 이미지인데도 webp로 변환한 파일은 그 용량이 고작 9.4KB밖에 되지 않았다. <br>

메인 페이지에서 보여주는 이미지를 webp로 변경하자 TBT도 절반 가량 줄어들었다! 두둥.. <br><br>

![tbt](/assets/images/tbt-improved-2022-10-29.png)

<br><br>

직접 사용해보면서도 사이트를 접속하자마자 이미지가 바로 보여진다는 느낌을 받았다! <br>

FID 메트릭에도 유의미한 변화가 있었다.

{% include gallery id="gallery" caption="왼쪽: FID 개선 이전, 오른쪽: FID 개선 후" %}




