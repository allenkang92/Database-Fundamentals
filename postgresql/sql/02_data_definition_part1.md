# 데이터 정의

## 1. 테이블 기본

### 1.1 테이블의 개념
- 행(row)과 열(column)로 구성
- 열의 수와 순서는 고정
- 각 열은 이름과 데이터 타입을 가짐
- 행의 수는 가변적 (데이터 양에 따라 변동)

### 1.2 데이터 타입
- 숫자형: integer, numeric, real
- 문자열: text, varchar, char
- 날짜/시간: date, time, timestamp
- 기타: boolean, enum, json
- 사용자 정의 타입 지원

### 1.3 테이블 생성
```sql
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric
);
```

## 2. 기본값 (Default Values)

### 2.1 기본값 설정
```sql
CREATE TABLE products (
    product_no integer DEFAULT nextval('products_product_no_seq'),
    name text,
    price numeric DEFAULT 9.99
);
```

### 2.2 기본값 타입
- 상수
- 함수 호출
- 표현식

## 3. 식별자 열 (Identity Columns)

### 3.1 식별자 열 정의
```sql
CREATE TABLE products (
    product_no integer GENERATED ALWAYS AS IDENTITY,
    name text,
    price numeric
);
```

### 3.2 식별자 속성
- ALWAYS: 사용자 지정 값 거부
- BY DEFAULT: 사용자 지정 값 허용
- 자동으로 NOT NULL 설정
- 시퀀스 타입만 사용 가능

## 4. 생성된 열 (Generated Columns)

### 4.1 생성된 열 정의
```sql
CREATE TABLE people (
    height_cm numeric,
    height_in numeric GENERATED ALWAYS AS (height_cm / 2.54) STORED
);
```

### 4.2 생성된 열 특징
- 다른 열을 기반으로 계산
- 읽기 전용
- STORED: 값을 물리적으로 저장
- 자동 업데이트
