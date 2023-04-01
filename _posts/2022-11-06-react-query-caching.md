---
title:  "React Query(리액트 쿼리): Caching"
categories: 
  - React
tags:
  - React Query
  - 상태 관리
  - 리액트 쿼리 라이프 사이클
  - 캐싱
  - staleTime
  - cacheTime
toc: true
toc_sticky: true
toc_label: "React Query: Caching"
share: true
---

리액트 쿼리가 캐싱을 어떻게 관리하는지 알아보자  
이를 통해 리액트 쿼리에서 패치한 데이터의 라이프 사이클을 이해할 수 있다
아래 포스팅의 'Important Defaults' 의 `staleTime`과 `cacheTime`에 대해 이해하고 읽으시면 좋습니다 :)  
[React Query(리액트 쿼리)를 사용해보자: Queries](https://hjk329.github.io/react/react-query-queries/)

<br>

## `cacheTime` 을 5분, `staleTime`을 0으로 설정했다고 가정해보자.  

### `useQuery({ queryKey: ['todos'], queryFn: fetchTodos })` 인스턴스가 마운트된 상황 

이전에 `['todos']` 라는 쿼리 키를 가진 다른 쿼리가 호출된 적이 없었다면 위 쿼리는 데이터를 가져오기 위해 `hard loading state`와 네트워크 요청을 생성할 것이다.  

이때 `useQuery` 에서 리턴하는 `isLoading` 을 콘솔에 찍어본다면 true 라고 기록될 것이다.  
`isLoading` 은 `hard loading state` (예를 들어, 서버에 최초로 요청을 보냄)를 의미하기 때문이다.  

네트워크 요청이 완료되면 반환된 데이터는 `['todos']` 라는 쿼리 키 아래에 캐시될 것이다.  
캐시된 데이터는 `staleTime`이 지나면 해당 데이터를 stale한 것으로 간주된다.   

만약 `staleTime`이 지나지 않았다면 쿼리가 언마운트된 후 다시 마운트된다고 하더라도 데이터 패칭이 일어나지 않는다.  

위 예제에서 `staleTime` 을 0으로 설정했으니 네트워크 요청이 완료되어 데이터를 반환하자마자 stale한 것으로 간주한다.  
따라서, fresh한 데이터로 업데이트하기 위해 쿼리가 마운트될때마다 해당 쿼리 키에 대한 `Background Refetching`이 발생할 것이다.  

즉, `staleTime` 을 0으로 설정하면 데이터 패칭이 빈번하게 발생하므로 올바른 설정인지 잘 고민해보자!

<br><br>

### `useQuery({ queryKey: ['todos'], queryFn: fetchTodos }` 인스턴스가 다른 곳에서 마운트된 상황
`['todos']` 라는 쿼리 키를 가진 쿼리에 캐시된 데이터가 있기 때문에 캐시된 데이터를 가져올 것이다.  

따라서, 캐시 데이터를 사용하기 위해 동일한 쿼리 키를 사용하는 것이 중요하다!

새로운 인스턴스는 쿼리 함수를 통해 새로운 네트워크 요청을 트리거할 것이다.
fetchTodos 쿼리 함수가 동일한지에 관계없이 두 인스턴스의 쿼리 키가 동일하기 때문에 두 쿼리의 상태(`isFetching`, `isLoading` 및 기타 관련 값 포함)가 업데이트될 것이다.
네트워크 요청이 성공적으로 완료되면 `['todos']` 키 아래의 캐시 데이터가 새 데이터로 업데이트되고 두 인스턴스 모두 새 데이터로 업데이트될 것이다.

<br><br> 

### `useQuery({ queryKey: ['todos'], queryFn: fetchTodos })` 위 두 인스턴스가 언마운트되어서 더이상 사용되지 않는 상황  

해당 쿼리에 대해 활성화된 인스턴스가 없기 때문에 `cacheTime`에 따라서 캐시 시간 초과가 설정되어 쿼리를 삭제하고 가비지 콜렉터의 수집 대상이 될 것이다(기본값은 5분임).
Since there are no more active instances of this query, a cache timeout is set using cacheTime to delete and garbage collect the query (defaults to 5 minutes).  

<br><br>

만약 캐시 초과 시간이 완료되기 전에 `useQuery({ queryKey: ['todos'], queyFn: fetchTodos })` 의 다른 인스턴스가 마운트된다면 쿼리는 fetchTodos 함수가 백그라운드에서 실행되는 동안 사용 가능한 캐시된 데이터를 즉시 반환한다.   
성공적으로 완료되면 캐시를 새 데이터로 교체한다.

<br><br>

`useQuery({ queryKey: ['todos'], queryFn: fetchTodos })` 인스턴스가 이제 더는 사용되지 않고, `cacheTime`으로 설정한 5분이 지났다면  
`['todos']` 라는 쿼리 키 아래에 저장된 캐시 데이터는 삭제되고 가비지 콜렉터로 수집될 것이다.   


실험해본 결과! `cacheTime`이 초과하면 GC의 수거 대상이 될 뿐, 즉각적으로 삭제되는 것은 아니다.  
구체적으로 GC가 해당 캐시를 `cacheTime`이 지난 어느 시점에서 삭제할지는 알 수 없다.  


--- 
참고
[Caching Examples](https://tanstack.com/query/v4/docs/guides/caching)
