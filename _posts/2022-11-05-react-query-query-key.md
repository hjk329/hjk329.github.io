---
title:  "React Query(리액트 쿼리): Query Keys"
categories: 
  - React
tags:
  - React Query
  - 상태 관리
toc: true
toc_sticky: true
toc_label: "React Query: Query Keys"
share: true
---

## Query Keys
기본적으로 리액트 쿼리는 쿼리의 고유한 key를 바탕으로 해당 쿼리에 대한 캐싱을 관리한다.

---
리액트 쿼리 공식 문서를 읽고 번역 및 정리한 내용입니다 :) <br>
1편은 아래 링크를 참고해주세요! <br>
[React Query(리액트 쿼리)를 사용해보자: Queries](https://hjk329.github.io/react/react-query-queries/)

---

쿼리 키는 단순히 문자열을 포함하는 배열에서부터, 중첩 객체를 가지는 배열이 될 수도 있다. <br> 쿼리 키가 고유하고, 직렬화가능한 한 사용할 수 있다.

### 단순한 쿼리 키
주로 리스트나, 인덱스 리소스, 위계가 없는 리소스에 대해서는 쿼리 키를 이렇게 작성한다.

```typescript
// A list of todos
useQuery({ queryKey: ['todos'], ... })

// Something else, whatever!
useQuery({ queryKey: ['something', 'special'], ... })
```

<br>

### 변수가 있는 배열 키 (조금 더 복잡한 쿼리 키)
쿼리 키에 쿼리의 데이터를 설명하기 위한 추가 정보를 전달할 수도 있다.
- 항목을 고유하게 식별하기 위해 ID, 인덱스 또는 기타 프리미티브를 전달하거나,
- 추가적인 파라미터를 전달하기 위해 위와 같은 형태로 쿼리 키를 작성한다.

```typescript
// An individual todo
useQuery({ queryKey: ['todo', 5], ... })

// An individual todo in a "preview" format
useQuery({ queryKey: ['todo', 5, { preview: true }], ...})

// A list of todos that are "done"
useQuery({ queryKey: ['todos', { type: 'done' }], ... })
```

<br>

### 쿼리 키는 결과적으로 해시된다.
즉, 객체 안의 키의 순서가 어떻든지 모두 동일한 것으로 간주된다.

```typescript
useQuery({ queryKey: ['todos', { status, page }], ... })
useQuery({ queryKey: ['todos', { page, status }], ...})
useQuery({ queryKey: ['todos', { page, status, other: undefined }], ... })
```

<br>
하지만, '배열'일 경우 인덱스의 순서가 중요하므로 다음 예시에서는 쿼리 키가 동등하게 간주되지 않는다.
```typescript
useQuery({ queryKey: ['todos', status, page], ... })
useQuery({ queryKey: ['todos', page, status], ...})
useQuery({ queryKey: ['todos', undefined, page, status], ...})
```

<br>

### 쿼리 함수가 변수에 의존한다면 쿼리 키에 변수를 포함한다
쿼리 키는 가져오는 데이터를 고유하게 설명하는 용도로 사용되므로 쿼리 함수를 변경하는 어떤 변수든지 쿼리 키에 포함되어야 한다. <br>
```typescript
function Todos({ todoId }) {
  const result = useQuery({
    queryKey: ['todos', todoId],
    queryFn: () => fetchTodoById(todoId),
  })
}
```