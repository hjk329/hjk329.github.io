---
title:  "React Query(리액트 쿼리): Mutations"
categories: 
  - React
tags:
  - React Query
  - 상태 관리
toc: true
toc_sticky: true
toc_label: "React Query: Mutations"
share: true
---
## Mutations
Queries와 다르게 Mutations는 주로 데이터를 생성하거나, 업데이트하거나, 지우거나 혹은 서버 사이드 이펙트를 수행하는 목적으로 사용된다. <br><br>
Mutations는 `useMutation` 훅으로 사용한다. <br>
아래 코드는 서버에 새로운 todo를 추가하는 mutation의 예시이다.

```javascript
function App() {
  const mutation = useMutation({
    mutationFn: newTodo => {
      return axios.post('/todos', newTodo)
    }
  })

  return (
    <div>
      {mutation.isLoading ? (
        'Adding todo...'
      ) : (
        <>
          {mutation.isError ? (
            <div>An error occurred: {mutation.error.message}</div>
          ) : null}

          {mutation.isSuccess ? <div>Todo added!</div> : null}

          <button
            onClick={() => {
              mutation.mutate({ id: new Date(), title: 'Do Laundry' })
            }}
          >
            Create Todo
          </button>
        </>
      )}
    </div>
  )
}
```
<br>

Mutation은 다음 상태중 하나의 값만 가질 수 있다. <br>
- `isIdle` 혹은 `status === 'idle'` : mutation은 현재 유휴 상태이거나 새로/재설정된 상태임
- `isLoading` 혹은 `status === 'loading'` : mutation은 현재 실행중임
- `isError` 혹은 `status === 'error'` : mutation에 오류가 발생함
- `isSuccess` 혹은 `status === 'success'` : mutation이 성공적이고 해당 데이터를 사용할 수 있음

<br>

이외에도 mutation의 상태에 따라 더 많은 정보를 확인할 수 있다.
- `error` : mutation이 `error` 상태에 있다면 `error` 프로퍼티에서 해당 에러를 확인할 수 있음
- `data` : mutation이 `success` 상태에 있다면 해당 데이터는 `data` 프로퍼티에서 확인할 수 있음

<br>

앞선 예시에서 mutation 함수에 변수를 전달하기 위해 mutate 라는 메소드를 사용했다. <br>
onSuccess 옵션, Query Client의 invalidateQueries 메소드 및 Query Client의 setQueryData 메소드와 함께 mutations를 더욱 강력하게 사용할 수 있다. <br>

```javascript
const CreateTodo = () => {
  const mutation = useMutation({ mutationFn: event => {
    event.preventDefault()
    return fetch('/api', new FormData(event.target))
  }})

  return <form onSubmit={mutation.mutate}>...</form>
}
```
<br>

### Mutation 상태값 재설정하기
mutation 의 `error` 나 `data` 상태를 초기화하기 위해  `reset` 함수를 사용할 수 있다.
```javascript
const CreateTodo = () => {
  const [title, setTitle] = useState('')
  const mutation = useMutation({ mutationFn: createTodo })

  const onCreateTodo = e => {
    e.preventDefault()
    mutation.mutate({ title })
  }

  return (
    <form onSubmit={onCreateTodo}>
      {mutation.error && (
        <h5 onClick={() => mutation.reset()}>{mutation.error}</h5>
      )}
      <input
        type="text"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <br />
      <button type="submit">Create Todo</button>
    </form>
  )
}
```
<br>

### Mutation 사이드 이펙트
`useMutation` 은 mutation의 라이프 사이클의 모든 단계에서 빠르고 쉽게 사이드 이펙트를 허용하는 몇가지 헬퍼 옵션이 있다. <br>
이는 mutation, optimistic update 이후 쿼리를 무효화하고 다시 가져오는 데 유용하다.

```javascript
useMutation({
  mutationFn: addTodo,
  onMutate: variables => {
    // mutation이 곧 발생함!

    // 롤백할때 사용할 데이터가 포함된 컨텍스트를 선택적으로 반환함
    return { id: 1 }
  },
  onError: (error, variables, context) => {
    // 오류가 발생함
    console.log(`rolling back optimistic update with id ${context.id}`)
  },
  onSuccess: (data, variables, context) => {
    
  },
  onSettled: (data, error, variables, context) => {

  },
})
```
<br>
콜백 함수에서 promise를 반환하면 다음 콜백이 호출되기 전에 기다린다.

```javascript
useMutation({
  mutationFn: addTodo,
  onSuccess: async () => {
    console.log("I'm first!")
  },
  onSettled: async () => {
    console.log("I'm second!")
  },
})
```
<br>

`mutate`를 호출할 때 `useMutation`에 정의된 콜백 외에 추가 콜백을 트리거하고 싶을 수 있다. <br>
mutation 변수 뒤에 동일한 콜백 옵션을 `mutate` 함수에 제공하면 된다. <br>
`onSuccess`, `onError`, `onSettled` 에 대해 오버라이드할 수 있다. <br>
하지만, 만약 컴포넌트가 mutation을 완료하기 전에 언마운트된다면 재정의한 콜백이 실행되지 않는다.

```javascript
useMutation({
  mutationFn: addTodo,
  onSuccess: (data, variables, context) => {
    // I will fire first
  },
  onError: (error, variables, context) => {
    // I will fire first
  },
  onSettled: (data, error, variables, context) => {
    // I will fire first
  },
})

mutate(todo, {
  onSuccess: (data, variables, context) => {
    // I will fire second!
  },
  onError: (error, variables, context) => {
    // I will fire second!
  },
  onSettled: (data, error, variables, context) => {
    // I will fire second!
  },
})
```
<br>

