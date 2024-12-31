redis 기본 명령어


key 네이밍 컨벤션

# Redis 기본 명령어

## 1. 문자열(String) 명령어

### 1.1 기본 CRUD
```bash
# 값 설정
SET key value
SET name "John Doe"

# 값 조회
GET key
GET name

# 값 삭제
DEL key
DEL name

# 여러 키 한번에 설정
MSET key1 value1 key2 value2
MSET firstname "John" lastname "Doe"

# 여러 키 한번에 조회
MGET key1 key2
MGET firstname lastname
```

### 1.2 만료시간 설정
```bash
# 키 만료시간 설정 (초)
EXPIRE key seconds
EXPIRE session_token 3600

# 키 만료시간 설정 (밀리초)
PEXPIRE key milliseconds
PEXPIRE session_token 3600000

# 만료시간과 함께 값 설정
SETEX key seconds value
SETEX session_token 3600 "abc123"
```

## 2. 리스트(List) 명령어

### 2.1 기본 조작
```bash
# 왼쪽에 요소 추가
LPUSH key value [value ...]
LPUSH mylist "first" "second"

# 오른쪽에 요소 추가
RPUSH key value [value ...]
RPUSH mylist "last" "very last"

# 왼쪽에서 요소 제거 및 반환
LPOP key
LPOP mylist

# 오른쪽에서 요소 제거 및 반환
RPOP key
RPOP mylist

# 리스트 범위 조회
LRANGE key start stop
LRANGE mylist 0 -1  # 전체 리스트 조회
```

## 3. 집합(Set) 명령어

### 3.1 기본 조작
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

## 4. 정렬된 집합(Sorted Set) 명령어

### 4.1 기본 조작
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

## 5. 해시(Hash) 명령어

### 5.1 기본 조작
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

## 6. 키 관리 명령어

### 6.1 키 조작
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

# 모든 키 조회 (주의: 프로덕션에서 사용 금지)
KEYS pattern
KEYS user:*
```

### 6.2 데이터베이스 관리
```bash
# 데이터베이스 선택
SELECT index
SELECT 1

# 현재 데이터베이스의 모든 키 삭제
FLUSHDB

# 모든 데이터베이스의 모든 키 삭제
FLUSHALL
```

## 7. 서버 관리 명령어

### 7.1 정보 조회
```bash
# 서버 정보 조회
INFO

# 메모리 사용량 조회
INFO memory

# 클라이언트 목록 조회
CLIENT LIST
```

### 7.2 백업
```bash
# 백그라운드에서 RDB 스냅샷 생성
BGSAVE

# AOF 재작성
BGREWRITEAOF