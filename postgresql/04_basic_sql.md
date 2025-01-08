# PostgreSQL 기본 SQL

## 소개
이 장에서는 SQL을 사용하여 간단한 작업을 수행하는 방법에 대한 개요를 제공합니다. 이 튜토리얼은 입문을 목적으로 하며 SQL에 대한 완전한 튜토리얼이 아닙니다. SQL에 관한 수많은 책이 저술되었으며, 그중에는 [melt93]과 [date97]이 포함됩니다. 일부 PostgreSQL 언어 기능은 표준의 확장임을 유의해야 합니다.

## 1. SQL 기초

### 1.1 개념
- SQL이란?
- SQL 문법 기초
- 데이터 타입

### 1.2 기본 명령어
- SELECT
- INSERT
- UPDATE
- DELETE

## 2. 테이블 작업

### 2.1 테이블 생성
```sql
CREATE TABLE weather (
    city        varchar(80),
    temp_lo     int,           -- 최저 기온
    temp_hi     int,           -- 최고 기온
    prcp        real,          -- 강수량
    date        date
);
```

### 2.2 테이블 수정
- 컬럼 추가/삭제
- 데이터 타입 변경
- 제약 조건 추가/삭제

## 3. 데이터 조작

### 3.1 데이터 삽입
```sql
INSERT INTO weather VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');
```

### 3.2 데이터 조회
```sql
SELECT * FROM weather;
SELECT city, temp_lo, temp_hi, prcp, date FROM weather;
```

### 3.3 데이터 수정
```sql
UPDATE weather SET temp_hi = temp_hi - 2 WHERE city = 'San Francisco';
```

### 3.4 데이터 삭제
```sql
DELETE FROM weather WHERE city = 'San Francisco';
```

## 4. 조건과 필터링

### 4.1 WHERE 절
- 비교 연산자
- 논리 연산자
- LIKE 연산자

### 4.2 정렬과 그룹화
- ORDER BY
- GROUP BY
- HAVING

## 5. 조인

### 5.1 조인 유형
- INNER JOIN
- LEFT JOIN
- RIGHT JOIN
- FULL OUTER JOIN

### 5.2 조인 예제
```sql
SELECT w.city, w.temp_lo, w.temp_hi,
       c.location, c.altitude
FROM weather w, cities c
WHERE w.city = c.name;
```

## 6. 집계 함수

### 6.1 기본 집계
- COUNT()
- SUM()
- AVG()
- MAX()
- MIN()

### 6.2 그룹별 집계
```sql
SELECT city, count(*), max(temp_hi)
FROM weather
GROUP BY city
HAVING count(*) > 1;
```

## 7. 테이블 쿼리
테이블에서 데이터를 검색하려면 테이블을 쿼리합니다. SQL SELECT 문을 사용하여 이를 수행합니다. 이 문은 반환될 열을 나열하는 선택 목록, 데이터를 검색할 테이블을 나열하는 테이블 목록, 선택적 조건(제한)을 포함합니다. 예를 들어, weather 테이블의 모든 행을 검색하려면 다음을 입력하십시오:

```sql
SELECT * FROM weather;
```

여기서 *는 "모든 열"의 약어입니다. 따라서 동일한 결과는 다음과 같이 얻을 수 있습니다:

```sql
SELECT city, temp_lo, temp_hi, prcp, date FROM weather;
```

출력 결과는 다음과 같습니다:

```
     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 San Francisco |      43 |      57 |    0 | 1994-11-29
 Hayward       |      37 |      54 |      | 1994-11-29
(3 rows)
```

선택 목록에는 단순한 열 참조뿐만 아니라 표현식을 사용할 수 있습니다. 예를 들어:

```sql
SELECT city, (temp_hi+temp_lo)/2 AS temp_avg, date FROM weather;
```

이는 다음과 같은 결과를 제공합니다:

