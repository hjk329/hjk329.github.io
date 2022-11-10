---
title:  "타입스크립트 유틸리티"
categories: 
  - Typescript
tags:
  - 유틸리티
  - Pick
  - Partial
toc: true
toc_sticky: true
toc_label: "타입스크립트 유틸리티"
share: true
published: false

---

타입스크립트(Typescript)의 유틸리티 타입(Utility Types)중 Pick, Partial을 사용해보자 :)  

```javascript
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

`<html>TS7053: Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'UserInfo'.<br/>No index signature with a parameter of type 'string' was found on type 'UserInfo'.`

<br><br>

위와 같이 컴파일 오류가 발생한 이유는 
