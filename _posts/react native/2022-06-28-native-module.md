---
title: "[React Native] 네이티브 모듈"
date: 2022-06-28
categories: react-native
---

# 네이티브 모듈

가끔 React Native에서 제공하지 않은 네이티브 API 예를 들어 Apple, Google Play에 접근해야하는 네이티브 api가 필요할 때가 있다. 또는 높은 퍼포먼스나 멀티 스레드 환경의 코드를 위해 Objective-C, Swift, Java 또는 C++로 이미 작성되어진 라이브러리를 JS로 다시 구현하지 않고 사용할 수 있어야 한다. **NativeModule 시스템**은 이러한 네이티브 코드를 JS 객체로 변환하여 JS내에서 네이티브 코드를 실행할 수 있도록 해준다.

React Native에서 네이티브 모듈을 구성하는 방법은 2가지가 있다.

1. React Native 프로젝트 내의 ios, android 폴더에 직접적으로 작성하는 방법
2. dependency로 프로젝트 내에 설치할 수 있는 NPM 패키지를 구성하는 방법