```
     city      | temp_avg |    date
---------------+----------+------------
 San Francisco |       48 | 1994-11-27
 San Francisco |       50 | 1994-11-29
 Hayward       |       45 | 1994-11-29
(3 rows)
```

AS 절이 출력 열의 레이블을 변경하는 방법을 주목하십시오. (AS 절은 선택 사항입니다.)

쿼리는 WHERE 절을 추가하여 "자격을 부여"할 수 있으며, 이는 원하는 행을 지정합니다. WHERE 절은 불리언(참/거짓) 표현식을 포함하며, 이 표현식이 참인 행만 반환됩니다. 일반적인 불리언 연산자(AND, OR, NOT)가 자격에 허용됩니다. 예를 들어, 다음은 비오는 날의 San Francisco 날씨를 검색합니다:

```sql
SELECT * FROM weather
    WHERE city = 'San Francisco' AND prcp > 0.0;
```

결과:

```
     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
(1 row)
```

쿼리 결과를 정렬된 순서로 반환하도록 요청할 수 있습니다:

```sql
SELECT * FROM weather
    ORDER BY city;
```

     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 Hayward       |      37 |      54 |      | 1994-11-29
 San Francisco |      43 |      57 |    0 | 1994-11-29
 San Francisco |      46 |      50 | 0.25 | 1994-11-27

이 예제에서는 정렬 순서가 완전히 지정되지 않았으므로 San Francisco 행이 어떤 순서로든 나타날 수 있습니다. 그러나 다음과 같이 하면 항상 위에 표시된 결과를 얻을 수 있습니다:

```sql
SELECT * FROM weather
    ORDER BY city, temp_lo;
```

쿼리 결과의 중복 행을 제거하도록 요청할 수 있습니다:

```sql
SELECT DISTINCT city
    FROM weather;
```

     city
---------------
 Hayward
 San Francisco
(2 rows)

결과 행의 순서도 변할 수 있습니다. DISTINCT와 ORDER BY를 함께 사용하여 일관된 결과를 보장할 수 있습니다:

```sql
SELECT DISTINCT city
    FROM weather
    ORDER BY city;
```

## 8. 테이블 간 조인
지금까지 우리의 쿼리는 한 번에 하나의 테이블에만 액세스했습니다. 쿼리는 한 번에 여러 테이블에 액세스하거나 동일한 테이블에 여러 번 액세스하여 테이블의 여러 행을 동시에 처리할 수 있습니다. 한 번에 여러 테이블(또는 동일한 테이블의 여러 인스턴스)에 액세스하는 쿼리는 조인 쿼리라고 합니다. 이는 한 테이블의 행을 두 번째 테이블의 행과 결합하고, 어느 행을 짝지을지 지정하는 표현식을 사용합니다. 예를 들어, 모든 날씨 기록과 관련된 도시의 위치를 함께 반환하려면, weather 테이블의 각 행의 city 열을 cities 테이블의 모든 행의 name 열과 비교하고, 이 값이 일치하는 행 쌍을 선택해야 합니다. 이는 다음 쿼리로 수행됩니다:

```sql
SELECT * FROM weather JOIN cities ON city = name;
```

     city      | temp_lo | temp_hi | prcp |    date    |     name      | location
---------------+---------+---------+------+------------+---------------+-----------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27 | San Francisco | (-194,53)
 San Francisco |      43 |      57 |    0 | 1994-11-29 | San Francisco | (-194,53)
(2 rows)

결과 집합에 대해 두 가지를 주목하십시오:

Hayward 도시의 결과 행이 없습니다. 이는 cities 테이블에 Hayward에 대한 일치하는 항목이 없기 때문에 weather 테이블의 일치하지 않는 행을 조인이 무시하기 때문입니다. 이 문제를 해결하는 방법을 곧 보게 될 것입니다.

두 테이블에 도시 이름을 포함하는 두 개의 열이 있습니다. 이는 weather와 cities 테이블의 열 목록이 연결되었기 때문에 올바른 것입니다. 실제로 이는 바람직하지 않으므로, *를 사용하는 대신 출력 열을 명시적으로 나열하는 것이 좋습니다:

