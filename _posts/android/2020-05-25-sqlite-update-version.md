---
title: "[Android]  SQLite로 db table 업그레이드"
date: 2020-05-25 22:31:28 -0400
categories: android java SQLite
---
## [Android]  SQLite로 db table 업그레이드

### 업그레이드 방법

앱을 업데이트 하면서 user db에 2개의 columns를 추가해야하는 상황이 생겼다.
```
ALTER TABLE {tableName} ADD COLUMN {newColumn} {type};
```
당연히 위 sql문을 통해 간단히 할 수 있을 줄 알았지만, 문제가 있었다.
**업데이트 도중 user db의 version을 관리하지 않아서 다양한 종류의 table이 version 1로 통일 되어있는 상황이였다.**

따라서 어떤 종류의 table을 가지든 user table을 삭제하고 다시 만들기로 하였다.
순서는

- 임시 user table 생성
- 원하는 user data만 기존 user table에서 임시 table로 복사
- 기존 user table 삭제
- 4개의 칼럼이 추가된 새로운 user table 생성
- 새로운 user table에 임시 table의 data 복사
- 임시 user table 삭제


### 코드

**변경 전** 테이블

| id | name | mealCost | trafficCost | totalVac | extraColumn... | ... |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| INTEGER | TEXT | INTEGER | INTEGER | INTEGER | TYPE... | ... |
 
**변경 후** 테이블

| id | name | mealCost | trafficCost | totalVac | payDay | promotionDay |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| INTEGER | TEXT | INTEGER | INTEGER | INTEGER | INTEGER | INTEGER |

다양한 user table에 대하여 실행을 하였을 때 정상적으로 업데이트가 되는 것을 확인하였다. 더 나은 방법이나
오류가 확인되면 수정을 하여야겠다. 오류에 대해 말씀해주시면 감사하겠습니다 :)


#### 전체 코드
```java
public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        if (oldVersion < 2) {
            try {
                db.beginTransaction();
                db.execSQL("CREATE TEMPORARY TABLE TABLE_TEMPORARY_USER (" + TABLE_USER_COLUMNS_VER_1 + ");");
                db.execSQL("INSERT INTO " + TABLE_TEMPORARY_USER + " SELECT id, name ,  mealCost , trafficCost ,  totalVac FROM " + TABLE_USER + ";");
                db.execSQL("DROP TABLE TABLE_USER;");
                db.execSQL("CREATE TABLE " + TABLE_USER + "(" + TABLE_USER_COLUMNS_VER_2 + " );");
                db.execSQL("INSERT INTO " + TABLE_USER + "(id, name ,  mealCost , trafficCost ,  totalVac) SELECT * FROM " + TABLE_TEMPORARY_USER + ";");
                db.execSQL("DROP TABLE TABLE_TEMPORARY_USER;");
                db.setTransactionSuccessful();
            } catch (IllegalStateException e) {

            } finally {
                db.endTransaction();
            }
        }
    }
```

### 추가적인 내용
#### SQLite의 장단점
##### 장점
  - 어떤 os든 (android, ios포함) 쉽게 이용이 가능하다.
##### 단점
  - 데이터 형식은 비교적 소수이기 때문에 다양한 데이터 형식 지정이 필요한 앱에 적합하지 않다. 예를 들어 기본 datetime 형식이 없다. 따라서 애플리케이션에서 이러한 형식을 처리해야 한다. 애플리케이션이 아닌 데이터베이스에서 datetime 값을 위한 입력을 정규화하고 제약하고자 한다면 SQLite는 적합하지 않다.
  - SQLite 인스턴스는 단일체이며 독립적이라 동기화 기능을 제공하지 않는다. 수평확장 설계를 하는 프로그램에 적합하지 않다.


#### Android에서 사용할 수 있는 데이터 저장 방법들
1. SQLite
2. SharedPreference
간단한 정보들을 key-value형태로 저장할 때 주로 사용한다.
3. File Storage
    - external storage
    - internal storage
