# 테이블 상속 (Table Inheritance)

## 1. 상속 개요

### 1.1 상속의 목적
- 데이터 모델 계층화
- 코드 재사용
- 데이터 구조화
- 쿼리 최적화

### 1.2 상속 예시
```sql
CREATE TABLE cities (
    name text,
    population real,
    elevation int
);

CREATE TABLE capitals (
    state char(2)
) INHERITS (cities);
```

## 2. 상속 특징

### 2.1 열 상속
- 부모 테이블의 모든 열 상속
- 추가 열 정의 가능
- 열 타입 일치 필요

### 2.2 제약 조건 상속
- CHECK 제약 조건 상속
- NOT NULL 제약 조건 상속
- 기본값 상속
- 인덱스는 상속되지 않음

### 2.3 다중 상속
```sql
CREATE TABLE holiday_cities (
    holiday_name text
) INHERITS (cities, holidays);
```

## 3. 상속 관리

### 3.1 상속 관계 추가
```sql
ALTER TABLE cities_europe 
INHERIT cities;
```

### 3.2 상속 관계 제거
```sql
ALTER TABLE cities_europe 
NO INHERIT cities;
```

### 3.3 상속 테이블 쿼리
```sql
-- 모든 도시 (상속 포함)
SELECT * FROM cities;

-- 특정 테이블만
SELECT * FROM ONLY cities;
```

## 4. 상속 주의사항

### 4.1 제약사항
- PRIMARY KEY 제약 조건 상속 불가
- UNIQUE 제약 조건 상속 불가
- FOREIGN KEY 제약 조건 상속 불가
- 트리거 상속 불가

### 4.2 성능 고려사항
- 쿼리 계획 복잡성
- 인덱스 사용 제한
- 데이터 변경 오버헤드

### 4.3 대안 고려
- 파티셔닝
- 뷰 사용
- 외래 키 관계

## 5. 상속 모범 사례

### 5.1 설계 지침
- 명확한 계층 구조
- 적절한 분할 기준
- 성능 영향 고려

### 5.2 유지보수
- 문서화
- 정기적인 검토
- 불필요한 상속 제거

### 5.3 쿼리 최적화
- 적절한 인덱스 사용
- 파티션 프루닝 활용
- 실행 계획 모니터링
