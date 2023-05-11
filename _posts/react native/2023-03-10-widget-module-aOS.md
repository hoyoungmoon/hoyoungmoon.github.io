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

AppWidgetManager를 상속받은 SimpleWidgetManager에는 onEnable, onDisabled, onReceive, onUpdate 메서드를 Override한다.

```java
public class SimpleWidgetManager extends AppWidgetProvider {

    static void updateAppWidget(Context context, AppWidgetManager appWidgetManager,
                                ComponentName componentName) {
        try {
            SharedPreferences sharedPref = context.getSharedPreferences("exampleKey", Context.MODE_PRIVATE);
            String appString = sharedPref.getString("widget-data", "{\"startDate\":\"no-data\",\"endDate\":\"no-data\"}");
            JSONObject widgetData = new JSONObject(appString);

            RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.widget_default);

            SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd");
            Date startDate = formatter.parse(widgetData.getString("startDate"));
            Date endDate = formatter.parse(widgetData.getString("endDate"));
            Date today = new Date();

            int dayInMillis = 24 * 60 * 60 * 1000;
            int ddays = (int) Math.floor((endDate.getTime() - today.getTime()) / dayInMillis) + 1;
            int countDays = (int) Math.floor((double) (today.getTime() - startDate.getTime()) / (double) dayInMillis) + 1;

            views.setTextViewText(R.id.start_date_text, widgetData.getString("startDate").replaceAll("-", ". "));
            views.setTextViewText(R.id.end_date_text, widgetData.getString("endDate").replaceAll("-", ". "));
            views.setTextViewText(R.id.d_day, "D" + (ddays > 0 ? "-" : "+") + Integer.toString(Math.abs(ddays)));
            views.setTextViewText(R.id.count_day, "D" + (countDays > 0 ? "+" : "-") + Integer.toString(Math.abs(countDays)));

            appWidgetManager.updateAppWidget(componentName, views);

            // reschedule the widget refresh
             AlarmHandler alarmHandler = new AlarmHandler(context);
             alarmHandler.cancelAlarmManager();
             alarmHandler.setAlarmManager();
        } catch (JSONException | ParseException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        updateAppWidget(context, appWidgetManager, new ComponentName(context, SimpleWidgetManager.class));
    }

    @Override
    public void onEnabled(Context context) {
        // Enter relevant functionality for when the first widget is created
        super.onEnabled(context);
        AlarmHandler alarmHandler = new AlarmHandler(context);
        alarmHandler.setAlarmManager();
    }

    @Override
    public void onDisabled(Context context) {
        super.onDisabled(context);
        AlarmHandler alarmHandler = new AlarmHandler(context);
        alarmHandler.cancelAlarmManager();
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        super.onReceive(context, intent);
        if (intent.getStringExtra(("mode")) != null && intent.getStringExtra(("mode")).equals("widget-update")) {
            AppWidgetManager manager = AppWidgetManager.getInstance(context);
            updateAppWidget(context, manager, new ComponentName(context, SimpleWidgetManager.class));
        }
    }
}
```