```sql
SELECT city, temp_lo, temp_hi, prcp, date, location
    FROM weather JOIN cities ON city = name;
```

열 이름이 모두 다르기 때문에 파서는 자동으로 어떤 테이블에 속하는지 찾았습니다. 두 테이블에 중복 열 이름이 있는 경우에는 어떤 열을 의미하는지 보여주기 위해 열 이름을 한정해야 합니다. 예를 들어:

```sql
SELECT weather.city, weather.temp_lo, weather.temp_hi,
       weather.prcp, weather.date, cities.location
    FROM weather JOIN cities ON weather.city = cities.name;
```

조인 쿼리의 모든 열 이름을 한정하는 것은 좋은 스타일로 간주되므로, 나중에 테이블 중 하나에 중복 열 이름이 추가되어도 쿼리가 실패하지 않습니다.

지금까지 본 조인 쿼리는 다음과 같은 형식으로 작성할 수도 있습니다:

```sql
SELECT *
    FROM weather, cities
    WHERE city = name;
```

이 구문은 SQL-92에서 도입된 JOIN/ON 구문보다 이전의 것입니다. FROM 절에 테이블을 단순히 나열하고, 비교 표현식을 WHERE 절에 추가합니다. 이전의 암시적 구문과 새로운 명시적 JOIN/ON 구문의 결과는 동일합니다. 그러나 쿼리를 읽는 사람에게는 명시적 구문이 의미를 더 쉽게 이해할 수 있게 해줍니다: 조인 조건이 자신의 키워드로 도입되는 반면, 이전에는 조건이 다른 조건과 함께 WHERE 절에 혼합됩니다.

이제 Hayward 기록을 다시 가져오는 방법을 알아보겠습니다. 원하는 것은 weather 테이블을 스캔하고 각 행에 대해 일치하는 cities 행을 찾는 것입니다. 일치하는 행이 없으면 cities 테이블의 열에 대해 일부 "빈 값"을 대체하고자 합니다. 이러한 종류의 쿼리는 외부 조인이라고 합니다. (지금까지 본 조인은 내부 조인입니다.) 명령은 다음과 같습니다:

```sql
SELECT *
    FROM weather LEFT OUTER JOIN cities ON weather.city = cities.name;
```

     city      | temp_lo | temp_hi | prcp |    date    |     name      | location
---------------+---------+---------+------+------------+---------------+-----------
 Hayward       |      37 |      54 |      | 1994-11-29 |               |
 San Francisco |      46 |      50 | 0.25 | 1994-11-27 | San Francisco | (-194,53)
 San Francisco |      43 |      57 |    0 | 1994-11-29 | San Francisco | (-194,53)
(3 rows)

이 쿼리는 왼쪽 외부 조인이라고 하며, 조인 연산자의 왼쪽에 언급된 테이블의 각 행이 최소한 한 번은 출력에 포함되도록 합니다. 반면 오른쪽 테이블은 왼쪽 테이블의 일부 행과 일치하는 행만 출력됩니다. 오른쪽 테이블과 일치하는 행이 없는 왼쪽 테이블의 행을 출력할 때는 오른쪽 테이블의 열에 대해 빈(null) 값이 대체됩니다.

연습 문제: 오른쪽 외부 조인과 전체 외부 조인도 있습니다. 이들이 무엇을 하는지 알아보십시오.

테이블을 자체적으로 조인할 수도 있습니다. 이는 자기 조인이라고 합니다. 예를 들어, 다른 날씨 기록의 온도 범위 내에 있는 모든 날씨 기록을 찾고자 한다고 가정합시다. 따라서 각 weather 행의 temp_lo와 temp_hi 열을 다른 모든 weather 행의 temp_lo와 temp_hi 열과 비교해야 합니다. 이를 다음 쿼리로 수행할 수 있습니다:

