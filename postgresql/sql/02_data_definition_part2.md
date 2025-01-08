# 제약 조건 (Constraints) - Part 1

## 1. 제약 조건 개요

### 1.1 제약 조건의 목적
- 데이터 무결성 보장
- 데이터베이스 규칙 강제
- 잘못된 데이터 입력 방지

### 1.2 제약 조건 종류
- 열 제약 조건
- 테이블 제약 조건

## 2. Check 제약 조건

### 2.1 기본 사용법
```sql
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric CHECK (price > 0)
);
```

### 2.2 제약 조건 이름 지정
```sql
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric CONSTRAINT positive_price CHECK (price > 0)
);
```

## 3. Not-Null 제약 조건

### 3.1 기본 사용법
```sql
CREATE TABLE products (
    product_no integer NOT NULL,
    name text NOT NULL,
    price numeric
);
```

### 3.2 Null 허용
- NULL 제약 조건 사용 가능
- 기본적으로 모든 열은 NULL 허용

## 4. Unique 제약 조건

### 4.1 단일 열 Unique
```sql
CREATE TABLE products (
    product_no integer UNIQUE,
    name text,
    price numeric
);
```

### 4.2 다중 열 Unique
```sql
CREATE TABLE example (
    a integer,
    b integer,
    c integer,
    UNIQUE (a, c)
);
```

## 5. Primary Key

### 5.1 기본 사용법
```sql
CREATE TABLE products (
    product_no integer PRIMARY KEY,
    name text,
    price numeric
);
```

### 5.2 Primary Key 특징
- NOT NULL + UNIQUE 조합
- 테이블당 하나만 가능
- 테이블의 식별자 역할
- 자동 인덱스 생성
