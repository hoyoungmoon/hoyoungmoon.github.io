---
title: "[SQL 코딩의 기술] Chapter 1. 데이터 모델 설계"
date: 2023-08-10
categories: database
---

### Better way 1. 모든 테이블에 기본키가 있는지 확인하자

- 관계형 데이터베이스 시스템은 한 테이블의 특정 로우와 나머지 로우를 구별할 수 있어야하므로 기본키(Primary key)가 필요하다.
  - 기본키는 유일해야하며 NULL을 허용하지 않아야한다.
- 기본키가 없어도 규칙에 어긋나지는 않지만 반복되는 데이터가 쌓여 테이블 간의 관계를 모델링하는 것이 어렵고 쿼리 수행이 느려진다.
- 자동으로 생성되는 숫자 값을 기본키로 사용하는 것이 가장 간단하다. 하지만 테이블 특성상 중복이 허용되지 않아야하는 텍스트 기반의 칼럼이 존재한다면 그것을 기본키로 사용해도 된다. 숫자, 텍스트에 관계 없이 핵심은 **기본키로 사용되는 칼럼은 반드시 유일한 값을 가져야한다는 것.**
- 복합 기본키(Compound Primary Key)는 가급적 사용하지 않는 것이 좋다. 이유는 아래와 같다.
  - 칼럼 두 개 이상이 기본키가 되면 각 칼럼에 대한 유일 인덱스를 만들게 되어 데이터베이스가 할 일이 많아진다.
  - 일반적으로 조인을 하는 경우 기본키를 이용하게 되는데 여러 칼럼으로 구성되는 경우 복잡해진다.
  - 하지만, 테이블 특성상 두 테이블의 값을 연결하는 등의 역할(ex. ProductID, VenderId 칼럼으로 이루어진 테이블)을 하는 경우 두 칼럼을 이용하여 복합 기본키를 구성하는게 유리할 수 있다.

### Better way 2. 중복으로 저장된 데이터 항목을 제거하자

- 데이터가 중복으로 저장되는 경우 비정상적인 CUD가 발생하는 등 문제를 일으킨다. 데이터 중복은 사용자가 동일한 데이터를 한 군데 이상에서 입력하는 상황에 가깝다.
  - 예를 들어, Customer 테이블을 정의하지 않고 SalesTransaction 테이블에 CustomerName 등의 칼럼으로 저장하는 방식
  - Customer의 이름이 변경된다면 SalesTransaction 테이블에 해당 CustomerName을 모두 교체해야하며 중복되는 CustomerName인지 구별할 수도 없다.
- 정규화의 한가지 목표는 한 데이터베이스 내에서 동일한 테이블이든 다른 테이블간에든 반복되는 데이터를 최소화하는 것이다.

### Better way 3. 반복 그룹을 제거하자

- 테이블의 비슷한 목적을 가진 여러개의 칼럼들의 반복으로 이루어져 있다면 주의해야한다. 테이블 구성이 ID, Question_1, Question_2, ..., Question_N와 같은 형태를 예로 들 수 있다. Question을 추가하기 위해선 칼럼을 추가해야하고 이는 로우를 추가하는 것보다 비용적인 측면에서 비싼 작업이 된다.
  - 어쩔 수 없이 테이블의 설계가 위와 같이 이루어져야한다면 UNION을 이용하여 데이터를 정규화할 수 있다.
    ```sql
        SELECT ID AS SurveyID, Question_1 as Question FROM Survey WHERE Question_1 IS NOT NULL
        UNION
        SELECT ID AS SurveyID, Question_2 as Question FROM Survey WHERE Question_2 IS NOT NULL
        UNION
        ...
        SELECT ID AS SurveyID, Question_N as Question FROM Survey WHERE Question_M IS NOT NULL
        ORDER BY SurveyID;
    ```

### Better way 4. 컬럼당 하나의 특성만 저장하자

- 해당 칼럼만의 개별 특성을 자체 칼럼에 할당한다. 하나의 칼럼에 여러 특성이 조합되어 있으면 검색, 그루핑이 어렵다.

### Better way 5. 왜 계산 데이터를 저장하면 좋지 않은지 이해하자

계산 칼럼을 가지는 방법

- 트리거
  - 칼럼 값의 삭제, 추가, 변경이 있을때마다 계산을 해주는 트리거를 발생시킨다. 하지만 비용이 비싸다.
- 테이블 생성시 계산 칼럼 정의

  ```sql
  CREATE FUNCTION dbo.getOrderTotal(@orderId int)
  RETURNS money
  AS
  BEGIN
    DECLARE @r money
    SELECT @r = SUM(Quantity * Price)
    FROM Order_Details WHERE OrderNumber = @orderId
    RETURN @r;
  END; GO
  CREATE TABLE Orders (
    OrderNumber int NOT NULL,
    OrderDate date NULL,
    ShipDate date NULL,
    CustomerID int NULL,
    EmployeeID int NULL,
    OrderTotal money AS dbo.getOrderTotal(OrderNumber)
  );
  ```

  따로 코드 작성하지 않아도 칼럼으로 보여주는 장점이 있다. 하지만 다른 테이블의 데이터에 의존하고 비결정적 함수이므로 인덱스 생성이 어렵다. 또한 칼럼 참조할 때마다 각 로우에 대해 함수 호출이 필요하므로 부하가 걸린다.

  비결정적 함수란 특정 값이 입력되었을 때 언제나 같은 결과를 반환하지 못하는 함수를 뜻한다. 예를 들어 GETDATE() 함수는 실행될 때마다 매번 다른 값이 반환된다.

- 결정적 함수 또는 표현식을 이용한 칼럼 정의
  ```sql
  -- 계산 칼럼 추가
  ALTER TABLE Order_Details
  ADD COLUMN ExtendedPrice decimal(15,2)
    GENERATED ALWAYS AS (QuantityOrdered * QuotedPrice);
  -- 인덱스 생성
  CREATE INDEX Order_Details_ExtendedPrice
  ON Order_Details (ExtendedPrice);
  ```
  위의 방법과는 다르게 언제나 같은 결과를 반환하므로 인덱스 생성 또한 가능하다.

하지만 위의 모든 방법들은 결국 데이터의 변경시 인덱스 값을 다시 저장하거나 select를 수행할 때 함수를 수행해야하는 등 부하가 걸리는 방법이 될 수 밖에 없다. 이런 단점을 감수하더라도 계산 칼럼을 정의했을때의 혜택이 큰 경우에만 설계하는 것이 좋다.

### Better way 6. 참조 무결성을 보호하려면 외래키를 정의하자.
