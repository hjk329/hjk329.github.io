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
Queries와 다르게 Mutations는 주로 데이터를 생성하거나, 업데이트하거나, 지우거나 혹은 서버 사이드 이펙트를 수행하는 목적으로 사용된다. <br>
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

Mutation은 다음 상태중 하나의 값만 가질 수 있다. <br>
- `isIdle` 혹은 `status === 'idle'` : mutation은 현재 유휴 상태이거나 새로/재설정된 상태임
- `isLoading` 혹은 `status === 'loading'` : mutation은 현재 실행중임
- `isError` 혹은 `status === 'error'` : mutation에 오류가 발생함
- `isSuccess` 혹은 `status === 'success'` : mutation이 성공적이고 해당 데이터를 사용할 수 있음

이외에도 mutation의 상태에 따라 더 많은 정보를 확인할 수 있다.
- `error` : mutation이 `error` 상태에 있다면 `error` 프로퍼티에서 해당 에러를 확인할 수 있음
- `data` : mutation이 `success` 상태에 있다면 해당 데이터는 `data` 프로퍼티에서 확인할 수 있음

<br>

앞선 예시에서 mutation 함수에 변수를 전달하기 위해 mutate 라는 메소드를 사용했다. <br>
onSuccess 옵션, Query Client의 invalidateQueries 메소드 및 Query Client의 setQueryData 메소드와 함께 mutations를 더욱 강력하게 사용할 수 있다. <br>
