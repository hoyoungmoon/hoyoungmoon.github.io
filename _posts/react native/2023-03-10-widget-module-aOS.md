---
title: "React Native에서 위젯 구현하기 (2) - Android편"
date: 2023-05-10
categories: react-native
---

[React Native에서 위젯 구현하기 (1) - iOS편](https://hoyoungmoon.github.io/react-native/widget-module-iOS/)에서 처럼

1. 위젯 데이터를 공유할 수 있는 모듈 구현
2. 위젯 생성
3. 데이터 업데이트 로직 구현

## 위젯 데이터를 공유할 수 있는 모듈 구현

React Native의 javascript와 iOS, android 플랫폼별 네이티브 코드 사이에 위젯 관련 데이터를 공유할 수 있는 모듈이 필요하다. android에는 [SharedPreferences](https://developer.android.com/training/data-storage/shared-preferences?hl=ko), iOS에는 [NSUserDefault](https://developer.apple.com/documentation/foundation/nsuserdefaults)를 이용하여 앱 내에서 데이터를 공유할 수 있다.

Android의 경우 SharedPreference를 이용해 위젯 데이터를 저장할 수 있는 Native Module을 작성하였다. iOS 위젯을 구현할 때 사용하였던 appGroupIdentifier와 동일한 키값(exampleKey)를 이용하여 SharedPreferences를 추가할 수 있는 editor를 생성 후 string 형태의 위젯 데이터를 추가 또는 수정할 수 있는 메서드를 추가하였다. Native Module을 구현하는 방식은 [공식 문서](https://reactnative.dev/docs/native-modules-android)를 통해 확인할 수 있다.

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

안드로이드의 [AlarmManager](https://developer.android.com/training/scheduling/alarms?hl=ko)를 통해 시스템 알람 서비스를 등록 후 특정 시간마다 Intent를 실행시켜 위젯을 업데이트할 수 있다. AlarmManager를 생성, 삭제할 수 있는 AlarmHandler 클래스를 먼저 작성하였다. 디데이 위젯을 특성상 정각에 한 번만 업데이트 되도록 AlarmManager를 등록하였다.

```java
public class AlarmHandler {
    private final Context context;

    public AlarmHandler(Context context) {
        this.context = context;
    }

    public void setAlarmManager() {
        Intent intent = new Intent(context, SimpleWidgetManager.class);
        intent.putExtra("mode", "widget-update");
        PendingIntent sender;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            sender = PendingIntent.getBroadcast(context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        } else {
            sender = PendingIntent.getBroadcast(context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
        }

        AlarmManager am = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
        Calendar c = Calendar.getInstance();
        c.add(Calendar.DATE, 1);
        c.set(Calendar.HOUR_OF_DAY, 0);
        c.set(Calendar.MINUTE, 1);
        c.set(Calendar.SECOND, 0);
        long l = c.getTimeInMillis();

        if (am != null) {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                am.setExactAndAllowWhileIdle(AlarmManager.RTC_WAKEUP, l, sender);
            } else {
                am.setExact(AlarmManager.RTC_WAKEUP, l, sender);
            }
        }
    }

    public void cancelAlarmManager() {
        Intent intent = new Intent(context, SimpleWidgetManager.class);
        intent.putExtra("mode", "widget-update");
        PendingIntent sender;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            sender = PendingIntent.getBroadcast(context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        } else {
            sender = PendingIntent.getBroadcast(context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
        }
        AlarmManager am = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
        if (am != null) {
            am.cancel(sender);
            sender.cancel();
        }
    }
}
```

위젯을 추가하면서 생성된 AppWidgetProvider 클래스에 updateAppWidget 메서드를 추가하였다. AlarmHandler에서 구현한 AlarmManager에 의해 해당 클래스가 실행되었을 때 updateAppWidget 메서드에 의해 위젯의 값이 업데이트되도록할 예정이다. 크게 3가지 단계로 업데이트가 이루어진다.

1. 위의 SharedStorage에 의해 SharedPreferences에 저장되어 있는 값을 가져오고 JSON 형태로 파싱
2. 날짜 계산 후 위젯 뷰 업데이트
3. 기존 AlarmManager를 취소 후 새로운 AlarmManager 설정

```java
public class SimpleWidgetManager extends AppWidgetProvider {

    static void updateAppWidget(Context context, AppWidgetManager appWidgetManager,
                                ComponentName componentName) {
        try {
            // 위젯 데이터 파싱
            SharedPreferences sharedPref = context.getSharedPreferences("exampleKey", Context.MODE_PRIVATE);
            String appString = sharedPref.getString("widget-data", "{\"startDate\":\"no-data\",\"endDate\":\"no-data\"}");
            JSONObject widgetData = new JSONObject(appString);

            RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.widget_default);

            SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd");
            Date startDate = formatter.parse(widgetData.getString("startDate"));
            Date endDate = formatter.parse(widgetData.getString("endDate"));
            Date today = new Date();

            // 날짜 계산
            int dayInMillis = 24 * 60 * 60 * 1000;
            int ddays = (int) Math.floor((endDate.getTime() - today.getTime()) / dayInMillis) + 1;
            int countDays = (int) Math.floor((double) (today.getTime() - startDate.getTime()) / (double) dayInMillis) + 1;

            views.setTextViewText(R.id.start_date_text, widgetData.getString("startDate").replaceAll("-", ". "));
            views.setTextViewText(R.id.end_date_text, widgetData.getString("endDate").replaceAll("-", ". "));
            views.setTextViewText(R.id.d_day, "D" + (ddays > 0 ? "-" : "+") + Integer.toString(Math.abs(ddays)));
            views.setTextViewText(R.id.count_day, "D" + (countDays > 0 ? "+" : "-") + Integer.toString(Math.abs(countDays)));

            appWidgetManager.updateAppWidget(componentName, views);
            // AlarmManager 재생성
             AlarmHandler alarmHandler = new AlarmHandler(context);
             alarmHandler.cancelAlarmManager();
             alarmHandler.setAlarmManager();
        } catch (JSONException | ParseException e) {
            e.printStackTrace();
        }
    }
    ...
}
```

AppWidgetManager를 상속받은 SimpleWidgetManager에서 Override된 onEnable, onDisabled, onReceive, onUpdate 메서드를 통해 업데이트 로직을 구현하였다.

```java
public class SimpleWidgetManager extends AppWidgetProvider {

    static void updateAppWidget(Context context, AppWidgetManager appWidgetManager,
                                ComponentName componentName) {
       ...
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
