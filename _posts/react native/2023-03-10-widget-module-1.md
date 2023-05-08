---
title: "[React Native] 위젯 구현하기"
date: 2023-03-10
categories: react-native
---

## 위젯 구현

위젯을 구현하기 위해 아래와 같이 크게 3단계로 작업하였다.

1. 플랫폼별 데이터를 네이티브 공유할 수 있는 모듈 구현
2. 데이터 통신 및 위젯 레이아웃 구현
3. 데이터 업데이트 로직 구현

### 플랫폼별 데이터를 네이티브 공유할 수 있는 모듈 구현

React Native의 javascript 코드와 iOS, android 플랫폼별 네이티브 코드 사이에 위젯 관련 데이터를 공유할 수 있는 모듈이 필요하다.

android에는 sharedPreference iOS에는 NSUserDefault를 이용하여 앱 내에서 데이터를 공유할 수 있다.

### 데이터 통신 및 위젯 레이아웃 구현

### 데이터 업데이트 로직 구현
