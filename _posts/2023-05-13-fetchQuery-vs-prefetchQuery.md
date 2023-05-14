---
title:  "fetchQuery vs prefetchQuery (React-Query 리액트 쿼리)"
tags:
  - React Query
  - 상태 관리
  - fetchQuery
  - prefetchQuery
  - 프리 패칭
  - Next.js
  - SSR
toc: true
toc_sticky: true
toc_label: "🤭 fetchQuery vs prefetchQuery"
share: true

---

Next.js 에서 리액트 쿼리를 활용해서 서버 사이드 렌더링 시점에 데이터를 프리 패칭할 수 있다.  
프리 패칭하고 캐시한 데이터가 있으면 사용자의 대기 시간을 줄일 수 있다.  

<br/>

# 👀 프리 패칭하는 방법

리액트 쿼리 공식 문서에서는 데이터 프리 패칭하는 방식을 두 가지로 설명한다.  

1. `initialData` 
서버에서 데이터를 프리 패칭하고 `initialData` 로 컴포넌트에 내려주는 방식이다.  
이건 시도해보진 않았다.  
뎁스가 깊어지면 비효율적이라고 생각했기 때문이다. (props-drilling이 될 수도 있다)  
또한, 동일한 쿼리 키를 사용하는 모든 위치에 `initialData` 를 넘겨줘야 하기 때문에 휴먼 에러가 발생하기 쉽다고 생각했다.  

2. `Hydration`
서버에서 프리 패칭한 데이터를 dehydrate 해서 클라이언트 사이드로 넘겨주고, 클라이언트 사이드에서는 rehydrate 해서 사용한다.  

공식 문서에 소개된 예제를 그대로 따라하면 된다! 아주 간단하다!🤗  

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

```typescript
// pages/posts.jsx
// 데이터를 프리 패칭하려는 페이지 컴포넌트
import { dehydrate, QueryClient, useQuery } from '@tanstack/react-query'

export async function getStaticProps() {
  const queryClient = new QueryClient()

  await queryClient.prefetchQuery(['posts'], getPosts)

  return {
    props: {
      dehydratedState: dehydrate(queryClient), // dehydrate 함수로 서버 사이드 렌더링에서 패치한 데이터를 직렬화
    },
  }
}

function Posts() {
  // This useQuery could just as well happen in some deeper child to
  // the "Posts"-page, data will be available immediately either way
  const { data } = useQuery({ queryKey: ['posts'], queryFn: getPosts }) // 리액트 쿼리는 쿼리 키로 캐싱을 관리한다!

  // This query was not prefetched on the server and will not start
  // fetching until on the client, both patterns are fine to mix
  const { data: otherData } = useQuery({
    queryKey: ['posts-2'],
    queryFn: getPosts,
  })

  // ...
}
```

이렇게 하면 서버 사이드에서 프리 패칭한 데이터를 캐시해서 사용할 수 있다.  
주의할 점은 리액트 쿼리는 `queryKey` 로 캐시를 관리하기 때문에 서버 사이드에서 호출하는 쿼리 키와 클라이언트 사이드에서 호출하는 쿼리 키를 동일하게 설정해야 한다.  
클라이언트 사이드에서 `useQuery` 를 호출하지 않는다면 고려하지 않아도 된다.  
<br/>

## 프리 패칭한 데이터인지를 어떻게 확인할 수 있을까?  

`useQuery` 에서 `isFetching`, `isLoading` 을 리턴하고, 콘솔 로그에 찍어 보자.  

`isLoading` 은 하드 로딩 상태를 의미한다. 즉, 서버에서 실제로 데이터를 통신하는 상태를 나타낸다.  
예를 들어, 캐시된 데이터가 없어서 서버에 요청을 날리는 상황에는 `isLoading` 이 `true` 이다.  

`isFetching` 은 캐시된 데이터를 가져오는 상태이다.  

자세한 내용은 공식 문서를 확인해주세요! [useQuery](https://tanstack.com/query/v4/docs/react/reference/useQuery)

만약, 하드 로딩이 아니라 캐시를 가져온다면 `isLoading` 은 `false`, `isFetching`은 `true` 이다.  

즉, 서버 사이드에서 데이터가 제대로 프리 패칭이 되었고, 클라이언트 사이드에서 동일한 쿼리 키로 `useQuery` 를 호출한다면 위에 적은 대로 콘솔에 찍힌다.  

예시 화면)
![state](https://github.com/hjk329/hjk329.github.io/assets/84058944/2ea8e6b0-2547-422a-9cd4-26c0cda92c8d)  

우리 팀은 백오피스의 CMS로 블로그 게시물을 직접 발행하고 있다.  
SSG로 구현했는데 만약 게시물에 변경 사항이 생긴 경우 즉각 반영될 수 있도록 SSG에서 `fetchQuery` 메소드로 호출하는 쿼리 키와 동일한 쿼리를 클라이언트 사이드에서도 호출한다.  

만약 서버 사이드 렌더링 시점에 캐시한 데이터에 변경 사항이 없다면 캐시된 데이터를 바로 보여준다.  
블로그 게시물을 발행한 이후에 내용이 수정되더라도 Background Refetching 으로 신속하게 대응할 수 있다.  

<br/>
<br/>

# 그래서 fetchQuery 와 prefetchQuery 는 무엇이 다를까?

공식 문서의 예제에서는 `prefetchQuery` 메소드를 사용해서 데이터를 프리 패칭했다.  
`prefetchQuery` 메소드는 데이터를 프리 패칭하고 캐시할 뿐, 어떠한 결과 값을 리턴하지 않는다.  
반면에 `fetchQuery` 메소드는 데이터를 프리 패칭하고 캐시하고, 결과 값까지 리턴한다.  

따라서, 목적에 적합한 메소드를 선택해야 한다.  

단순히 프리 패칭만 하면 되는 경우 `prefetchQuery` 를 사용하면 되겠다.  
하지만, 서버 사이드 렌더링 시점에서 패치한 데이터의 값 자체를 사용해야 한다면 `fetchQuery` 를 사용한다.  

예를 들어, next-seo 같은 패키지를 사용한다고 가정해 보자.  
서버 사이드 렌더링때 패치한 데이터를 SEO 의 props 값으로 넘겨주고 싶다면 실제로 패치한 데이터의 `값` 이 필요하다.  
이럴때는 `fetchQuery` 를 사용해서 리턴 받은 값을 사용하면 된다.  

자세한 내용은 공식 문서를 확인해 주세요!  
[queryclientfetchquery](https://tanstack.com/query/v4/docs/react/reference/QueryClient#queryclientfetchquery)

<br/>
<br/>

그럼 이만 (정중하게)  
![image](https://github.com/hjk329/hjk329.github.io/assets/84058944/0cf6bf31-b557-4441-af23-ee4ad71e6567)  

오늘도 행복 코딩 하세용~

이미지 출처: @namsee.jpg (https://www.instagram.com/namsee.jpg)

