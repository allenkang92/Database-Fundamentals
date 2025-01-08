# Redis 기본 명령어

## 1. Redis CLI 접속
```bash
docker exec -it redis-server redis-cli
```

## 2. 기본 키 관리
```bash
# 키 존재 여부 확인
EXISTS key
EXISTS mykey

# 키 타입 확인
TYPE key
TYPE mykey

# 키 이름 변경
RENAME key newkey
RENAME oldkey newkey

# 키 삭제
DEL key
DEL mykey

# 모든 키 조회 (주의: 프로덕션에서 사용 금지)
KEYS pattern
KEYS user:*
```

## 3. 만료시간 (TTL) 설정
```bash
# 만료시간 설정 (초 단위)
SET key value EX seconds
SET mykey "hello" EX 30    # 30초 후 만료

# 만료시간과 함께 설정
SETEX key seconds value
SETEX mykey 30 "hello"     # 30초 후 만료

# 남은 만료시간 확인
TTL key
TTL mykey                  # 남은 시간(초) 반환
                          # -2: 키가 없음
                          # -1: 만료시간 설정 안됨
```

## 4. 데이터 타입별 명령어

### 4.1 문자열(String)
```bash
# 값 설정
SET key value
SET name "John Doe"    # 공백이 있는 경우 따옴표 사용

# 값 조회
GET key
GET name

# 다중 설정
MSET key1 value1 key2 value2
MSET firstname "John" lastname "Doe"

# 다중 조회
MGET key1 key2
MGET firstname lastname
```

### 4.2 리스트(List)
```bash
# 왼쪽에 요소 추가
LPUSH key value [value ...]
LPUSH mylist "first" "second"

# 오른쪽에 요소 추가
RPUSH key value [value ...]
RPUSH mylist "last" "very last"

# 왼쪽/오른쪽에서 요소 제거 및 반환
LPOP key
RPOP key

# 리스트 범위 조회
LRANGE key start stop
LRANGE mylist 0 -1    # 전체 리스트 조회
```

### 4.3 집합(Set)
```bash
# 요소 추가
SADD key member [member ...]
SADD myset "apple" "banana" "orange"

# 요소 제거
SREM key member [member ...]
SREM myset "apple"

# 모든 요소 조회
SMEMBERS key
SMEMBERS myset

# 요소 존재 여부 확인
SISMEMBER key member
SISMEMBER myset "apple"
```

### 4.4 정렬된 집합(Sorted Set)
```bash
# 점수와 함께 요소 추가
ZADD key score member [score member ...]
ZADD leaderboard 100 "player1" 200 "player2"

# 범위로 요소 조회 (오름차순)
ZRANGE key start stop [WITHSCORES]
ZRANGE leaderboard 0 -1 WITHSCORES

# 범위로 요소 조회 (내림차순)
ZREVRANGE key start stop [WITHSCORES]
ZREVRANGE leaderboard 0 -1 WITHSCORES
```

### 4.5 해시(Hash)
```bash
# 필드 설정
HSET key field value
HSET user:1 name "John" age "30"

# 필드 조회
HGET key field
HGET user:1 name

# 모든 필드-값 쌍 조회
HGETALL key
HGETALL user:1

# 필드 삭제
HDEL key field [field ...]
HDEL user:1 age
```

## 5. 데이터베이스 관리
```bash
# 데이터베이스 선택
SELECT index
SELECT 1

# 현재 데이터베이스의 모든 키 삭제
FLUSHDB

# 모든 데이터베이스의 모든 키 삭제
FLUSHALL
```

## 6. 서버 관리
```bash
# 서버 정보 조회
INFO

# 메모리 사용량 조회
INFO memory

# 클라이언트 목록 조회
CLIENT LIST

# 백업
BGSAVE                # RDB 스냅샷 생성
BGREWRITEAOF         # AOF 재작성