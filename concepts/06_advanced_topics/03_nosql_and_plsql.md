# NoSQL과 PL/SQL

## 1. NoSQL (Not Only SQL)

### 1.1 NoSQL의 개념
- 비관계형 데이터베이스
- 유연한 스키마
- 수평적 확장성
- 대용량 데이터 처리에 적합

### 1.2 NoSQL의 유형
1. 문서형 (Document)
   - MongoDB, CouchDB
   - JSON 형태의 문서 저장

2. 키-값형 (Key-Value)
   - Redis, DynamoDB
   - 단순한 키-값 쌍으로 데이터 저장

3. 컬럼형 (Column-Family)
   - Cassandra, HBase
   - 컬럼 기반 저장

4. 그래프형 (Graph)
   - Neo4j, OrientDB
   - 노드와 관계로 데이터 저장

## 2. PL/SQL

### 2.1 PL/SQL의 개념
- Oracle's Procedural Language extension to SQL
- SQL에 프로그래밍 기능을 추가한 절차적 언어

### 2.2 기본 구문
```sql
CREATE [OR REPLACE] PROCEDURE procedure_name
(
    parameter1 [IN|OUT|IN OUT] datatype,
    parameter2 [IN|OUT|IN OUT] datatype,
    ...
)
IS
    -- 지역변수 선언
BEGIN
    -- 실행문
EXCEPTION
    -- 예외 처리
END;
```

### 2.3 주요 특징
1. 블록 구조
   - 선언부 (Declarative)
   - 실행부 (Executable)
   - 예외처리부 (Exception)

2. 변수와 상수
   - 스칼라 변수
   - 레코드 타입
   - 컬렉션 타입

3. 제어문
   - IF-THEN-ELSE
   - LOOP
   - WHILE
   - FOR

4. 커서 처리
   - 명시적 커서
   - 암시적 커서

### 2.4 장점
1. 성능 향상
2. 모듈화
3. 재사용성
4. 보안성
