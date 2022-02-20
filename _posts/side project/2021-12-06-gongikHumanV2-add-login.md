---
title: "[공익인간 2.0] 2. 회원가입 및 로그인 유지"
date: 2022-01-12
categories: javascript react-native side-project
---

앞서 기능 추가를 위한 리팩토링, 디자인 시스템을 추가 후

기본적인 기능을 위한 서버 구축과 클라이언트&서버 간 구조를 설정

## DB모델링

### 테이블

유저가 프리미엄 혜택을 받기 위해

1. 회원가입 및 로그인
2. 나의 복무지 등록
3. 일정 조건을 만족하는 리뷰 작성

위의 단계를 거쳐야 하므로 필요한 칼럼들을 구성하고 테이블간 관계를 구성하였다.

TODO: ERD 추가 및 칼럼 설명

크게 유저(user), 복무지(workPlace), 복무지에 대한 리뷰 및 댓글(workPlaceReview, workPlaceComment)을 구성하였다.(추가 에정: reviewReaction(도움되었어요 등..), community, communityComment)

## 회원가입

### OAuth

- 간단한 어플 유저 인증이므로 OAuth를 통해 빠르게 nickName, email, loginType 등 만을 이용하여 회원가입 후 accessToken, refreshToken을 반환

## 로그인 유지

### 서버 passport로 체크

- header에 전달된 accessToken을 바탕으로 해당 토큰을 가진 유저 정보 확인, 만료기간 지나지 않았는지 확인한다.
- 위의 조건 모두 만족시 다음 로직을 필요한 정보를 반환하는 middleware 역할

### 클라이언트 axios default header에 accessToken 관리

- 로그인시 default header에 accessToken을 저장하고 서버에서 유저 인증이 필요시 passport middleware를 통해 검증 하는 방식
- 로그아웃시 default header 제거, MainContext 로그인 유저 정보 제거

### 클라이언트 axios interceptor 등록

- default 토큰이 없을 시에는 앱 진입시 로그인 유저 정보 요청하지 않는다.
- 토큰이 있을시에는 현재 로그인된 유저의 정보를 요청한다.
  - 토큰이 만료되어 에러를 반환하면 axios interceptor에서 에러의 종류에 따라 토큰 리프레쉬를 요청할지 결정
  - 정상적인 토큰일시 해당 유저의 필요한 정보를 리턴하고 MainContext에 저장

### 유저별 리뷰 및 프리미엄 혜택 관리

- 프리미엄 구매 시 영구적으로 또는 리뷰 등록시 차등적으로 일정 기간 프리미엄(광고제거) 혜택을 준다.
  - 회원가입, 로그인 후 프리미엄을 구매할 수 있고 구매시 purchaseToken을 저장하여 플랫폼과 관계없이 프리미엄 적용될 수 있도록 한다.
  - 리뷰를 등록한 회원은 앱 진입시 리뷰의 createAt 칼럼과 특정 칼럼의 내용량에 따라 프리미엄 기간을 차등적으로 지원할 수 있는 api 구현
  - 만약 혜택을 위해 내용을 무성의하게 작성했을시 혜택 제외할 수 있는 칼럼 또는 로직 구성 필요
