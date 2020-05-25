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
 
**변경 후** 테이블

| id | name | mealCost | trafficCost | totalVac | **payDay** | **promotionDay** |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |

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
#### SQLite의 장단점 및 한계

#### android에서 사용할 수 있는 데이터 저장 방법들
1. SQLite
2. SharedPreference
3. File Storage
    - external storage
    - internal storage
