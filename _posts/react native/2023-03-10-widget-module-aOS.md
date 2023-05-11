---
title: "React Native에서 위젯 구현하기 (2) - Android편"
date: 2023-05-10
categories: react-native
---

# React Native에서 위젯 구현하기 (2) - Android편

[React Native에서 위젯 구현하기 (1) - iOS편](https://hoyoungmoon.github.io/react-native/widget-module-iOS/)에서 처럼

1. 위젯 데이터를 공유할 수 있는 모듈 구현
2. 위젯 생성
3. 데이터 업데이트 로직 구현

## 위젯 데이터를 공유할 수 있는 모듈 구현

React Native의 javascript와 iOS, android 플랫폼별 네이티브 코드 사이에 위젯 관련 데이터를 공유할 수 있는 모듈이 필요하다. android에는 [SharedPreferences](https://developer.android.com/training/data-storage/shared-preferences?hl=ko), iOS에는 [NSUserDefault](https://developer.apple.com/documentation/foundation/nsuserdefaults)를 이용하여 앱 내에서 데이터를 공유할 수 있다.

Android의 경우 sharedPreference를 이용해 위젯 데이터를 저장할 수 있는 Native Module을 작성하였다. Native Module을 구현하는 방식은 [공식 문서](https://reactnative.dev/docs/native-modules-android)를 통해 확인할 수 있다.

```java
public class SharedStorage extends ReactContextBaseJavaModule {
    ReactApplicationContext context;

    public SharedStorage(ReactApplicationContext reactContext) {
        super(reactContext);
        context = reactContext;
    }

    @ReactMethod
    public void set(String data) {
        SharedPreferences.Editor editor = context.getSharedPreferences("exampleKey", Context.MODE_PRIVATE).edit();
        editor.putString("widgetData", data);
        editor.commit();

        // AppWidgetManager를 이용한 위젯 업데이트
        ...
    }
}
```

set 메서드에 AppWidgetManager를 이용하여 위젯을 갱신하는 로직을 `데이터 업데이트 로직 구현` 단계에서 추가할 예정이다.

## 위젯 생성

## 데이터 업데이트 로직 구현
