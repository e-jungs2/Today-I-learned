# advanced 8주차
## Pivot과 Unpivot 정리
### 1. Pivot
Pivot은 데이터를 행에서 열로 변환하는 과정. 주로 집계 함수와 함께 사용되어, 특정 기준에 따라 데이터를 요약함함.

예시:

원본 데이터:

날짜	카테고리	판매량
2025-01-01	A	10
2025-01-01	B	20
2025-01-02	A	15
2025-01-02	B	25
Pivot 후 데이터:

날짜	A	B
2025-01-01	10	20
2025-01-02	15	25
SQL 예시:

```sql
SELECT 
    날짜,
    SUM(CASE WHEN 카테고리 = 'A' THEN 판매량 ELSE 0 END) AS A,
    SUM(CASE WHEN 카테고리 = 'B' THEN 판매량 ELSE 0 END) AS B
FROM 
    판매데이터
GROUP BY 
    날짜;
```

### 2. Unpivot
Unpivot은 데이터를 열에서 행으로 변환하는 과정. 주로 여러 열에 분산된 데이터를 하나의 열로 통합할 때 사용됨.

예시:

원본 데이터:

날짜	A	B
2025-01-01	10	20
2025-01-02	15	25
Unpivot 후 데이터:

날짜	카테고리	판매량
2025-01-01	A	10
2025-01-01	B	20
2025-01-02	A	15
2025-01-02	B	25
SQL 예시:

```sql
SELECT 
    날짜,
    카테고리,
    판매량
FROM 
    (SELECT 날짜, A, B FROM 판매데이터) AS SourceTable
UNPIVOT
    (판매량 FOR 카테고리 IN (A, B)) AS UnpivotedTable;
```

#### 결론
Pivot과 Unpivot은 데이터 분석에서 매우 유용한 도구. 데이터를 원하는 형식으로 변환하여, 보다 쉽게 인사이트를 도출할 수 있음. VSCode에서 SQL 쿼리를 작성하거나, 데이터 분석을 위한 스크립트를 작성할 때 이 개념들을 활용할 수 있음.

### 문제 풀이
![
!\[alt text\](image.png)](image/8주차/1.png)
오류 뜸..

# SQL 쿼리 성능 최적화를 위한 6가지 팁

## 1. 왼쪽 연산을 사용하지 마라
### 결과:
왼쪽(코드에서 왼쪽에 위치) 연산을 사용하면 인넷스를 활용할 수 없고, 반대로 전체 데이터를 통상 조회해야 함함.

**반대**
```sql
SELECT * FROM sales WHERE YEAR(date) = 2021;
```

**조회 성능 바이트**
```sql
SELECT * FROM sales WHERE date >= '2021-01-01' AND date <= '2021-12-31';
```

---
## 2. OR 대시 UNION 사용
### 결과:
`OR` 연산자는 인넷스 활용에 문제를 일으키며, `UNION`을 이용하면 더 훨씬 보다 효율적인 조회가 가능함함.

**반대**
```sql
SELECT * FROM employees WHERE department = 'Marketing' OR department = 'IT';
```

**조회 성능 바이트**
```sql
SELECT * FROM employees WHERE department = 'Marketing'
UNION
SELECT * FROM employees WHERE department = 'IT';
```

---
## 3. 필요한 Row, Column만 선택
### 결과:
`SELECT *` 는 비필요한 데이터를 가지고 오기 때문에 성능이 엄청나게 떨어짐.

**반대**
```sql
SELECT * FROM employees;
```

**조회 성능 바이트**
```sql
SELECT name, email FROM employees WHERE department = 'Marketing' AND sales >= 100000;
```

---
## 4. 분석 함수(Analytic Functions) 활용
### 결과:
`ROW_NUMBER()`, `RANK()`, `LEAD()`, `LAG()` 등의 분석 함수를 활용하면 간단한 수행에서 보다 성능이 훨씬 높아지며, 복수 확인에 효과적임.

**예제**
```sql
SELECT
  name, department, salary,
  ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
FROM employees;
```

---
## 5. LIKE 반응에서 가로(% 사용) 위치 조절
### 결과:
`가로(%)` 사용은 보유한 반응에 영향을 미칠 수 있으며, 원본 데이터를 통상 반영해야 함.

**반대**
```sql
SELECT * FROM users WHERE name LIKE '%John';
```

**조회 성능 바이트**
```sql
SELECT * FROM users WHERE name LIKE 'John%';
```

---
## 6. 계산값은 미리 저장한다.
### 결과:
자주 사용되는 전체 계산값을 미리 계산하고 경제 한복으로 환율하여 데이터 조회에 적용할 수 있음.

**예제**
```sql
CREATE TABLE product_stats AS
SELECT product_id, AVG(price) AS avg_price, SUM(price) AS total_sales
FROM orders GROUP BY product_id;

SELECT * FROM product_stats WHERE product_id = 101;
```
## 문제
```sql
UPDATE customers c
SET
    avg_order_value = (
        SELECT AVG(od.quantity * od.unit_price)
        FROM order_details od
        JOIN orders o ON od.order_id = o.order_id
        WHERE o.customer_id = c.customer_id
    ),
    total_spent = (
        SELECT SUM(od.quantity * od.unit_price)
        FROM order_details od
        JOIN orders o ON od.order_id = o.order_id
        WHERE o.customer_id = c.customer_id
    ),
    num_orders = (
        SELECT COUNT(DISTINCT o.order_id)
        FROM orders o
        WHERE o.customer_id = c.customer_id
    ),
    repurchase_rate = (
        SELECT
            COUNT(DISTINCT CASE WHEN od.product_id IN (
                SELECT product_id
                FROM order_details
                JOIN orders o ON order_details.order_id = o.order_id
                WHERE o.customer_id = c.customer_id
                GROUP BY order_details.product_id
                HAVING COUNT(order_details.product_id) > 1
            ) THEN od.product_id END) * 1.0 / COUNT(DISTINCT od.product_id)
        FROM order_details od
        JOIN orders o ON od.order_id = o.order_id
        WHERE o.customer_id = c.customer_id
    );
    ```
    절대.. 못할 듯.....
