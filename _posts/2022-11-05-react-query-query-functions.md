---
title:  "React Query(리액트 쿼리): Query Functions"
categories: 
  - React
tags:
  - React Query
  - 상태 관리
toc: true
toc_sticky: true
toc_label: "React Query: Query Functions"
share: true
---

## Query Functions
쿼리 함수는 프로미스를 리턴하는 함수이다.

다음 예시는 유효한 쿼리 함수 구성의 예시이다.

```typescript
useQuery({ queryKey: ['todos'], queryFn: fetchAllTodos })
useQuery({ queryKey: ['todos', todoId], queryFn: () => fetchTodoById(todoId) })
useQuery({ queryKey: ['todos', todoId], queryFn: async () => {
  const data = await fetchTodoById(todoId)
  return data
}})
useQuery({
  queryKey: ['todos', todoId],
  queryFn: ({ queryKey }) => fetchTodoById(queryKey[1]),
})
```
쿼리 함수에 queryKey를 받아와서 파라미터로 넘겨줄 수도 있다!

### 에러 처리하기
리액트 쿼리는 쿼리에 에러가 발생했는지 확인하기 위해 쿼리 함수에서 rejected된 프로미스를 리턴해야 한다. <br>
쿼리 함수에서 발생한 모든 오류는 쿼리의 `error` 상태에서 지속될 것이다.

```typescript
const { error } = useQuery({
  queryKey: ['todos', todoId],
  queryFn: async () => {
    if (somethingGoesWrong) {
      throw new Error('Oh no!')
    }
    if (somethingElseGoesWrong) {
      return Promise.reject(new Error('Oh no!'))
    }

    return data
  }
})
```

### 기본적으로 throw하지 않는 fetch나 다른 클라이언트와 함께 사용하는 방법
axios나 graphql-request는 실패한 HTTP 요청에 대해 자동으로 오류를 발생시키는 반면, fetch는 기본적으로 오류를 발생시키지 않는다. <br>
따라서, 아래와 같이 직접 예외 처리를 해야 한다. <br>
```typescript
useQuery({
  queryKey: ['todos', todoId],
  queryFn: async () => {
    const response = await fetch('/todos/' + todoId)
    if (!response.ok) {
      throw new Error('Network response was not ok')
    }
    return response.json()
  }
})
```

### 쿼리 함수 변수
쿼리 키는 패칭하는 데이터에 대한 고유한 설명일 뿐만 아니라, QueryFunctionContext의 일부로 쿼리 함수에도 전달된다! <br>
항상 필요한 것은 아니지만, 필요한 경우에 쿼리 함수를 추출하기 쉬워진다. <br>
```typescript
function Todos({ status, page }) {
  const result = useQuery({
    queryKey: ['todos', { status, page }],
    queryFn: fetchTodoList,
  })
}

// 쿼리 키에 전달해줬던 key, status, page 변수에 대해 쿼리 함수에서 접근할 수 있다!
function fetchTodoList({ queryKey }) {
  const [_key, { status, page }] = queryKey
  return new Promise()
}
```

### QueryFunctionContext
`QueryFunctionContext`는 각 쿼리 함수에 전달되는 객체이다. <br> 아래와 같은 정보를 가진다.
- `queryKey: QueryKey` : 쿼리 키
- `pageParam?: unknown` : Infinite Queries에서만 사용됨 (현재 페이지를 가져오기 위해 사용되는 페이지 파라미터)
- `signal?: AbortSignal` : 리액트 쿼리에서 제공되는 AbortSignal 인터페이스, Query Cancellation에 사용할 수 있음
- `meta: Record<string, unknown> | undefined` : 쿼리에 대한 추가 정보를 제공할 수 있는 옵셔널한 필드


---
위 내용은 리액트 쿼리 공식 문서를 읽고 번역 및 정리한 내용입니다 :) <br>
리액트 쿼리와 관련한 다른 포스팅은 아래 링크에서도 확인하실 수 있습니다 <br>
[React Query(리액트 쿼리)를 사용해보자: Queries](https://hjk329.github.io/react/react-query-queries/) <br>
[React Query(리액트 쿼리): Query Keys](http://localhost:4000/react/react-query-query-key/) 


참고 <br>
[Query Functions](https://tanstack.com/query/v4/docs/guides/query-functions)

---



