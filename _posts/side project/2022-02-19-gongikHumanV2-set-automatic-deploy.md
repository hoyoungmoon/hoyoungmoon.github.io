---
title: "[공익인간 2.0] 3. 브랜치 전략 및 배포 자동화 구현"
date: 2022-02-19
categories: javascript react-native side-project
---

# **Github Actions**와 **Fastlane**으로 React Native 배포 자동화 구현하기

## **배경**

공익인간 1.0 에서는 크게 **master, develop, feature 브랜치**를 구성하여 개발하였고 배포 또한 메뉴얼하게 진행하였다.

공익인간 2.0 개발을 진행하면서 기존의 브랜치 플로우와 배포 방식에 아래와 같은 불편함이 발생하였다.

- **여러가지 feature 개발을 동시에 진행하고, 각각 테스트할 수 있는 환경 구성이 어렵다.**
  - 매번 develop 브랜치에 pull request를 한 다음 테스트 배포를 하였기 테스트가 끝나지 않은 브랜치들이 하나로 모일 수 있는 문제가 있었다.
- **안드로이드 또한 RN으로 마이그레이션이 완료되어 매번 테스트, 스토어 배포를 위한 빌드를 진행할 시 OS별로 2번씩 동일한 작업을 반복해야하는 효율성 문제가 있었다.**

<br/>

## **브랜치 전략**

### release 브랜치 추가

스토어 배포전 테스트를 위한 빌드를 진행할 때 develop에서 따로 release 브랜치로 checkout하는 방식을 추가하였다.

1. 개발 완료된 feature 브랜치 develop 브랜치로 pull reqeust & merge
2. develop 브랜치에서 "git branch checkout -b release/v${CURRENT_VERSION}"
3. release 브랜치에서 iOS TestFlight, android 내부테스트 빌드

### hotfix 브랜치 추가

release 브랜치와 마찬가지로 급하게 수정해야하는 사항을 위한 브랜치를 구분하였다.

- release 브랜치에서 테스트 빌드를 통해 최종적인 테스트 도중 문제가 발생한 경우
- 최악의 경우 스토어 배포가 완료된 후 문제가 발생한 경우

### master 브랜치에서 배포

release 브랜치에서 테스트 검토 완료 하였을 시,

1. release 브랜치 push to remote repo
2. master 브랜치에서 해당 release 브랜치 pull
3. 태그 생성 "git tag v{CURRENT_VERSION}"
4. 원격 저장소 푸시 "git push origin v{CURRENT_VERSION}"

<br/>

## **배포 자동화**
