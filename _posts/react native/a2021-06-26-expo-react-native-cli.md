---
title: "expo cli vs react-native cli"
---

React Native를 이용하여 개발하기 위해 2가지 방법이 있다. 배포, 네이티브 모듈 사용 등에서 차이점이 있다.

## CLI (Command Line Interface)
**명령어 인터페이스(CLI, Command line interface)** 는 텍스트 터미널을 통해
사용자와 컴퓨터가 상호 작용하는 방식을 뜻한다. 즉, 작업 명령은 사용자가 컴퓨터 키보드 등을 통해 문자열의 형태로 입력하며, 컴퓨터로부터의 출력 역시 문자열의 형태로 주어진다.

## Expo CLI
>Expo CLI is a command line app that is the main interface between a developer and Expo tools.

Expo 공식 문서에 나와있듯 expo cli는 expo에서 제공하는 툴들과 개발자를 연결해주는 인터페이스에 해당한다. 주로 아래의 주요 목적으로 사용된다.

- 새 프로젝트 생성하고 시뮬레이터로 앱을 구동하여 개발 / 배포
- 앱스토어 또는 플레이스토어에 올릴 바이너리 (apk, ipa)파일 빌드
- Apple Credentials, Google Keystores 관리

등을 주요 목적으로 사용된다.

## React-native CLI
