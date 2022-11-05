---
title:  "React Query(리액트 쿼리)를 사용해보자"
categories: 
  - React
tags:
  - React Query
  - 상태 관리
toc: true
toc_sticky: true
toc_label: "React Query"
share: true
published: true

---

상태 관리 라이브러리로 Redux(Redux-saga, Redux-toolkit), Recoil을 사용해왔다. <br>
프로덕트를 릴리즈한 이후에 캐싱에 대한 필요성이 대두되었다. <br>
빈번하게 변경되는 데이터가 아니라면 매번 서버에 요청을 날리는 것보다, 캐싱된 정보를 제공하는 것이 서버의 부담을 줄이면서도 사용자에게는 더 빠른 상호작용을 제공할 수 있기 때문이다. <br>

캐싱에 대해 알아보던중 서버 상태를 관리하는 라이브러리, 캐싱 처리에 강점을 가지는 React Query에 대해 알게되었다. <br>
사실 React Query에 대해 들어본 적은 많았지만, 익숙한 리덕스나 리코일만 사용하다보니 이제껏 리액트 쿼리를 제대로 공부하거나 사용해본 적이 없었다. (반성합니다 🥲) <br>

React Query 공식 문서를 읽고 이해한 내용을 정리하려고 한다.

## Motivation: 리액트 쿼리를 왜 사용할까?
React Query를 접하면서 꽤나 생소하게 느껴졌던 점이 서버 상태 관리와 클라이언트 상태 관리의 구별이었다. <br>
이전까지는 대개 전역 상태 관리, 글로벌(global) 상태 관리 등으로 서버 상태와 클라이언트 상태를 따로 구분해서 사용한 적이 없었다. <br> <br>

### React Query는 서버 상태와 클라이언트 상태를 구분한다. <br>
서버 상태는 단어에서 유추해볼 수 있듯이 비동기 통신을 통해 응답 받아서 서버와 클라이언트가 동기적으로 공유할 수 있는 데이터를 의미한다. <br>
서버 데이터를 가져오기 위해 비동기 API 요청이 필요하기 때문에 이는 다시 말해서 응답 받아온 데이터가 잠재적으로 '오래된 상태'가 될 수도 있다. <br>
클라이언트 상태는 클라이언트에서만 발생하고 사용하는 상태를 의미한다. <br>
예를 들어 Drawer가 열려있는지, 닫혀있는지, 스크롤Y 값 등등은 클라이언트 상태의 예시가 되겠다. <br> <br>

서버 상태에 대한 정의를 이해하고 나면 서버 상태와 관련하여 꽤나 많은 문제가 있다는걸 알 수 있다. 예를 들어 아래와 같다. <br>

### 서버 상태와 관련한 이슈들
- 캐싱 (동일한 데이터에 대한 여러 요청을 단일 요청으로 중복을 제거)
- 해당 데이터가 오래됐다는 것을 어떻게 알 수 있을까?
- 오래된 데이터인 경우 어떻게 빨리 업데이트해줄 수 있을까?

리액트 쿼리는 위와 같은 이슈를 해결하고자 한다. <br>

이제 리액트 쿼리 설치 방법부터 사용 방법을 차근차근 알아보겠다.

## Installation: 리액트 쿼리 설치하기

아래 명령어로 패키지를 설치한다.


```
$ npm i @tanstack/react-query
# or
$ pnpm add @tanstack/react-query
# or
$ yarn add @tanstack/react-query
```

## Queries

리액트 쿼리와 관련한 기본 개념을 알아보자!

### Queries 기본 개념

쿼리는 고유한 key에 연결된 비동기 데이터 소스에 대한 선언적 종속성이다. <br>
쿼리는 서버에서 데이터를 fetch하기 위해 GET, POST 방식과 같은 프로미스 기반의 메소드와 함께 사용할 수 있다. <br>
서버의 데이터를 수정하기 위해서는 Mutations 를 사용하길 권장한다. <br>

컴포넌트나 커스텀 훅에서 쿼리를 구독하려면 useQuery 라는 훅을 사용하면 된다. <br>
useQuery 훅을 사용할때는 최소한
- 쿼리에 대한 고유한 key가 있어야 한다
- 다음과 같은 프로미스를 리턴하는 함수가 있어야 한다
  - 데이터를 리졸브하거나, 혹은
  - 에러를 던진다

기본적인 사용 방법을 알아보겠다. <br>
 
```typescript
import { useQuery } from '@tanstack/react-query'

function App() {
  const info = useQuery({ queryKey: ['todos'], queryFn: fetchTodoList })
}
```

queryKey로 제공하는 고유한 key는 쿼리를 다시 가져오고, 캐싱하고, 공유하는 데 내부적으로 사용된다. <br>
useQuery에서 리턴하는 쿼리 결과는 데이터 사용에 필요한 쿼리에 대한 모든 정보를 포함한다.

