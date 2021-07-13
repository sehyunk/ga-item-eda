# GA 샘플데이터 제품별 분석

- bigquery-public-data의 ga데이터를 활용하였습니다.
- 해당 샘플 데이터를 빅쿼리를 이용해서 추출하고

- 파이썬으로 불러와 카테고리별, 제품별, 일자별 분석을 하였습니다.



## 빅쿼리로 데이터 추출

```sql
# 날짜, 카테고리, 제품이름
select date, prod.v2ProductCategory as category,prod.v2ProductName as name, 
		# 제품 구매수량 합
    sum(prod.productQuantity) as qty , 
    # 제품 구매수량 * 제품가격의 합 = 총 구매 금액
    sum(prod.productPrice*prod.productQuantity) as sales
from `bigquery-public-data.google_analytics_sample.ga_sessions_*`
		# hits array 풀기
    left join unnest(hits) as hits
    # 매출 집계를 위해 product 풀기
    left join unnest(hits.product) as prod
# 7월 데이터 필터링
where _table_suffix between '20170701' and '20170801'
		# '구매완료' 필터링
    and hits.eCommerceAction.action_type = '6'
    # unnest시 생긴 다른 행 카운트 하지 않기 위해 넣은 조건
    and prod.productQuantity is not null
# 날짜, 카테고리, 이름으로 그룹바이
group by date, prod.v2ProductCategory, prod.v2ProductName
```



## 판다스로 데이터 분석

1. 데이터 불러오기
2. 전처리
3. 카테고리 별 제품 별 분석
   - 인사이트
4. 일자 별 분석
   - 인사이트