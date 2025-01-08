# 제약 조건 (Constraints) - Part 2

## 1. Foreign Key 제약 조건

### 1.1 기본 사용법
```sql
CREATE TABLE orders (
    order_id integer PRIMARY KEY,
    product_no integer REFERENCES products (product_no),
    quantity integer
);
```

### 1.2 다중 열 Foreign Key
```sql
CREATE TABLE order_items (
    product_no integer,
    order_id integer,
    quantity integer,
    FOREIGN KEY (product_no, order_id) 
    REFERENCES products_orders (product_no, order_id)
);
```

### 1.3 참조 동작
- RESTRICT: 참조되는 행 삭제 방지
- CASCADE: 연관된 행도 함께 삭제
- SET NULL: NULL로 설정
- SET DEFAULT: 기본값으로 설정
- NO ACTION: 나중에 검사

## 2. Exclusion 제약 조건

### 2.1 기본 개념
- 지정된 연산자로 비교했을 때 모든 행 쌍이 참이 되지 않도록 보장
- 특수한 인덱스 타입 사용

### 2.2 사용 예시
```sql
CREATE TABLE circles (
    c circle,
    EXCLUDE USING gist (c WITH &&)
);
```

### 2.3 적용 분야
- 시간 간격 중복 방지
- 공간 데이터 중복 방지
- 복잡한 비즈니스 규칙 적용

## 3. 제약 조건 관리

### 3.1 제약 조건 추가
```sql
ALTER TABLE products 
ADD CONSTRAINT positive_price 
CHECK (price > 0);
```

### 3.2 제약 조건 삭제
```sql
ALTER TABLE products 
DROP CONSTRAINT positive_price;
```

### 3.3 제약 조건 비활성화/활성화
```sql
ALTER TABLE products 
ALTER CONSTRAINT positive_price 
DEFERRABLE INITIALLY DEFERRED;
```

## 4. 제약 조건 모범 사례

### 4.1 명명 규칙
- 의미있는 이름 사용
- 일관된 접두사/접미사
- 제약 조건 유형 표시

### 4.2 성능 고려사항
- 인덱스 자동 생성 이해
- 제약 조건 검사 시점
- 지연된 제약 조건 활용