```typescript
const result = useQuery({ queryKey: ['todos'], queryFn: fetchTodoList })
```

예를 들어, 위와 같이 useQuery가 반환하는 result라는 쿼리 결과는 아래와 같은 중요한 상태를 가진 객체이다. <br>
- `isLoading` 혹은 `status === 'loading'` : 쿼리에 아직 데이터가 없음
- `isError` 혹은 `status === 'error'` : 쿼리에 오류가 있음
- `isSuccess` 혹은  `status === 'success'` : 쿼리가 성공적이며 데이터를 사용할 수 있음

혹은 이라고 표현한 이유는 `isLoading`, `isError`, `isSuccess` 처럼 불린 값을 사용하는 대신에 `status` 라는 상태값을 사용할 수 있기 때문이다. <br>

위의 기본 상태 이외에도 사용할 수 있는 더 많은 정보가 있다.

- `error` : 만약 쿼리의 상태가 `isError` 라면 `error` 프로퍼티로 오류를 사용할 수 있음 
- `data` :  만약 쿼리의 상태가 `success` 라면 `data` 프로퍼티로 데이터를 사용할 수 있음

쿼리는 일반적으로 isLoading 상태를 확인한 다음 isError 상태를 확인하고, 마지막으로 데이터를 사용할 수 있다고 가정하여 성공적인 상태를 렌더링한다. <br>
예시 코드를 살펴 보자. <br>


- `isLoading`, `isError`, `isSuccess` 과 같은 불린 타입의 상태를 사용하는 예시

```typescript
function Todos() {
  const { isLoading, isError, data, error } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodoList,
  })

  if (isLoading) {
    return <span>Loading...</span>
  }

  if (isError) {
    return <span>Error: {error.message}</span>
  }

  // isLoading, isError까지 확인한 후에 isSuccess === true 라고 판단하여 상태값을 렌더링함
  return (
    <ul>
      {data.map(todo => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  )
}
```

- 불린 타입의 상태값 대신에 status라는 상태값을 이용하는 예시

```typescript
function Todos() {
  const { status, data, error } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodoList,
  })

  if (status === 'loading') {
    return <span>Loading...</span>
  }

  if (status === 'error') {
    return <span>Error: {error.message}</span>
  }

  // 마찬가지로 이 시점에서 status === 'success' 가 됨. "else" 로직 또한 사용할 수 있음
  return (
    <ul>
      {data.map(todo => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  )
}
```

또한 타입스크립트는 데이터에 엑세스하기 전에 `loading` 과 `error` 상태를 확인한 경우에 해당 데이터의 타입을 올바르게 파악한다.

### FetchStatus
`status` 필드와 더불어 `result` 객체는 `fetchStatus` 라는 프로퍼티를 갖는다. 해당 프로퍼티는 아래와 같은 옵션을 사용할 수 있다.
- `fetchStatus === 'fetching'` : 쿼리가 데이터를 가져오는 중임
- `fetchStatus === 'paused'` : 쿼리가 데이터를 가져오는 것을 멈춘 상태임
- `fetchStatus === 'idle'` : 쿼리는 현재 아무 작업도 하지 않음

### FetchStatus 와 status 
리액트 쿼리는 쿼리 상태에 대해 fetchStatus, status 와 같이 두 개의 상태를 제공한다. 이 둘은 어떻게 다를까? <br>
Background refetches (브라우저가 캐시에 대한 데이터를 재요청하는 것), stale-while-revalidate (캐싱 전략의 일종: 요청에 빠르게 응답하기 위해 캐싱된 데이터를 반환하고 캐싱된 데이터에 대해 재검증을 한다. 새로 가져온 데이터가 이전에 캐시된 데이터와 동일하지 않다면 새로운 데이터로 캐시를 갱신한다.) 로직은 `status`, `fetchStatus` 를 다양한 조합으로 사용할 수 있게 한다.

- `success` 상태인 쿼리는 대개 fetchStatus 프로퍼티의 `idle` 상태를 가지지만, background refetch가 발생하는 중이라면 `fetching` 상태일 수도 있다
- 마운트되고 데이터가 없는 쿼리는 대개 `loading` 상태이고 fetchStatus를 가져오는 중이지만, 네트워크 연결이 없는 경우 `paused` 상태일 수도 있다

따라서, 쿼리가 실제로 데이터를 가져오는 중이 아니면서 `loading` 상태일 수도 있다.
경험상 
- `status`는 데이터에 대한 정보를 알려준다 : 데이터가 있는지 없는지?
- `fetchStatus` 는 `queryFn(데이터 패칭을 위한 비동기 함수)` 에 대한 정보를 알려준다 : 해당 함수가 실행중인지 아닌지?

다음으로는 쿼리 키에 대해 알아보겠다!

---
참고
[React Query](https://tanstack.com/query/v4)