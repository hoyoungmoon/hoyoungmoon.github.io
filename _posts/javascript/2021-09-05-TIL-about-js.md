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
    ```
  
- 배열의 최댓값, 최솟값 구하기
  
  Math.max(n1, n2, n3 ...) 이런식으로 값을 하나씩 넣어주어야한다.
  - 방법1
    ```javascript
    Math.max.apply(null, numberArray)
    ```
    > apply, call, bind
    
    Array의 기본 메소드 bind는 함수가 가리키는 this를 변환해주는 역할.
    
    apply, call은 bind의 역할 + 함수를 실행.
    
    call은 값을 하나씩, apply는 배열(+유사배열)로 한 번에 할당 가능.
    
    유사배열과 같이 배열의 메소드(join, slice, concat ...)를 쓸 수 없는 경우 아래와 같이 사용 가능하다.
    ```javascript
    Array.prototype.join.call(null, someArrayLike)
    ```
    
  - 방법2
    ```javascript
    Math.max(...numberArray)
    ```

- Map 정렬하기
  Map 생성자에 스프레드 또는 Array.from을 이용하여 배열 형태로 변환하고 sort로 정렬 후 다시 Map을 생성한다.
  ```javascript
  // sort by key
  const sortedMap = new Map([...unsortedMap.entries()].sort());
  // sort by value
  const sortedMap = new Map([...unsortedMap.entries()].sort((a, b)=> a[1] - b[1]));
  ```
