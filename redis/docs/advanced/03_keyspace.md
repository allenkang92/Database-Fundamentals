# Redis 키스페이스 가이드

## 1. 키스페이스 기본 개념

### 1.1 키 구조
- 네이밍 규칙
- 키 패턴
- 예제:
```bash
# 키 네이밍 패턴
user:1000:profile
order:2024:01:31
session:abc123:data

# 키 패턴 검색
KEYS user:*
SCAN 0 MATCH order:2024:*
```

### 1.2 키 관리
- 키 생성/삭제
- 만료 시간 설정
- 예제:
```bash
# 키 존재 여부 확인
EXISTS mykey

# 키 삭제
DEL mykey

# 만료 시간 설정
EXPIRE mykey 3600
EXPIREAT mykey 1735689600
```

## 2. 키스페이스 이벤트

### 2.1 키스페이스 알림
- 이벤트 구독
- 이벤트 처리
- 예제:
```bash
# 키스페이스 이벤트 활성화
CONFIG SET notify-keyspace-events KEA

# 이벤트 구독
SUBSCRIBE __keyspace@0__:mykey
PSUBSCRIBE __keyevent@0__:expired
```

### 2.2 이벤트 타입
- 키 변경
- 만료
- 예제:
```python
import redis

r = redis.Redis()
pubsub = r.pubsub()

# 만료 이벤트 구독
pubsub.psubscribe('__keyevent@0__:expired')

# 이벤트 처리
for message in pubsub.listen():
    if message['type'] == 'pmessage':
        print(f"키 만료: {message['data']}")
```

## 3. 키스페이스 스캔

### 3.1 스캔 명령어
- SCAN
- SSCAN
- HSCAN
- ZSCAN
- 예제:
```bash
# 기본 스캔
SCAN 0 MATCH user:* COUNT 100

# 집합 스캔
SSCAN myset 0 MATCH *pattern* COUNT 50

# 해시 스캔
HSCAN myhash 0 MATCH field* COUNT 20
```

### 3.2 스캔 전략
- 커서 기반 반복
- 패턴 매칭
- 예제:
```python
def scan_keys(pattern):
    cursor = 0
    while True:
        cursor, keys = r.scan(cursor, match=pattern, count=100)
        for key in keys:
            yield key
        if cursor == 0:
            break
```

## 4. 키 만료 관리

### 4.1 만료 정책
- 만료 시간 설정
- 만료 이벤트 처리
- 예제:
```bash
# 절대 시간 만료
EXPIREAT mykey 1735689600

# 상대 시간 만료
EXPIRE mykey 3600

# 밀리초 단위 만료
PEXPIRE mykey 3600000
```

### 4.2 만료 모니터링
- 만료 시간 조회
- 만료 이벤트 구독
- 예제:
```bash
# 만료 시간 조회
TTL mykey
PTTL mykey

# 만료 제거
PERSIST mykey
```

## 5. 키스페이스 분할

### 5.1 데이터베이스 선택
- 다중 데이터베이스
- 데이터베이스 전환
- 예제:
```bash
# 데이터베이스 선택
SELECT 1

# 데이터베이스 정보
INFO keyspace

# 데이터베이스 플러시
FLUSHDB
```

### 5.2 네임스페이스 관리
- 키 프리픽스
- 네임스페이스 분리
- 예제:
```bash
# 프리픽스 기반 키 관리
SET app1:user:1 "data"
SET app2:user:1 "data"

# 네임스페이스 검색
KEYS app1:*
```

## 6. 메모리 관리

### 6.1 메모리 정책
- 최대 메모리 설정
- 제거 정책
- 예제:
```bash
# 메모리 제한 설정
CONFIG SET maxmemory 2gb
CONFIG SET maxmemory-policy allkeys-lru

# 메모리 사용량 확인
INFO memory
MEMORY USAGE mykey
```

### 6.2 키 제거 전략
- LRU (Least Recently Used)
- LFU (Least Frequently Used)
- 예제:
```bash
# LRU 정책 설정
CONFIG SET maxmemory-policy allkeys-lru
CONFIG SET maxmemory-samples 5

# 수동 키 제거
DEL mykey
UNLINK mykey
```

## 7. 트랜잭션 관리

### 7.1 트랜잭션 기본
- MULTI/EXEC
- WATCH
- 예제:
```bash
# 기본 트랜잭션
MULTI
SET key1 "value1"
SET key2 "value2"
EXEC

# WATCH를 사용한 트랜잭션
WATCH mykey
MULTI
SET mykey newvalue
EXEC
```

### 7.2 트랜잭션 패턴
- 낙관적 잠금
- 롤백 처리
- 예제:
```python
def atomic_increment():
    pipe = r.pipeline(transaction=True)
    while True:
        try:
            pipe.watch('counter')
            current_value = pipe.get('counter')
            pipe.multi()
            pipe.set('counter', int(current_value) + 1)
            pipe.execute()
            break
        except redis.WatchError:
            continue
```

## 8. 키스페이스 모니터링

### 8.1 모니터링 도구
- INFO 명령어
- MONITOR
- 예제:
```bash
# 키스페이스 정보
INFO keyspace

# 실시간 모니터링
MONITOR

# 슬로우 로그
SLOWLOG GET 10
```

### 8.2 통계 수집
- 키 개수
- 메모리 사용량
- 예제:
```bash
# 데이터베이스 크기
DBSIZE

# 메모리 분석
MEMORY STATS
MEMORY DOCTOR
```

## 9. 백업 및 복구

### 9.1 백업 전략
- RDB 스냅샷
- AOF 로그
- 예제:
```bash
# RDB 스냅샷 생성
SAVE
BGSAVE

# AOF 재작성
BGREWRITEAOF
```

### 9.2 복구 절차
- 스냅샷 복구
- AOF 복구
- 예제:
```bash
# RDB 파일 복구
CONFIG SET dir /var/lib/redis
CONFIG SET dbfilename dump.rdb

# AOF 파일 복구
CONFIG SET appendonly yes
CONFIG SET appendfilename "appendonly.aof"
```

## 10. 모범 사례

### 10.1 키 설계 원칙
- 명확한 네이밍
- 적절한 만료 시간
- 메모리 효율성

### 10.2 운영 팁
- 정기적인 모니터링
- 백업 자동화
- 성능 최적화 전략
- 보안 설정 관리