### 연속적인 Mutations
위의 예시처럼 연속적으로 mutation을 하는 경우 `onSuccess`, `onError`, `onSettled` 을 다루는 데에 차이점이 있다. <br>
위 콜백 옵션이 `mutate` 함수에 전달되면 컴포넌트가 마운트되어있을때 단 한번만 실행될 것이다. <br>
이는 mutation observer가 `mutate` 함수가 호출될 때마다 제거되고 재구독되기 때문이다. <br>
반면에 `useMutation` 핸들러는 `mutate`가 호출될 때마다 실행된다.

```javascript
useMutation({
  mutationFn: addTodo,
  onSuccess: (data, error, variables, context) => {
    // 3번 호출될것임
  },
})

['Todo 1', 'Todo 2', 'Todo 3'].forEach((todo) => {
  mutate(todo, {
    onSuccess: (data, error, variables, context) => {
      // 마지막 mutation인 Todo 3 이후에 단 한번만 실행됨
    },
  })
})
```
<br>

### Promises
`mutate` 대신 `mutateAsync`를 사용하면 성공시 리졸브되거나 오류를 발생시키는 프로미스를 얻을 수 있다. <br>
```javascript
const mutation = useMutation({ mutationFn: addTodo })

try {
  const todo = await mutation.mutateAsync(todo)
  console.log(todo)
} catch (error) {
  console.error(error)
} finally {
  console.log('done')
}
```
<br>

### Retry
리액트 쿼리는 mutation 요청이 실패했을때 자동으로 다시 시도하지 않지만, `retry` 옵션을 사용하면 된다. <br>
```javascript
const mutation = useMutation({
  mutationFn: addTodo,
  retry: 3,
})
```
<br>

### Persist Mutations
muatation은 필요한 경우 스토리지에 저장되고 나중에 다시 시작될 수 있다. <br>
```javascript
const queryClient = new QueryClient()

// Define the "addTodo" mutation
queryClient.setMutationDefaults(['addTodo'], {
  mutationFn: addTodo,
  onMutate: async (variables) => {
    // Cancel current queries for the todos list
    await queryClient.cancelQueries({ queryKey: ['todos'] })

    // Create optimistic todo
    const optimisticTodo = { id: uuid(), title: variables.title }

    // Add optimistic todo to todos list
    queryClient.setQueryData(['todos'], old => [...old, optimisticTodo])

    // Return context with the optimistic todo
    return { optimisticTodo }
  },
  onSuccess: (result, variables, context) => {
    // Replace optimistic todo in the todos list with the result
    queryClient.setQueryData(['todos'], old => old.map(todo => todo.id === context.optimisticTodo.id ? result : todo))
  },
  onError: (error, variables, context) => {
    // Remove optimistic todo from the todos list
    queryClient.setQueryData(['todos'], old => old.filter(todo => todo.id !== context.optimisticTodo.id))
  },
  retry: 3,
})

// Start mutation in some component:
const mutation = useMutation({ mutationKey: ['addTodo'] })
mutation.mutate({ title: 'title' })

// If the mutation has been paused because the device is for example offline,
// Then the paused mutation can be dehydrated when the application quits:
const state = dehydrate(queryClient)

// The mutation can then be hydrated again when the application is started:
hydrate(queryClient, state)

// Resume the paused mutations:
queryClient.resumePausedMutations()
```
<br>

### Persisting Offline Mutations
PersisQueryClient 플러그인을 사용하여 오프라인에서 mutation을 지속하는 경우 기본 mutation 함수를 제공하지 않는 한 페이지를 다시 로드할 때 mutation을 다시 시작하지 않는다. <br>
이는 기술적 한계이다. 외부 저장소에 유지하는 경우 함수를 직렬화할 수 없으므로 mutations 상태만 지속될 것이다. <br>
하이드레이션 이후 mutation을 유발하는 컴포넌트가 마운트되지 않을 수 있으므로 `resumePausedMutations`를 호출하면 `No mutationFn found.` 와 같은 오류가 발생할 수 있다.

```javascript
const persister = createSyncStoragePersister({
  storage: window.localStorage,
})
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      cacheTime: 1000 * 60 * 60 * 24, // 24 hours
    },
  },
})

// we need a default mutation function so that paused mutations can resume after a page reload
queryClient.setMutationDefaults(['todos'], {
  mutationFn: ({ id, data }) => {
    return api.updateTodo(id, data)
  },
})

export default function App() {
  return (
    <PersistQueryClientProvider
      client={queryClient}
      persistOptions={{ persister }}
      onSuccess={() => {
        // resume mutations after initial restore from localStorage was successful
        queryClient.resumePausedMutations()
      }}
    >
      <RestOfTheApp />
    </PersistQueryClientProvider>
  )
}
```
<br>

---
위 포스팅은 리액트 쿼리 공식 문서를 읽고 번역 및 정리한 내용입니다. <br>

참고 <br>
[Mutations](https://tanstack.com/query/v4/docs/guides/mutations) <br>

아래 포스팅에서도 리액트 쿼리에 대한 내용을 확인하실 수 있습니다! <br>
[React Query(리액트 쿼리)를 사용해보자: Queries](https://hjk329.github.io/react/react-query-queries/) <br>
[React Query(리액트 쿼리): Query Keys](https://hjk329.github.io/react/react-query-query-key/) <br>
[React Query(리액트 쿼리): Query Functions](https://hjk329.github.io/react/react-query-query-functions/) <br>