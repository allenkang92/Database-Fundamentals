# 테이블 파티셔닝 (Table Partitioning)

## 1. 파티셔닝 개요

### 1.1 파티셔닝의 목적
- 대용량 테이블 관리
- 쿼리 성능 향상
- 데이터 유지보수 용이성
- 효율적인 데이터 보관

### 1.2 파티셔닝 유형
- 범위 파티셔닝
- 리스트 파티셔닝
- 해시 파티셔닝

## 2. 선언적 파티셔닝

### 2.1 범위 파티셔닝
```sql
CREATE TABLE measurement (
    city_id int not null,
    logdate date not null,
    peaktemp int,
    unitsales int
) PARTITION BY RANGE (logdate);

CREATE TABLE measurement_y2020m01 PARTITION OF measurement
    FOR VALUES FROM ('2020-01-01') TO ('2020-02-01');
```

### 2.2 리스트 파티셔닝
```sql
CREATE TABLE sales (
    sale_id int not null,
    sale_date date not null,
    region text
) PARTITION BY LIST (region);

CREATE TABLE sales_west PARTITION OF sales
    FOR VALUES IN ('west', 'northwest');
```

### 2.3 해시 파티셔닝
```sql
CREATE TABLE orders (
    order_id int not null,
    user_id int not null,
    order_date date
) PARTITION BY HASH (user_id);

CREATE TABLE orders_0 PARTITION OF orders
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);
```

## 3. 파티션 관리

### 3.1 파티션 추가
```sql
CREATE TABLE measurement_y2020m02 PARTITION OF measurement
    FOR VALUES FROM ('2020-02-01') TO ('2020-03-01');
```

### 3.2 파티션 삭제
```sql
DROP TABLE measurement_y2020m01;
```

### 3.3 파티션 분리/결합
```sql
ALTER TABLE sales DETACH PARTITION sales_west;
ALTER TABLE sales ATTACH PARTITION sales_west
    FOR VALUES IN ('west', 'northwest');
```

## 4. 파티션 프루닝

### 4.1 개념
- 불필요한 파티션 스캔 방지
- 쿼리 플래너 최적화
- 자동 파티션 선택

### 4.2 제약 조건 제외
- 파티션 경계 조건
- CHECK 제약 조건
- 실행 계획 확인

## 5. 파티셔닝 모범 사례

### 5.1 파티션 키 선택
- 데이터 분포 고려
- 쿼리 패턴 분석
- 유지보수 용이성

### 5.2 파티션 크기
- 균형잡힌 크기
- 성능 최적화
- 관리 용이성

### 5.3 유지보수 전략
- 파티션 순환
- 오래된 데이터 관리
- 백업 및 복구

## 6. 서브파티셔닝

### 6.1 복합 파티셔닝
- 다중 기준 파티셔닝
- 계층적 구조
- 세분화된 데이터 관리

### 6.2 구현 예시
```sql
CREATE TABLE sales (
    sale_date date,
    region text,
    product_id int
) PARTITION BY RANGE (sale_date);

CREATE TABLE sales_2020 PARTITION OF sales
    FOR VALUES FROM ('2020-01-01') TO ('2021-01-01')
    PARTITION BY LIST (region);
```
