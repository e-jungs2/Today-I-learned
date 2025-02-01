# advanced 2주차
## 1. 우리 플랫폼에 정착한 판매자1
### 문제
```olist_order_items_dataset 테이블에는 주문 안에 어떤 상품이 포함되어 있는지, 상품의 판매자는 누구인지 등 상품 단위의 데이터가 들어있습니다.

우리 플랫폼에서 상품을 많이 판매하고 있는 판매자가 누구인지 알고 싶습니다. 총 주문이 100건 이상 들어온 판매자 리스트를 출력하는 쿼리를 작성해주세요.

쿼리 결과에는 아래 컬럼이 있어야 합니다.

* seller_id - 판매자 ID
* orders - 판매자가 판매한 주문 건수
```

### 풀이
```sql
SELECT 
  seller_id,
  COUNT(DISTINCT order_id) AS orders
FROM olist_order_items_dataset
GROUP BY seller_id
HAVING COUNT(DISTINCT order_id) >= 100
ORDER BY orders DESC;
```
SELECT를 써 데이터를 불러옴.\
DISTINCT를 써 중복된 데이터를 제외하고 고유값만 가져와 수를 세 orders로 지정.\
FROM을 통해 데이터를 가져올 테이블 지정.\
GROUP BY를 통해 seller_id를 기준으로 그룹화 함.\
HAVING
