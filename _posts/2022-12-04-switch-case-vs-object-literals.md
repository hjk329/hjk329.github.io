---
title:  "Switch Case vs Object Literals"
categories: 
  - Javascript
tags:
  - Switch Case
  - Object Literals
toc: true
toc_sticky: true
toc_label: "Switch Case vs Object Literals"
share: true

---

## Switch Case와 Object Literals 중 어떤 것을 사용하는게 좋을까?

참고 [Object Lookups over Switch Statements and If/Else Conditionals](https://dev.to/laurenclark/object-lookups-over-switch-statements-and-if-else-conditionals-2p5h)   
[Replacing switch statements with Object literals](https://ultimatecourses.com/blog/deprecating-the-switch-statement-for-object-literals)  

### 성능면에서는 큰 차이 없는 것 같다
[switch vs object lookup performance](https://stackoverflow.com/questions/37730199/switch-vs-object-lookup-performance-since-jsperf-is-down)


### Switch Case의 단점
- `break` 혹은 함수의 경우 `return`을 사용해서 수동으로 중단해주어야 합니다
- `default` 를 설정해주어야 합니다.
- 각 케이스 내의 명령문은 디버깅을 복잡하게 할 수도 있습니다.
- 어떤 value가 무엇을 리턴하는지 Switch Case의 expression과, value를 계속 주시해야 한다.
- 더 많은 case가 추가될수록 코드를 파악하기에 어려워질 것이다.

## 정리
아래와 같은 경우에는 Switch Case보다는 Object Literals를 활용하도록 재고해보자.
- 체크해야할 value와 type이 많거나 복잡할때
- 다른 요소에서 인자를 동적으로 입력할 수 있도록 매개변수가 있는 함수를 만들고 싶을때
- 코드의 가독성을 높이고 싶을때