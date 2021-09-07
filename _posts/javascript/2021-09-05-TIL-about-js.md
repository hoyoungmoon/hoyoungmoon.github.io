---
title: "TIL about JS"
date: 2021-09-05
categories: javascript
---

오늘 배운 JS에 관련한 기록

## TIL about JS

- javascript에 관하여 배운 것 아무거나 기록
- 나중에 분류 & 따로 포스팅

### 코딩테스트 관련 기록
- 문자열 split으로 char 하나씩 비교가능
```javascript
anyString.split('').map((c, i)=>{...}) 
```

- 2차원 배열 행, 열 바꾸기
  - 배열 할당해서 하는 방법
  
  Array.from은 array-like, iterable한 object들을 얕게 복사하여 새로운 배열을 생성해준다.
  > 유사배열객체(array-like object)의 조건
  > 1. 반드시 length가 있어야한다.
  > 2. index가 0부터 1씩 증가해야한다. 그렇지 않으면 예상치 못한 오류 발생할 수 있다.
  
  ```javascript
  let arr = Array.from(new Array(5), v=>new Array(5))
  originalArr.map((row, i)=>{
    row.map((data, j)=>{
      arr[j][i]=data;
    }
  }
  ```
  - 할당하지 않고 하는 방법
  ```javascript
  const arr = originalArr.map((_, i)=>{
    return originalArr.map(v=>v[i]);
  })
