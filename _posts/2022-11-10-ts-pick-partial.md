---
title:  "타입스크립트 유틸리티와 Index Signature 사용하기"
categories: 
  - Typescript
tags:
  - Pick
  - Partial
  - Literal Types
  - Index Signature
toc: true
toc_sticky: true
toc_label: "타입스크립트 유틸리티"
share: true

---

타입스크립트(Typescript)의 유틸리티 타입(Utility Types)중 Pick, Partial을 사용해보자!  

### 문제

```typescript
export interface UserInfo {
  userId: number;
  email: string;
  name?: string;
  nickname?: string;
  phone?: string;
  createdAt: Date;
}
```
요런 형태로 구성된 객체가 있다.  
사용자가 Input에 검색어를 입력했을때 해당 검색어와 일치하는 필드 값을 찾기 위해 아래와 같은 함수를 생성했다.  

```typescript
const hasUserInfo = (filteredItem: UserInfo, searchTerm: string): boolean => {
  const keyList = ['name', 'nickname', 'email', 'phone'];

  return Object.keys(filteredItem).some((key: string) => {
    if (keyList.includes(key)) {
      return String(filteredItem[key]).toLowerCase().includes(searchTerm.toLowerCase());
    }

    return false;
  });
};
```

그랬더니 아래와 같이 타입스크립트 컴파일 오류가 발생했다!  

#### 컴파일 오류

`<html>TS7053: Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'UserInfo'.<br/>No index signature with a parameter of type 'string' was found on type 'UserInfo'.`

<br><br>

### 원인

#### `string literal` vs `string`

타입스크립트에서는 문자열 유형의 값으로 특정 키가 있는 객체의 프로퍼티에 접근하려고 하면 오류가 발생한다.  
이는 타입스크립트가 `string literal` 타입과 `string` 타입을 다르게 보기 때문이다.  

객체의 프로퍼티에 접근할때 `string literal`이 아닌, `string` 타입의 키로 접근하면 컴파일 오류가 발생한다.

```typescript
const a = 'hyungji';
const b: string = 'hyungji';
let c = 'hyungji';
```

a의 타입은 `string literal`이다.  
b의 타입은 `string` 이다.  
c의 타입도 `string` 이다.  
왜 이러한 차이가 생길까?  

변수 b는 명시적으로 `string`이라는 타입을 선언해주었다.  
변수 c는 `let` 키워드를 사용하여 선언했기 때문에 변수 값의 재할당이 가능하므로 컴파일러는 변수 c의 타입을 `string`으로 추론한다.  
마찬가지로 만약 아래와 같이 선언했다면 변수 c의 타입은 `number`로 추론되었을 것이다.
```typescript
let c = 123;
```

<br>

마지막으로 변수 a는 그 타입이 `string literal` 이다.  
`string literal`은 `string`보다 더 구체적으로 좁혀진 타입(narrowed type)을 의미한다.  

변수 a는 `const` 키워드로 변수 선언을 했기 때문에 값이 재할당되지 않는다.   
따라서, 변수 a에는 문자열에 해당하는 값이 무한대로 재할당될 수 없고, 변수 선언시 할당해준 `'hyungji'` 라는 값만을 가질 수 있다.  

타입스크립트는 이에 대해 `string` 타입보다 더 구체적인, `string`과 `'hyungji'`의 부분집합인 `'hyungji'` 만을 허용하는 타입을 선언한 것으로 간주한다.  
참고로, 이렇게 `Literal Types`으로 간주될 수 있는 타입에는 `string`, `number`, `boolean`이 있다.

<br>

### Index Signature


이제 다시 인터페이스 `UserInfo`와 코드 예제를 살펴보자!  

```typescript
export interface UserInfo {
  [index: string]: string;
  userId: number;
  email: string;
  name?: string;
  nickname?: string;
  phone?: string;
  createdAt: Date;
}
```


처음에는 위와 같이 Index Signature를 추가해서 사용하는 방법을 생각했다.  
그런데 index signature를 추가하더라도 인터페이스 `UserInfo`의 다른 필드들은 Index Signature를 따르지 않기 때문에 또 다른 오류가 발생했다.   
<br>

즉, 인터페이스 `UserInfo` 의 다른 필드 타입도 고려하여 `string`이 아닌, 다른 타입으로 선언해주어야 했다.  



### 해결
#### Pick
타입스크립트의 유틸리티 Pick 타입은 선택한 특정한 요소만을 포함하는 타입으로 정의한다.  
예를 들어,   
```typescript
type requiredUserInfo = Pick<UserInfo, "userId" | "createdAt" >;
```

이렇게 선언해주면 인터페이스 UserInfo에서 `userId`, `createdAt` 속성만 가져온다.

#### Partial
타입스크립틑의 유틸리티 Partial 타입은 특정 타입에서 부분 집합을 만족하는 타입을 정의할 수 있다.  
```typescript
type requiredUserInfo = Partial<UserInfo>;
```
위와 같이 타입 `requiredUserInfo`을 선언하면 해당 타입은 인터페이스 `UserInfo`의 모든 필드를 optional한 타입으로 지정해준 것과 같아진다.   
결과적으로 아래 코드와 같다!

```typescript
export interface UserInfo {
  userId?: number;
  email?: string;
  name?: string;
  nickname?: string;
  phone?: string;
  createdAt?: Date;
}
```


<br>
Pick 타입과 Partial 타입의 속성을 활용해서 인터페이스 `UserInfo`를 아래와 같이 구성해주었다.  

```typescript
export interface UserInfo {
  [index: string]: Pick<UserInfo, Partial<UserInfo>>;
  userId?: number;
  email?: string;
  name?: string;
  nickname?: string;
  phone?: string;
  createdAt?: Date;
}
```
이렇게 하면 `UserInfo` 의 모든 필드를 옵셔널한 타입으로 지정하고, 선택한 필드만 가지고 새로운 타입으로 정의할 수 있게 된다.  

```typescript
const hasUserInfo = (filteredItem: UserInfo, searchTerm: string): boolean => {
  const keyList = ['name', 'nickname', 'email', 'phone'];

  return Object.keys(filteredItem).some((key: string) => {
    if (keyList.includes(key)) {
      return String(filteredItem[key]).toLowerCase().includes(searchTerm.toLowerCase());
    }

    return false;
  });
};
```

<br>

### 정리
각 필드의 타입이 `Date`, `number` 등등 다양할때 `Pick` 타입과 `Partial` 타입으로 Index Signature를 사용할 수 있었다!

---
참고 <br>
[Literal Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types) <br>
[Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html#handbook-content) <br>
[TypeScript에서 string key로 객체에 접근하기](https://soopdop.github.io/2020/12/01/index-signatures-in-typescript/) <br>
[유틸리티 타입](https://kyounghwan01.github.io/blog/TS/fundamentals/utility-types/#partial)