```sql
SELECT w1.city, w1.temp_lo AS low, w1.temp_hi AS high,
       w2.city, w2.temp_lo AS low, w2.temp_hi AS high
    FROM weather w1 JOIN weather w2
        ON w1.temp_lo < w2.temp_lo AND w1.temp_hi > w2.temp_hi;
```

     city      | low | high |     city      | low | high
---------------+-----+------+---------------+-----+------
 San Francisco |  43 |   57 | San Francisco |  46 |   50
 Hayward       |  37 |   54 | San Francisco |  46 |   50
(2 rows)

여기서는 weather 테이블을 w1과 w2로 레이블을 지정하여 조인의 왼쪽과 오른쪽을 구분할 수 있도록 했습니다. 이러한 별칭은 쿼리에서 다른 곳에서도 사용할 수 있어 타이핑을 줄일 수 있습니다. 예를 들어:

```sql
SELECT *
    FROM weather w JOIN cities c ON w.city = c.name;
```

이 약식 스타일을 자주 접하게 될 것입니다.

## 9. 집계 함수
대부분의 다른 관계형 데이터베이스 제품과 마찬가지로, PostgreSQL도 집계 함수를 지원합니다. 집계 함수는 여러 입력 행으로부터 단일 결과를 계산합니다. 예를 들어, 집계 함수에는 집합의 행 수(count), 합계(sum), 평균(avg), 최대값(max), 최소값(min)을 계산하는 함수가 있습니다.

예를 들어, 어디에서나 가장 높은 낮은 온도 기록을 찾을 수 있습니다:

```sql
SELECT max(temp_lo) FROM weather;
```

 max
-----
  46
(1 row)

해당 기록이 발생한 도시(또는 도시들)를 알고 싶다면 다음과 같이 시도할 수 있습니다:

```sql
SELECT city FROM weather WHERE temp_lo = max(temp_lo);     잘못됨
```

그러나 이는 집계함수 max를 WHERE 절에서 사용할 수 없기 때문에 작동하지 않습니다. (이 제한은 WHERE 절이 집계 계산에 포함될 행을 결정해야 하기 때문에 존재합니다. 따라서 집계 함수는 계산되기 전에 평가되어야 합니다.) 그러나 종종 원하는 결과를 얻기 위해 쿼리를 다시 작성할 수 있습니다. 여기서는 서브쿼리를 사용하여 다음과 같이 할 수 있습니다:

```sql
SELECT city FROM weather
    WHERE temp_lo = (SELECT max(temp_lo) FROM weather);
```

     city
---------------
 San Francisco
(1 row)

이는 서브쿼리가 외부 쿼리와 별개로 자체 집계를 계산하기 때문에 가능합니다.

집계는 또한 GROUP BY 절과 결합하여 매우 유용합니다. 예를 들어, 각 도시에서 관측된 읽기 수와 최대 낮은 온도를 얻을 수 있습니다:

```sql
SELECT city, count(*), max(temp_lo)
    FROM weather
    GROUP BY city;
```

     city      | count | max
---------------+-------+-----
 Hayward       |     1 |  37
 San Francisco |     2 |  46
(2 rows)

이는 각 도시에 대해 하나의 출력 행을 제공합니다. 각 집계 결과는 해당 도시에 일치하는 테이블 행에 대해 계산됩니다. 이러한 그룹화된 행을 HAVING을 사용하여 필터링할 수 있습니다:

```sql
SELECT city, count(*), max(temp_lo)
    FROM weather
    GROUP BY city
    HAVING max(temp_lo) < 40;
```

  city   | count | max
---------+-------+-----
 Hayward |     1 |  37
(1 row)

이는 모든 temp_lo 값이 40 미만인 도시에 대해 동일한 결과를 제공합니다. 마지막으로, 이름이 "S"로 시작하는 도시만 관심이 있다면 다음과 같이 할 수 있습니다:

