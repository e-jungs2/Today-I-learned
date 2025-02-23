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
