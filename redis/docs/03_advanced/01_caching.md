# Redis 캐싱 전략

## 캐시(Cache)와 캐싱(Caching)

**캐시(Cache)**: 원본 데이터 저장소보다 빠르게 접근할 수 있는 임시 데이터 저장소입니다.

**캐싱(Caching)**: 캐시에 접근하여 데이터를 빠르게 가져오는 방식입니다.

## Redis를 캐시로 사용할 때의 주요 전략

### 1. Cache-Aside (Look Aside, Lazy Loading)

가장 일반적인 데이터 조회 전략입니다. 데이터베이스에서 직접 조회하기 전에 Redis에 데이터가 있는지 확인합니다.

**캐시 히트(Cache Hit):**
애플리케이션 -> Redis (데이터 요청)
Redis -> 애플리케이션 (데이터 응답)

**캐시 미스(Cache Miss):**
애플리케이션 -> Redis (데이터 요청)
애플리케이션 -> 데이터베이스 (데이터 요청)
데이터베이스 -> 애플리케이션 (데이터 응답)
애플리케이션 -> Redis (데이터 저장)


**예시:**

```python
import redis

r = redis.Redis(host='localhost', port=6379, db=0)

def get_user(user_id):
    key = f"user:{user_id}"
    user_data = r.get(key)
    if user_data:
        print("캐시 히트!")
        return user_data.decode('utf-8')
    else:
        print("캐시 미스!")
        # 데이터베이스에서 사용자 정보를 가져오는 로직 (가정)
        user_data_from_db = f"User data for ID: {user_id}"
        r.set(key, user_data_from_db, ex=3600)  # 1시간 만료
        return user_data_from_db


2. Write-Through
데이터를 쓸 때 캐시와 데이터베이스를 동시에 업데이트하는 전략입니다. 데이터 일관성이 중요한 경우에 사용됩니다.

import redis

r = redis.Redis(host='localhost', port=6379, db=0)

def update_user(user_id, data):
    # 데이터베이스 업데이트 로직 (가정)
    print(f"데이터베이스 업데이트: User ID {user_id} with {data}")
    r.set(f"user:{user_id}", data, ex=3600)

3. Write-Behind (Write-Back)
데이터를 먼저 캐시에만 쓰고, 나중에 데이터베이스에 일괄적으로 업데이트하는 전략입니다. 쓰기 성능이 중요할 때 유용합니다.

import redis

r = redis.Redis(host='localhost', port=6379, db=0)

def update_user(user_id, data):
    r.set(f"user:{user_id}:pending_update", data)
    # 별도의 프로세스 또는 스케줄러를 통해 데이터베이스 업데이트
    print(f"캐시에 저장 (DB 업데이트 대기): User ID {user_id} with {data}")

4. Write-Around
데이터를 저장할 때 캐시를 거치지 않고 데이터베이스에 직접 저장하는 방식입니다. 데이터를 조회할 때 캐시에 없으면 데이터베이스에서 조회 후 캐시에 저장합니다.

Cache-Aside와 Write-Around의 한계 및 해결 방법
1. 캐시된 데이터와 DB 데이터 불일치:

문제: Write-Around 전략에서 데이터를 수정할 때 데이터베이스만 업데이트하므로, 캐시된 데이터와 데이터베이스 데이터가 일치하지 않을 수 있습니다.

해결:

TTL(Time To Live) 활용: Redis의 TTL 기능을 사용하여 캐시된 데이터의 만료 시간을 설정합니다. 일정 시간이 지나면 캐시에서 데이터가 삭제되고, 다음 조회 시 데이터베이스에서 최신 데이터를 가져와 캐시를 갱신합니다.

데이터 갱신 시 캐시 무효화: 데이터베이스를 업데이트할 때 관련 캐시를 삭제하여 다음 조회 시 데이터베이스에서 최신 데이터를 가져오도록 합니다.

2. 캐시 공간 제한:

문제: 캐시는 메모리에 저장되므로 데이터베이스에 비해 저장 공간이 제한적입니다.

해결:

TTL 활용: 자주 조회되지 않는 데이터는 만료 시간을 설정하여 캐시에서 자동으로 제거되도록 합니다.

LRU (Least Recently Used) 등의 eviction 정책 활용: Redis의 maxmemory-policy 설정을 통해 메모리가 부족할 때 가장 오래 사용되지 않은 데이터를 자동으로 제거합니다.

캐시 무효화 전략
1. TTL (Time To Live)
캐시 항목에 만료 시간을 설정합니다.

r.set("mykey", "myvalue", ex=3600)  # 1시간 후 만료
r.expireat("mykey", int(time.time() + 7200)) # 특정 시간(2시간 후)에 만료

2. LRU (Least Recently Used)
Redis의 maxmemory-policy 설정을 통해 LRU 기반의 캐시 제거 정책을 활성화합니다.

# redis.conf 설정 예시
maxmemory 2gb
maxmemory-policy allkeys-lru

3. 이벤트 기반 무효화
데이터 변경 이벤트 발생 시 관련 캐시를 삭제합니다.

def update_user_profile(user_id, new_data):
    # 데이터베이스 업데이트 로직
    print(f"DB 업데이트: User {user_id}")
    # 관련 캐시 삭제
    keys_to_delete = r.keys(f"user:{user_id}:*")
    for key in keys_to_delete:
        r.delete(key)

캐싱 성능 최적화 전략
1. 배치 처리 (파이프라인)
여러 작업을 한 번의 요청으로 처리하여 네트워크 왕복 횟수를 줄입니다.

pipe = r.pipeline()
pipe.set('key1', 'value1')
pipe.get('key1')
results = pipe.execute()
print(results)

2. 데이터 압축
큰 데이터는 압축하여 저장 공간을 절약하고 네트워크 전송 시간을 단축합니다.

import zlib

def set_compressed(key, value):
    compressed_value = zlib.compress(value.encode())
    r.set(key, compressed_value)

def get_compressed(key):
    compressed_value = r.get(key)
    if compressed_value:
        return zlib.decompress(compressed_value).decode()
    return None

3. 부분 캐싱
전체 객체 대신 자주 사용되는 필드만 캐싱합니다.

def get_user_basic_info(user_id):
    key = f"user:{user_id}:basic"
    basic_info = r.hgetall(key)
    if not basic_info:
        # 데이터베이스에서 기본 정보를 가져오는 로직 (가정)
        user_from_db = {"name": "John", "email": "john@example.com"}
        r.hmset(key, user_from_db)
        return user_from_db
    return {k.decode('utf-8'): v.decode('utf-8') for k, v in basic_info.items()}

캐시 모니터링 및 디버깅
1. 캐시 히트율 모니터링
def get_with_stats(key):
    value = r.get(key)
    if value:
        r.incr("stats:hits")
    else:
        r.incr("stats:misses")
    return value

def get_hit_rate():
    hits = int(r.get("stats:hits") or 0)
    misses = int(r.get("stats:misses") or 0)
    total = hits + misses
    return (hits / total) * 100 if total > 0 else 0

2. 메모리 사용량 모니터링
info = r.info()
used_memory = info.get('used_memory_human')
print(f"Redis 메모리 사용량: {used_memory}")

데이터 조회 성능 개선 방법 (Redis 외)
Redis를 활용한 캐싱 외에도 데이터 조회 성능을 개선하는 다양한 방법이 있습니다.

SQL 튜닝: 비효율적인 SQL 쿼리를 개선하여 데이터베이스 자체의 성능을 향상시킵니다. 시스템 변경 없이 성능을 개선할 수 있는 효과적인 방법입니다.

애플리케이션 레벨 캐싱: 애플리케이션 내에서 자주 사용되는 데이터를 캐싱합니다.

데이터베이스 Master/Slave 구조: 읽기 작업을 Slave 서버로 분산하여 Master 서버의 부하를 줄입니다.

데이터베이스 샤딩: 데이터를 여러 데이터베이스에 분산하여 저장하고 처리합니다.

데이터베이스 스케일업: CPU, 메모리, SSD 등 데이터베이스 서버의 하드웨어를 업그레이드합니다.

일반적으로 SQL 튜닝을 먼저 고려하는 것이 좋습니다. 이는 추가적인 시스템 구축 없이 기존 시스템 내에서 성능을 개선할 수 있는 가장 비용 효율적인 방법일 수 있습니다. SQL 튜닝으로 기본적인 성능 향상을 이룬 후, 필요에 따라 캐싱 서버나 데이터베이스 구조 변경 등의 시스템적인 개선을 고려하는 것이 효율적입니다.