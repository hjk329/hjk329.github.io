---
title:  "React Query(리액트 쿼리): Caching"
categories: 
  - React
tags:
  - React Query
  - 상태 관리
toc: true
toc_sticky: true
toc_label: "React Query: Caching"
share: true
---

리액트 쿼리가 캐싱을 어떻게 관리하는지 알아보자<br><br>
아래 포스팅의 'Important Defaults' 내용을 이해하고 읽으시면 좋습니다 :)  
[React Query(리액트 쿼리)를 사용해보자: Queries](https://hjk329.github.io/react/react-query-queries/)

<br>

`cacheTime` 을 5분으로 설정하고, `staleTime`을 0으로 설정했다고 가정해보자.  

`useQuery({ queryKey: ['todos'], queryFn: fetchTodos })` 요런 인스턴스가 마운트된 상황을 생각해보자.  

- `['todos']` 라는 쿼리 키를 가진 다른 쿼리가 생성되지 않았기 때문에 위 쿼리는 데이터를 가져오기 위해 hard loading state와 네트워크 요청을 생성할 것이다.
- 네트워크 요청이 완료되면 반환된 데이터는 `['todos']` 라는 쿼리 키 아래에 캐시될 것이다.
- `staleTime`이 지나면 해당 데이터를 stale한 것으로 간주할 것이다. 만약 `staleTime`이 지나지 않았다면 쿼리가 언마운트된 후 다시 마운트된다고 하더라도 데이터 패칭이 일어나지 않는다.

<br><br>

`useQuery({ queryKey: ['todos'], queryFn: fetchTodos }` 인스턴스가 다른 곳에서 마운트되었다.
- `['todos']` 라는 쿼리 키를 가진 쿼리에 캐시된 데이터가 있기 때문에 캐시된 데이터를 가져올 것이다.
- 새로운 인스턴스는 쿼리 함수를 통해 새로운 네트워크 요청을 트리거할 것이다.
  - fetchTodos 쿼리 함수가 동일한지에 관계없이 두 인스턴스의 쿼리 키가 동일하기 때문에 두 쿼리의 상태(`isFetching`, `isLoading` 및 기타 관련 값 포함)가 업데이트될 것이다.
- 네트워크 요청이 성공적으로 완료되면 `['todos']` 키 아래의 캐시 데이터가 새 데이터로 업데이트되고 두 인스턴스 모두 새 데이터로 업데이트될 것이다.

<br><br> 

`useQuery({ queryKey: ['todos'], queryFn: fetchTodos })` 위 두 인스턴스가 언마운트되어서 더이상 사용되지 않는다고 가정해보자.  
- 해당 쿼리에 대해 활성화된 인스턴스가 없기 때문에 `cacheTime`에 따라서 캐시 시간 초과가 설정되어 쿼리를 삭제하고 가비지 콜렉터의 수집 대상이 될 것이다(기본값은 5분임).
Since there are no more active instances of this query, a cache timeout is set using cacheTime to delete and garbage collect the query (defaults to 5 minutes).

<br><br>

만약 캐시 초과 시간이 완료되기 전에 `useQuery({ queryKey: ['todos'], queyFn: fetchTodos })` 의 다른 인스턴스가 마운트된다면 쿼리는 fetchTodos 함수가 백그라운드에서 실행되는 동안 사용 가능한 캐시된 데이터를 즉시 반환한다.   
성공적으로 완료되면 캐시를 새 데이터로 교체한다.

<br><br>

`useQuery({ queryKey: ['todos'], queryFn: fetchTodos })` 인스턴스가 이제 더는 사용되지 않고, `cacheTime`으로 설정한 5분이 지났다면  
`['todos']` 라는 쿼리 키 아래에 저장된 캐시 데이터는 삭제되고 가비지 콜렉터로 수집될 것이다.


<br>
---
참고
[Caching Examples](https://tanstack.com/query/v4/docs/guides/caching)