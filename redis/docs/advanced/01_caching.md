캐싱

데이터 캐싱 전략

Cache Aside

# Redis 캐싱 전략

## 1. Redis 캐싱이란?

Redis 캐싱은 자주 접근하는 데이터를 메모리에 저장하여 빠른 응답 시간을 제공하는 기술입니다.
데이터베이스의 부하를 줄이고 애플리케이션의 성능을 향상시키는데 매우 효과적입니다.

## 2. 캐싱 패턴

### 2.1 Cache-Aside (Lazy Loading)
- 가장 일반적인 캐싱 패턴
- 캐시 미스가 발생할 때만 데이터베이스에서 데이터를 로드
```python
def get_user(user_id):
    # 1. 캐시에서 조회
    user = redis.get(f"user:{user_id}")
    if user:
        return user
    
    # 2. 캐시 미스: DB에서 조회
    user = db.query(f"SELECT * FROM users WHERE id = {user_id}")
    
    # 3. 캐시에 저장
    redis.set(f"user:{user_id}", user, ex=3600)  # 1시간 유효
    return user
```

### 2.2 Write-Through
- 데이터 쓰기 시 캐시와 DB를 동시에 업데이트
- 데이터 일관성이 중요할 때 사용
```python
def update_user(user_id, data):
    # 1. DB 업데이트
    db.query(f"UPDATE users SET ... WHERE id = {user_id}")
    
    # 2. 캐시 업데이트
    redis.set(f"user:{user_id}", data, ex=3600)
```

### 2.3 Write-Behind (Write-Back)
- 데이터를 캐시에만 먼저 쓰고, 나중에 DB에 일괄 업데이트
- 쓰기 성능이 중요할 때 사용
```python
def update_user(user_id, data):
    # 1. 캐시 업데이트
    redis.set(f"user:{user_id}", data)
    
    # 2. 업데이트 큐에 추가
    redis.rpush("db_updates", {"user_id": user_id, "data": data})
```

## 3. 캐시 무효화 전략

### 3.1 TTL (Time To Live)
- 캐시 항목에 만료 시간 설정
```python
# 1시간 후 만료
redis.set("key", "value", ex=3600)

# 특정 시간에 만료
redis.expireat("key", int(time.time() + 3600))
```

### 3.2 LRU (Least Recently Used)
- Redis maxmemory-policy 설정으로 LRU 구현
```bash
# redis.conf
maxmemory 2gb
maxmemory-policy allkeys-lru
```

### 3.3 이벤트 기반 무효화
- 데이터 변경 시 관련 캐시 삭제
```python
def update_user_profile(user_id, new_data):
    # 1. DB 업데이트
    db.update_user(user_id, new_data)
    
    # 2. 관련 캐시 모두 삭제
    keys_to_delete = redis.keys(f"user:{user_id}:*")
    redis.delete(*keys_to_delete)
```

## 4. 성능 최적화 전략

### 4.1 배치 처리
- 여러 작업을 한 번에 처리하여 네트워크 왕복 최소화
```python
# 파이프라인 사용
with redis.pipeline() as pipe:
    pipe.set("key1", "value1")
    pipe.set("key2", "value2")
    pipe.execute()
```

### 4.2 압축
- 큰 데이터는 압축하여 저장
```python
import zlib

def set_compressed(key, value):
    compressed = zlib.compress(value.encode())
    redis.set(key, compressed)

def get_compressed(key):
    compressed = redis.get(key)
    if compressed:
        return zlib.decompress(compressed).decode()
```

### 4.3 부분 캐싱
- 전체 객체가 아닌 자주 사용되는 필드만 캐싱
```python
def get_user_profile(user_id):
    # 기본 정보만 캐싱
    basic_info = redis.hgetall(f"user:{user_id}:basic")
    if not basic_info:
        user = db.get_user(user_id)
        redis.hmset(f"user:{user_id}:basic", {
            "name": user.name,
            "email": user.email
        })
        return user
    return basic_info
```

## 5. 모니터링 및 디버깅

### 5.1 캐시 히트율 모니터링
```python
def get_with_stats(key):
    result = redis.get(key)
    if result:
        redis.incr("stats:hits")
    else:
        redis.incr("stats:misses")
    return result

def get_hit_rate():
    hits = int(redis.get("stats:hits") or 0)
    misses = int(redis.get("stats:misses") or 0)
    total = hits + misses
    return (hits / total) if total > 0 else 0
```

### 5.2 메모리 사용량 모니터링
```python
def monitor_memory():
    info = redis.info()
    used_memory = info["used_memory_human"]
    peak_memory = info["used_memory_peak_human"]
    return {
        "current": used_memory,
        "peak": peak_memory
    }