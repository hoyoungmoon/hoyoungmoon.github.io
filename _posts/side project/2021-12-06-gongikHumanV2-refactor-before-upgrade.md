---
title: "[공익인간 2.0] 1. 프로젝트 준비 및 리팩토링"
date: 2021-12-06
categories: javascript react-native side-project
---

# 프로젝트 준비 및 리팩토링

## 네이티브 안드로이드 React Native 마이그레이션

처음 프로젝트를 진행할 때는 자바를 이용하여 네이티브 안드로이드로 먼저 출시하였다. `공익인간2.0`의 경우 iOS와 같이 리액트 네이티브로 개발을 진행하는게 훨씬 효율적일 것이라 판단하여 **리액트 네이티브로 마이그레이션**을 진행하였다.

- **개발 및 배포**

1. 리액트 네이티브 안드로이드 개발 환경 설정

   리액트 네이티브 공식문서 [Android development environment](https://reactnative.dev/docs/environment-setup) 참고하여 진행

2. 사용 중인 라이브러리 안드로이드 환경 설정

   react-native-bootsplash, react-native-admob 등 써드파티 라이브러리 세팅 진행

3. 배포키 설정

   ```bash
   keytool -genkeypair -v -storetype PKCS12 -keystore my-upload-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
   ```

   네이티브 안드로이드에서는 .jks 형식의 키스토어를 생성하여 배포를 진행하였다. 리액트 네이티브의 경우 위와 같이 PKCS12 형식의 키스토어를 생성하여 진행하므로 기존의 키스토어를 아래의 명령어로 변환하여 사용하였다.

   ```bash
   keytool -importkeystore -srckeystore <your_key_name.keystore> -destkeystore <your_key_name.keystore> -deststoretype pkcs12
   ```

- **앱 내 세팅**

1. 기존 네이티브 안드로이드 유저 정보 마이그레이션

   기존 네이티브 사용자들(android)과 리액트 네이티브(iOS)의 SQLite database가 달랐다. 앱 진입시 네이티브 사용자들의 SQLite를 확인 후 데이터가 있는 경우 해당 데이터를 마이그레이션하여 유저 데이터를 유지하였다.

2. 프리미엄 구매 유저

   react-native-iap 라이브러리를 이용하여 purchaseHistories를 받고 구매내역이 있는 경우 프리미엄 고객으로 MainContext를 update하는 방식으로 진행하였다.

## 디자인 시스템

### 컴포넌트

<img width="342" alt="스크린샷 2022-01-05 오후 8 40 08" src="https://user-images.githubusercontent.com/53747019/148212510-ae6a370e-c09f-4081-ad6f-d904e32a3393.png">
<img width="342" alt="스크린샷 2022-01-05 오후 8 44 13" src="https://user-images.githubusercontent.com/53747019/148212522-c801951b-4a18-4051-bfae-40e3e0652c0a.png">

기존엔 공통 컴포넌트 없이 모든게 페이지마다 제각각 개별적으로 만들어서 진행하였다. Atomic Design Pattern을 나름대로 적용하여 진행하려고 노력하였다. 컴포넌트의 디자인은 토스 디자인을 많이 참고하였다.

### 다크모드

<img width="300" alt="스크린샷 2022-01-05 오후 8 25 13" src="https://user-images.githubusercontent.com/53747019/148210509-7cea2082-7d48-4b49-a9c3-7af2287ff566.png">
<img width="300" alt="스크린샷 2022-01-05 오후 8 25 24" src="https://user-images.githubusercontent.com/53747019/148210494-ac30f36e-780b-4137-8bd4-ea0938f76320.png">
<br/>
<br/>

사용자분들이 리뷰나 메일로 **다크모드** 지원을 부탁하였다. 디자인 시스템을 적용하면서 동시에 진행하면 좋을 것 같다고 생각하였다. [쏘카 컬러 시스템 구축 과정](https://tech.socarcorp.kr/design/2020/07/22/dark-mode-02.html)에 관련된 기술 블로그 글을 많이 참고하여 진행하였다.

## 상태 관리

기존에는 props drilling을 이용하여 유저, 휴가정보 상태와 변경을 props를 통해 전달하였다.