```sql
SELECT city, count(*), max(temp_lo)
    FROM weather
    WHERE city LIKE 'S%'            -- (1)
    GROUP BY city;
```

     city      | count | max
---------------+-------+-----
 San Francisco |     2 |  46
(1 row)

(1)

LIKE 연산자는 패턴 매칭을 수행하며 섹션 9.7에서 설명합니다.

집계와 SQL의 WHERE 및 HAVING 절 간의 상호 작용을 이해하는 것이 중요합니다. WHERE와 HAVING의 근본적인 차이점은 다음과 같습니다: WHERE는 그룹과 집계가 계산되기 전에 입력 행을 선택합니다(따라서 집계 계산에 포함될 행을 제어합니다), 반면 HAVING은 그룹과 집계가 계산된 후 그룹 행을 선택합니다. 따라서 WHERE 절은 집계 함수를 포함할 수 없으며 집계에 포함될 행을 결정하기 위해 집계를 사용하는 것은 의미가 없습니다. 반면 HAVING 절은 항상 집계 함수를 포함합니다. (엄밀히 말하면 집계를 사용하지 않는 HAVING 절을 작성할 수 있지만, 이는 거의 유용하지 않습니다. 동일한 조건을 WHERE 단계에서 더 효율적으로 사용할 수 있습니다.)

이전 예제에서 도시 이름 제한은 집계를 필요로 하지 않기 때문에 WHERE에서 적용할 수 있습니다. 이는 WHERE 검사를 통과하지 못하는 모든 행에 대해 그룹화 및 집계 계산을 피할 수 있으므로 HAVING에 제한을 추가하는 것보다 더 효율적입니다.

집계 계산에 포함될 행을 선택하는 또 다른 방법은 FILTER를 사용하는 것입니다. 이는 집계별 옵션입니다:

```sql
SELECT city, count(*) FILTER (WHERE temp_lo < 45), max(temp_lo)
    FROM weather
    GROUP BY city;
```

     city      | count | max
---------------+-------+-----
 Hayward       |     1 |  37
 San Francisco |     1 |  46
(2 rows)

FILTER는 WHERE와 유사하지만, 특정 집계 함수에만 행을 제거합니다. 여기서 count 집계는 temp_lo가 45 미만인 행만 계산합니다. 그러나 max 집계는 여전히 모든 행에 대해 적용되므로 여전히 46을 찾습니다.

## 10. 업데이트
UPDATE 명령을 사용하여 기존 행을 업데이트할 수 있습니다. 예를 들어, 11월 28일 이후의 모든 온도 기록이 2도씩 낮다고 발견한 경우 데이터를 다음과 같이 수정할 수 있습니다:

```sql
UPDATE weather
    SET temp_hi = temp_hi - 2,  temp_lo = temp_lo - 2
    WHERE date > '1994-11-28';
```

데이터의 새 상태를 확인하십시오:

```sql
SELECT * FROM weather;
```

     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 San Francisco |      41 |      55 |    0 | 1994-11-29
 Hayward       |      35 |      52 |      | 1994-11-29
(3 rows)

## 11. 삭제
DELETE 명령을 사용하여 테이블에서 행을 제거할 수 있습니다. 예를 들어, 더 이상 Hayward의 날씨에 관심이 없는 경우 다음과 같이 해당 행을 테이블에서 삭제할 수 있습니다:

```sql
DELETE FROM weather WHERE city = 'Hayward';
```

Hayward에 속한 모든 날씨 기록이 제거됩니다.

```sql
SELECT * FROM weather;
```

     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 San Francisco |      41 |      55 |    0 | 1994-11-29
(2 rows)

다음과 같은 형식의 문에 주의하십시오:

```sql
DELETE FROM tablename;
```

조건 없이 DELETE를 실행하면 주어진 테이블의 모든 행이 제거되어 테이블이 비게 됩니다. 시스템은 이를 수행하기 전에 확인을 요청하지 않습니다!