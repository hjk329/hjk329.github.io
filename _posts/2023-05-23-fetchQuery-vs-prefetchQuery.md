---
title:  "fetchQuery vs prefetchQuery (React-Query 리액트 쿼리)"
tags:
  - React Query
  - 상태 관리
toc: true
toc_sticky: true
toc_label: "🤭 fetchQuery vs prefetchQuery"
share: true

---

Next.js 에서 리액트 쿼리를 활용해서 서버 사이드 렌더링 시점에 데이터를 프리패칭할 수 있다.  
프리 패칭을 하면 사용자의 대기 시간을 줄일 수 있다.  

리액트 쿼리 공식 문서를 보면 프리 패칭하는 방식을 두 가지로 설명한다.  

1. `initialData` 
서버에서 데이터를 프리 패칭한 다음 `initialData` 로 컴포넌트에 내려주는 방식이다.  
이건 시도해보진 않았다.  
뎁스가 깊어지면 비효율적이라고 생각했기 때문이다. (props-drilling이 될 수도 있다)  
또한, 동일한 쿼리 키를 사용하는 모든 위치에 `initialData` 를 넘겨줘야 하기 때문에 휴먼 에러가 발생하기 쉽겠다고 생각했다.  

2. `Hydration`
서버에서 프리 패칭한 데이터를 dehydrate 해서 클라이언트 사이드로 넘겨주고, 클라이언트 사이드에서는 rehydrate 해서 사용한다.  

공식 문서에 소개된 예제를 그대로 따라하면 된다! 아주 간단하다!  

```typescript
// _app.jsx
import {
  Hydrate,
  QueryClient,
  QueryClientProvider,
} from '@tanstack/react-query'

export default function MyApp({ Component, pageProps }) {
  const [queryClient] = React.useState(() => new QueryClient())

  return (
    <QueryClientProvider client={queryClient}>
      <Hydrate state={pageProps.dehydratedState}> // 쿼리 클라이언트를 초기화 
                                                // (서버 사이드에서 가져온 데이터를 클라이언트 사이드로 전달하고, 클라이언트 사이드에서 초기 데이터를 사용하여 렌더링을 수행)
        <Component {...pageProps} />
      </Hydrate>
    </QueryClientProvider>
  )
}
```