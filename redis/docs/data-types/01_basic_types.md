Redis 데이터 타입 가이드

## 1. 문자열 (Strings)

### 1.1 개요
- 가장 기본적인 데이터 타입
- 텍스트나 직렬화된 객체 저장
- 최대 512MB까지 저장 가능

### 1.2 사용 예제
```bash
# 기본 문자열 조작
SET user:1:name "John Doe"
GET user:1:name

# 숫자 조작
SET counter 100
INCR counter
INCRBY counter 50

# 만료시간 설정
SETEX session:123 3600 "session_data"
```

## 2. 리스트 (Lists)

### 2.1 개요
- 순서가 있는 문자열 컬렉션
- 양방향 큐로 사용 가능
- 메시지 큐나 최근 항목 관리에 적합

### 2.2 사용 예제
```bash
# 리스트 조작
LPUSH mylist "first"
RPUSH mylist "last"
LRANGE mylist 0 -1

# 큐 구현
LPUSH queue "job1"
RPOP queue
```

## 3. 집합 (Sets)

### 3.1 개요
- 순서가 없는 유일한 문자열 컬렉션
- 멤버십 테스트에 최적화
- 교집합, 합집합 등 집합 연산 지원

### 3.2 사용 예제
```bash
# 집합 조작
SADD myset "item1" "item2"
SMEMBERS myset
SISMEMBER myset "item1"

# 집합 연산
SINTER set1 set2
SUNION set1 set2
```

## 4. 정렬된 집합 (Sorted Sets)

### 4.1 개요
- 점수로 정렬된 유일한 문자열 멤버
- 실시간 순위표에 적합
- 범위 검색 효율적

### 4.2 사용 예제
```bash
# 점수와 함께 멤버 추가
ZADD leaderboard 100 "player1"
ZADD leaderboard 200 "player2"

# 순위 조회
ZRANK leaderboard "player1"
ZRANGE leaderboard 0 -1 WITHSCORES
```

## 5. 해시 (Hashes)

### 5.1 개요
- 필드-값 쌍의 컬렉션
- 객체 표현에 적합
- 메모리 효율적

### 5.2 사용 예제
```bash
# 해시 조작
HSET user:1 name "John" age "30"
HGET user:1 name
HGETALL user:1

# 다중 필드 설정
HMSET user:2 name "Jane" age "25" city "Seoul"
```

## 6. 비트맵 (Bitmaps)

### 6.1 개요
- 비트 단위 연산 지원
- 메모리 효율적인 불리언 정보 저장
- 사용자 활동 추적에 유용

### 6.2 사용 예제
```bash
# 비트 설정
SETBIT online:users 123 1
GETBIT online:users 123

# 비트 카운팅
BITCOUNT online:users
```

## 7. HyperLogLog

### 7.1 개요
- 고유 항목 개수 추정
- 매우 적은 메모리 사용
- 정확도는 0.81% 오차

### 7.2 사용 예제
```bash
# 요소 추가
PFADD visitors user1 user2 user3
PFCOUNT visitors
```

## 8. 스트림 (Streams)

### 8.1 개요
- 추가 전용 로그 데이터 구조
- 시계열 데이터에 적합
- 메시지 브로커로 사용 가능

### 8.2 사용 예제
```bash
# 메시지 추가
XADD mystream * sensor-id 1234 temperature 19.8
XADD mystream * sensor-id 1234 temperature 20.1

# 메시지 읽기
XREAD COUNT 2 STREAMS mystream 0
```

## 9. 지리공간 데이터 (Geospatial)

### 9.1 개요
- 위치 기반 데이터 저장 및 검색
- 거리 계산 및 반경 검색 지원
- 위치 기반 서비스에 적합

### 9.2 사용 예제
```bash
# 위치 추가
GEOADD locations 127.0276 37.4979 "Seoul"
GEOADD locations 126.9780 37.5665 "Gangnam"

# 거리 계산
GEODIST locations "Seoul" "Gangnam" km
```

## 10. 데이터 타입 선택 가이드

### 10.1 사용 사례별 추천
- 캐싱: Strings
- 세션 관리: Hashes
- 실시간 순위표: Sorted Sets
- 메시지 큐: Lists/Streams
- 사용자 활동 추적: Bitmaps
- 고유 방문자 수: HyperLogLog
- 위치 기반 검색: Geospatial

### 10.2 고려사항
- 메모리 사용량
- 연산 복잡도
- 데이터 접근 패턴
- 확장성 요구사항
- 데이터 일관성 요구사항