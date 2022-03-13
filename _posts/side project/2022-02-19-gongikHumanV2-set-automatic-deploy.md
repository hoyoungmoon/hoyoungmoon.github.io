---
title: "[공익인간 2.0] 3. 배포 자동화 구현 및 브랜치 workflow 수정"
date: 2022-02-19
categories: javascript react-native side-project
---

# Github Actions와 Fastlane으로 배포 자동화 구현하기

## **배경**

공익인간 1.0 에서는 크게 `master`, `develop`, `feature` 브랜치를 구성하여 개발하였고 배포 또한 메뉴얼하게 진행하였다.

공익인간 2.0 개발을 진행하면서 기존의 브랜치 플로우와 배포 방식에 아래와 같은 불편함이 발생하였다.

- **여러가지 feature 개발을 동시에 진행하고, 각각 테스트할 수 있는 환경 구성이 어렵다.**
  - 매번 develop 브랜치에 pull request를 한 다음 테스트 배포를 하였기 테스트가 끝나지 않은 브랜치들이 하나로 모일 수 있는 문제가 있었다.
- **안드로이드 또한 RN으로 마이그레이션이 완료되어 매번 테스트, 스토어 배포를 위한 빌드를 진행할 시 OS별로 2번씩 동일한 작업을 반복해야하는 효율성 문제가 있었다.**

<br/>

## **브랜치 전략**

TODO: Feature Branch Workflow에서 Gitflow Workflow로 변경하는 것 설명

### release 브랜치

스토어 배포전 테스트를 위한 빌드를 진행할 때 `develop`에서 따로 `release` 브랜치로 checkout하는 방식을 추가하였다.

1. 개발 완료된 `feature` 브랜치 `develop` 브랜치로 pull reqeust & merge
2. `develop` 브랜치에서 "git branch checkout -b release/v${CURRENT_VERSION}"
3. `release` 브랜치에서 Fastlane을 통한 iOS TestFlight, android 내부테스트 빌드

```
git push origin feature/{branch_name} & pull request into develop branch

git push checkout develop
git pull origin develop

git branch checkout -b release/v{CURRENT_VERSION}
fastlane build beta
```

### hotfix 브랜치

`release` 브랜치와 마찬가지로 급하게 수정해야하는 사항을 위한 브랜치를 구분하였다.

- `release` 브랜치에서 테스트 빌드를 통해 최종적인 테스트 도중 문제가 발생한 경우
- 최악의 경우 스토어 배포가 완료된 후 문제가 발생한 경우

### master 브랜치에서 배포

`release` 브랜치에서 테스트 검토 완료 하였을 시,

1. `release` 브랜치 push to remote repo
2. `master` 브랜치에서 해당 release 브랜치 pull
3. 태그 생성 "git tag v{CURRENT_VERSION}"
4. 원격 저장소 푸시 "git push origin v{CURRENT_VERSION}"
5. Github Actions, Fastlane을 통한 스토어 배포

```
git push origin release/v{CURRENT_VERSION}

git checkout master
git pull origin release/v{CURRENT_VERSION}

git tag v{CURRENT_VERSION}
git push origin v{CURRENT_VERSION}

<!-- github actions에서 진행 -->
fastlane build prod
```

<br/>

## **배포 자동화**

위에서 계획한 테스트 빌드와 스토어 배포 과정을 자동화하기 위해 github actions와 Fastlane을 사용하였다.

> ### **Fastlane**
>
> - Ruby 기반 클라이언트 자동 빌드 오픈소스 라이브러리이다.
> - iOS, android 배포 자동화를 위해 사용하였다.
>
> ### **Github Actions**
>
> - Github에서 제공하는 CI/CD 툴로 테스트, 빌드, 배포 등 개발 프로세스를 자동화할 수 있다.
> - master branch에 tag와 함께 push 되었을 때 자동으로 fastlane을 실행하기 위해 사용하였다.

### **Fastlane 설정**

```ruby
default_platform(:ios)

platform :ios do
  # TestFlight 배포 및 테스트
  desc "Push a new beta build to TestFlight"
  lane :beta do
    increment_build_number(xcodeproj: "iosAgentApp.xcodeproj")
    build_app(workspace: "iosAgentApp.xcworkspace", scheme: "iosAgentApp")
    upload_to_testflight(skip_waiting_for_build_processing: true)
  end

  # 배포 전 빌드, 버전 increament
  desc 'Version increament'
  lane :version do |options|
    updateVersion(options)
    increment_build_number(xcodeproj: 'iosAgentApp.xcodeproj')
  end

  # github actions를 통한 스토어 배포
  desc 'GitHub actions release'
  lane :github do |_options|
    create_keychain(
    #   name: 'distribution_ios_keychain',
    #   password: 'Alleyoops1402',
      timeout: 1800,
      default_keychain: true,
      unlock: true,
      lock_when_sleeps: false
    )
    import_certificate(
    #   certificate_path: '../certifications/distribution_ios_keychain.p12',
    #   certificate_password: 'Alleyoops1402',
    #   keychain_name: 'distribution_ios_keychain',
    #   keychain_password: 'Alleyoops1402'
    )
    install_provisioning_profile(path: '../certifications/distribution_ios.mobileprovision')
    update_project_provisioning(
      xcodeproj: 'iosAgentApp.xcodeproj',
      target_filter: 'github',
      profile: 'distribution_ios.mobileprovision',
      build_configuration: 'Release'
    )
    api_key = app_store_connect_api_key(
    #   key_id: 'QA9HNUZ732',
    #   issuer_id: 'e0741a70-9331-49e8-98cc-29086c11df7a',
    #   key_filepath: '../certifications/distribution_ios_api_key.p8'
    )

    build_app(workspace: 'iosAgentApp.xcworkspace', scheme: 'iosAgentApp')
    upload_to_app_store(
      force: true,
      reject_if_possible: true,
      skip_metadata: false,
      skip_screenshots: true,
      languages: ['ko'],
      release_notes: {
        'default' => 'bug fixed',
        'ko' => 'bug fixed'
      },
      submit_for_review: true,
      precheck_include_in_app_purchases: false,
      automatic_release: true,
      submission_information: {
        add_id_info_uses_idfa: true,
        add_id_info_serves_ads: true,
        add_id_info_tracks_install: true,
        add_id_info_tracks_action: false,
        add_id_info_limits_tracking: true,
        export_compliance_encryption_updated: false
      },
      api_key: api_key
    )
  end
end
```

### **Github Actions 설정**

프로젝트 레포지토리의 actions 탭에서 workflow를 생성하면 자동으로 프로젝트에 .github/workflows/build.yml 위치로 yml 파일이 추가된다.

```javascript
name: build
on:
  push:
    tags:
      - "v*"
jobs:
  release-ios:
    name: Build and release iOS app
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v1
        with:
          node-version: "10.x"
      - uses: actions/setup-ruby@v1
        with:
          ruby-version: "2.x"
      - name: Install Fastlane
        run: cd ios && bundle install && cd ..
      - name: Install packages
        run: npm install
      - name: Install pods
        run: cd ios && pod install && cd ..
      - name: Execute Fastlane command
        run: cd ios && fastlane github
  release-android:
    ...
```
