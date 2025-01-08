# 테이블 관리

## 1. 시스템 열 (System Columns)

### 1.1 기본 시스템 열
- oid: 객체 식별자
- tableoid: 테이블의 OID
- xmin: 삽입 트랜잭션 ID
- cmin: 삽입 명령 ID
- xmax: 삭제 트랜잭션 ID
- cmax: 삭제 명령 ID
- ctid: 물리적 행 위치

### 1.2 시스템 열 사용
- 시스템 정보 조회
- 버전 관리
- 성능 최적화

## 2. 테이블 수정

### 2.1 열 추가
```sql
ALTER TABLE products 
ADD COLUMN description text;

ALTER TABLE products 
ADD COLUMN description text DEFAULT 'None';
```

### 2.2 열 삭제
```sql
ALTER TABLE products 
DROP COLUMN description;

ALTER TABLE products 
DROP COLUMN description CASCADE;
```

### 2.3 열 이름 변경
```sql
ALTER TABLE products 
RENAME COLUMN product_no TO product_id;
```

### 2.4 열 데이터 타입 변경
```sql
ALTER TABLE products 
ALTER COLUMN price TYPE numeric(10,2);
```

### 2.5 기본값 변경
```sql
ALTER TABLE products 
ALTER COLUMN price SET DEFAULT 7.77;

ALTER TABLE products 
ALTER COLUMN price DROP DEFAULT;
```

## 3. 테이블 관리 작업

### 3.1 테이블 이름 변경
```sql
ALTER TABLE products 
RENAME TO items;
```

### 3.2 테이블 소유자 변경
```sql
ALTER TABLE products 
OWNER TO new_owner;
```

### 3.3 테이블 스키마 변경
```sql
ALTER TABLE products 
SET SCHEMA new_schema;
```

### 3.4 테이블 삭제
```sql
DROP TABLE products;
DROP TABLE IF EXISTS products CASCADE;
```

## 4. 테이블 유지보수

### 4.1 VACUUM
- 삭제된 행 정리
- 공간 회수
- 통계 업데이트

### 4.2 ANALYZE
- 통계 정보 수집
- 쿼리 최적화 지원

### 4.3 CLUSTER
- 인덱스 기반 물리적 재정렬
- 성능 최적화
