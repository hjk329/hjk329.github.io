---
title:  "React Query(리액트 쿼리): Query Invalidation"
categories: 
  - React
tags:
  - React Query
  - 상태 관리
toc: true
toc_sticky: true
toc_label: "React Query: Query Invalidation"
share: true

---

## Query Invalidation

리액트 쿼리는 서버 데이터가 오래되었다는 사실을 어떻게 파악할 것인지, 오래되었다는 것을 알았다면 언제 업데이트해줄 것인지 등의 이슈를 해결하려 한다. <br>
이를 위해 QueryClient에는 쿼리를 오래된 것으로 표시하고, 잠재적으로 다시 가져올 수 있는 invalidateQueries 메서드를 제공한다. <br>

```javascript
// 캐시의 모든 쿼리를 무효화함
queryClient.invalidateQueries()
// `todos`로 시작하는 키로 모든 쿼리를 무효화함
queryClient.invalidateQueries({ queryKey: ['todos'] })
```
<br>

쿼리가 `invalidateQueries` 메서드로 무효화될때 다음과 같은 두 가지 상황이 발생한다.
- 쿼리가 오래된 것으로 표시된다. 오래된 상태는 `useQuery` 또는 관련 훅에서 사용 중인 모든 `staleTime` 구성에 오버라이드한다.
- 만약 `useQuery` 나 관련 훅을 통해 쿼리가 렌더링되고 있다면 백그라운드에서도 refetch한다.
<br>

### invalidateQueries 을 사용해서 쿼리 매칭시키기
`invalidateQueries` 및 `removeQueries`와 같은 API를 사용할 때 프리픽스로 여러 쿼리를 일치시키거나 매우 구체적이고 정확한 쿼리를 일치시킬 수 있다. <br>
예를 들어, todo 프리픽스를 사용하여 쿼리 키에서 todos로 시작하는 모든 쿼리를 무효화할 수 있다.

```javascript
import { useQuery, useQueryClient } from '@tanstack/react-query'

const queryClient = useQueryClient()

queryClient.invalidateQueries({ queryKey: ['todos'] })

// 아래 두 쿼리 모두 무효화됨
const todoListQuery = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodoList,
})
const todoListQuery = useQuery({
  queryKey: ['todos', { page: 1 }],
  queryFn: fetchTodoList,
})
```
<br>

더 구체적인 쿼리 키를 `invalidateQueries` 메서드에 전달하여 특정 변수가 있는 쿼리를 무효화할 수도 있다.
```javascript
queryClient.invalidateQueries({
  queryKey: ['todos', { type: 'done' },
})

// 아래 쿼리는 무효화될 것임
const todoListQuery = useQuery({
  queryKey: ['todos', { type: 'done' }],
  queryFn: fetchTodoList,
})

// 얘는 무효화되지 않음!
const todoListQuery = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodoList,
})
```
<br>

변수나 하위 키가 없는 todo 쿼리만 무효화하려는 경우에는 `exact: true` 옵션을 `invalidateQueries` 메소드에 전달하여 각 쿼리를 구분해서 무효화할 수 있다.

```javascript

// 쿼리 키에 변수나 하위 키가 없음
queryClient.invalidateQueries({
  queryKey: ['todos'],
  exact: true,
})

// 아래 쿼리는 무효화될 것임
const todoListQuery = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodoList,
})

// 아래 쿼리는 무효화되지 않음
const todoListQuery = useQuery({
  queryKey: ['todos', { type: 'done' }],
  queryFn: fetchTodoList,
})
```
<br>

`predicate` 함수를 전달하면 해당 쿼리를 무효화할지 여부에 대해 true 또는 false를 반환할 수 있다.

```javascript
queryClient.invalidateQueries({
  predicate: query =>
    query.queryKey[0] === 'todos' && query.queryKey[1]?.version >= 10,
})

// 아래 쿼리는 무효화됨
const todoListQuery = useQuery({
  queryKey: ['todos', { version: 20 }],
  queryFn: fetchTodoList,
}),

// 아래 쿼리는 무효화됨
const todoListQuery = useQuery({
  queryKey: ['todos', { version: 10 }],
  queryFn: fetchTodoList,
}),

// 아래 쿼리는 무효화되지 않음
const todoListQuery = useQuery({
  queryKey: ['todos', { version: 5 }],
  queryFn: fetchTodoList,
}),
```
<br><br>

---
위 포스팅은 리액트 쿼리 공식 문서를 읽고 번역 및 정리한 내용입니다. <br>

참고 <br>
[Query Invalidation](https://tanstack.com/query/v4/docs/guides/query-invalidation) <br>

아래 포스팅에서도 리액트 쿼리에 대한 내용을 확인하실 수 있습니다! <br>
[React Query(리액트 쿼리)를 사용해보자: Queries](https://hjk329.github.io/react/react-query-queries/) <br>
[React Query(리액트 쿼리): Query Keys](https://hjk329.github.io/react/react-query-query-key/) <br>
[React Query(리액트 쿼리): Query Functions](https://hjk329.github.io/react/react-query-query-functions/) <br>
[React Query(리액트 쿼리): Mutations](https://hjk329.github.io/react/react-query-mutations/)