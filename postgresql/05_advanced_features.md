### 뷰(Views)

#### 뷰의 개념
뷰는 가상 테이블로서의 뷰, 복잡한 쿼리의 단순화, 데이터 접근 제어를 제공합니다.

#### 뷰 생성과 사용
```sql
CREATE VIEW myview AS
    SELECT name, temp_lo, temp_hi, prcp, date, location
        FROM weather, cities
        WHERE city = name;

SELECT * FROM myview;
```

### 외래 키(Foreign Keys)

#### 참조 무결성
외래 키는 데이터 일관성 보장, 부모-자식 관계 관리, 삭제/수정 시 동작 정의를 제공합니다.

#### 외래 키 제약조건
```sql
CREATE TABLE cities (
        name     varchar(80) primary key,
        location point
);

CREATE TABLE weather (
        city      varchar(80) references cities(name),
        temp_lo   int,
        temp_hi   int,
        prcp      real,
        date      date
);
```

### 트랜잭션(Transactions)

#### ACID 속성
트랜잭션은 원자성 (Atomicity), 일관성 (Consistency), 격리성 (Isolation), 지속성 (Durability)을 제공합니다.

#### 트랜잭션 제어
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100.00
    WHERE name = 'Alice';
UPDATE branches SET balance = balance - 100.00
    WHERE name = (SELECT branch_name FROM accounts WHERE name = 'Alice');
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Bob';
UPDATE branches SET balance = balance + 100.00
    WHERE name = (SELECT branch_name FROM accounts WHERE name = 'Bob');
COMMIT;
```

### 윈도우 함수(Window Functions)

#### 기본 개념
윈도우 함수는 파티션 내에서의 계산, 행 간의 관계 분석, 집계와의 차이점을 제공합니다.

#### 사용 예제
```sql
SELECT depname, empno, salary, avg(salary) OVER (PARTITION BY depname)
FROM empsalary;
```

### 상속(Inheritance)

#### 테이블 상속
상속은 상속 관계 정의, 데이터 모델링 확장, 쿼리 처리 방식을 제공합니다.

#### 구현 예제
```sql
CREATE TABLE cities (
        name       text,
        population real,
        elevation  int     -- 해발고도(피트 단위)
);

CREATE TABLE capitals (
        state      char(2)
) INHERITS (cities);
```

### 결론
PostgreSQL에는 이 튜토리얼에서 다루지 않은 많은 기능이 있으며, 이는 SQL의 초보 사용자에게 중점을 둔 소개입니다. 이러한 기능은 이 책의 나머지 부분에서 더 자세히 논의됩니다.

더 많은 입문 자료가 필요하다고 생각되면 PostgreSQL 웹사이트를 방문하여 추가 자료를 확인하십시오.