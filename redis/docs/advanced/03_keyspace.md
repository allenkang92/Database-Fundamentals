Keyspace
Managing keys in Redis: Key expiration, scanning, altering and querying the key space

Redis keys are binary safe; this means that you can use any binary sequence as a key, from a string like "foo" to the content of a JPEG file. The empty string is also a valid key.

A few other rules about keys:

Very long keys are not a good idea. For instance a key of 1024 bytes is a bad idea not only memory-wise, but also because the lookup of the key in the dataset may require several costly key-comparisons. Even when the task at hand is to match the existence of a large value, hashing it (for example with SHA1) is a better idea, especially from the perspective of memory and bandwidth.
Very short keys are often not a good idea. There is little point in writing "u1000flw" as a key if you can instead write "user:1000:followers". The latter is more readable and the added space is minor compared to the space used by the key object itself and the value object. While short keys will obviously consume a bit less memory, your job is to find the right balance.
Try to stick with a schema. For instance "object-type:id" is a good idea, as in "user:1000". Dots or dashes are often used for multi-word fields, as in "comment:4321:reply.to" or "comment:4321:reply-to".
The maximum allowed key size is 512 MB.
Altering and querying the key space
There are commands that are not defined on particular types, but are useful in order to interact with the space of keys, and thus, can be used with keys of any type.

For example the EXISTS command returns 1 or 0 to signal if a given key exists or not in the database, while the DEL command deletes a key and associated value, whatever the value is.

> set mykey hello
OK
> exists mykey
(integer) 1
> del mykey
(integer) 1
> exists mykey
(integer) 0
From the examples you can also see how DEL itself returns 1 or 0 depending on whether the key was removed (it existed) or not (there was no such key with that name).

There are many key space related commands, but the above two are the essential ones together with the TYPE command, which returns the kind of value stored at the specified key:

> set mykey x
OK
> type mykey
string
> del mykey
(integer) 1
> type mykey
none
Key expiration
Before moving on, we should look at an important Redis feature that works regardless of the type of value you're storing: key expiration. Key expiration lets you set a timeout for a key, also known as a "time to live", or "TTL". When the time to live elapses, the key is automatically destroyed.

A few important notes about key expiration:

They can be set both using seconds or milliseconds precision.
However the expire time resolution is always 1 millisecond.
Information about expires are replicated and persisted on disk, the time virtually passes when your Redis server remains stopped (this means that Redis saves the date at which a key will expire).
Use the EXPIRE command to set a key's expiration:

> set key some-value
OK
> expire key 5
(integer) 1
> get key (immediately)
"some-value"
> get key (after some time)
(nil)
The key vanished between the two GET calls, since the second call was delayed more than 5 seconds. In the example above we used EXPIRE in order to set the expire (it can also be used in order to set a different expire to a key already having one, like PERSIST can be used in order to remove the expire and make the key persistent forever). However we can also create keys with expires using other Redis commands. For example using SET options:

> set key 100 ex 10
OK
> ttl key
(integer) 9
The example above sets a key with the string value 100, having an expire of ten seconds. Later the TTL command is called in order to check the remaining time to live for the key.

In order to set and check expires in milliseconds, check the PEXPIRE and the PTTL commands, and the full list of SET options.

Navigating the keyspace
Scan
To incrementally iterate over the keys in a Redis database in an efficient manner, you can use the SCAN command.

Since SCAN allows for incremental iteration, returning only a small number of elements per call, it can be used in production without the downside of commands like KEYS or SMEMBERS that may block the server for a long time (even several seconds) when called against big collections of keys or elements.

However while blocking commands like SMEMBERS are able to provide all the elements that are part of a Set in a given moment. The SCAN family of commands only offer limited guarantees about the returned elements since the collection that we incrementally iterate can change during the iteration process.

Keys
Another way to iterate over the keyspace is to use the KEYS command, but this approach should be used with care, since KEYS will block the Redis server until all keys are returned.

Warning: consider KEYS as a command that should only be used in production environments with extreme care.

KEYS may ruin performance when it is executed against large databases. This command is intended for debugging and special operations, such as changing your keyspace layout. Don't use KEYS in your regular application code. If you're looking for a way to find keys in a subset of your keyspace, consider using SCAN or sets.

Supported glob-style patterns:

h?llo matches hello, hallo and hxllo
h*llo matches hllo and heeeello
h[ae]llo matches hello and hallo, but not hillo
h[^e]llo matches hallo, hbllo, ... but not hello
h[a-b]llo matches hallo and hbllo
Use \ to escape special characters if you want to match them verbatim.

Redis pipelining
How to optimize round-trip times by batching Redis commands

Redis pipelining is a technique for improving performance by issuing multiple commands at once without waiting for the response to each individual command. Pipelining is supported by most Redis clients. This document describes the problem that pipelining is designed to solve and how pipelining works in Redis.

Request/Response protocols and round-trip time (RTT)
Redis is a TCP server using the client-server model and what is called a Request/Response protocol.

This means that usually a request is accomplished with the following steps:

The client sends a query to the server, and reads from the socket, usually in a blocking way, for the server response.
The server processes the command and sends the response back to the client.
So for instance a four commands sequence is something like this:

Client: INCR X
Server: 1
Client: INCR X
Server: 2
Client: INCR X
Server: 3
Client: INCR X
Server: 4
Clients and Servers are connected via a network link. Such a link can be very fast (a loopback interface) or very slow (a connection established over the Internet with many hops between the two hosts). Whatever the network latency is, it takes time for the packets to travel from the client to the server, and back from the server to the client to carry the reply.

This time is called RTT (Round Trip Time). It's easy to see how this can affect performance when a client needs to perform many requests in a row (for instance adding many elements to the same list, or populating a database with many keys). For instance if the RTT time is 250 milliseconds (in the case of a very slow link over the Internet), even if the server is able to process 100k requests per second, we'll be able to process at max four requests per second.

If the interface used is a loopback interface, the RTT is much shorter, typically sub-millisecond, but even this will add up to a lot if you need to perform many writes in a row.

Fortunately there is a way to improve this use case.

Redis Pipelining
A Request/Response server can be implemented so that it is able to process new requests even if the client hasn't already read the old responses. This way it is possible to send multiple commands to the server without waiting for the replies at all, and finally read the replies in a single step.

This is called pipelining, and is a technique widely in use for many decades. For instance many POP3 protocol implementations already support this feature, dramatically speeding up the process of downloading new emails from the server.

Redis has supported pipelining since its early days, so whatever version you are running, you can use pipelining with Redis. This is an example using the raw netcat utility:

$ (printf "PING\r\nPING\r\nPING\r\n"; sleep 1) | nc localhost 6379
+PONG
+PONG
+PONG
This time we don't pay the cost of RTT for every call, but just once for the three commands.

To be explicit, with pipelining the order of operations of our very first example will be the following:

Client: INCR X
Client: INCR X
Client: INCR X
Client: INCR X
Server: 1
Server: 2
Server: 3
Server: 4
IMPORTANT NOTE: While the client sends commands using pipelining, the server will be forced to queue the replies, using memory. So if you need to send a lot of commands with pipelining, it is better to send them as batches each containing a reasonable number, for instance 10k commands, read the replies, and then send another 10k commands again, and so forth. The speed will be nearly the same, but the additional memory used will be at most the amount needed to queue the replies for these 10k commands.

It's not just a matter of RTT
Pipelining is not just a way to reduce the latency cost associated with the round trip time, it actually greatly improves the number of operations you can perform per second in a given Redis server. This is because without using pipelining, serving each command is very cheap from the point of view of accessing the data structures and producing the reply, but it is very costly from the point of view of doing the socket I/O. This involves calling the read() and write() syscall, that means going from user land to kernel land. The context switch is a huge speed penalty.

When pipelining is used, many commands are usually read with a single read() system call, and multiple replies are delivered with a single write() system call. Consequently, the number of total queries performed per second initially increases almost linearly with longer pipelines, and eventually reaches 10 times the baseline obtained without pipelining, as shown in this figure.

Pipeline size and IOPs

A real world code example
In the following benchmark we'll use the Redis Ruby client, supporting pipelining, to test the speed improvement due to pipelining:

require 'rubygems'
require 'redis'

def bench(descr)
  start = Time.now
  yield
  puts "#{descr} #{Time.now - start} seconds"
end

def without_pipelining
  r = Redis.new
  10_000.times do
    r.ping
  end
end

def with_pipelining
  r = Redis.new
  r.pipelined do |rp|
    10_000.times do
      rp.ping
    end
  end
end

bench('without pipelining') do
  without_pipelining
end
bench('with pipelining') do
  with_pipelining
end
Running the above simple script yields the following figures on my Mac OS X system, running over the loopback interface, where pipelining will provide the smallest improvement as the RTT is already pretty low:

without pipelining 1.185238 seconds
with pipelining 0.250783 seconds
As you can see, using pipelining, we improved the transfer by a factor of five.

Pipelining vs Scripting
Using Redis scripting, available since Redis 2.6, a number of use cases for pipelining can be addressed more efficiently using scripts that perform a lot of the work needed at the server side. A big advantage of scripting is that it is able to both read and write data with minimal latency, making operations like read, compute, write very fast (pipelining can't help in this scenario since the client needs the reply of the read command before it can call the write command).

Sometimes the application may also want to send EVAL or EVALSHA commands in a pipeline. This is entirely possible and Redis explicitly supports it with the SCRIPT LOAD command (it guarantees that EVALSHA can be called without the risk of failing).

Appendix: Why are busy loops slow even on the loopback interface?
Even with all the background covered in this page, you may still wonder why a Redis benchmark like the following (in pseudo code), is slow even when executed in the loopback interface, when the server and the client are running in the same physical machine:

FOR-ONE-SECOND:
    Redis.SET("foo","bar")
END
After all, if both the Redis process and the benchmark are running in the same box, isn't it just copying messages in memory from one place to another without any actual latency or networking involved?

The reason is that processes in a system are not always running, actually it is the kernel scheduler that lets the process run. So, for instance, when the benchmark is allowed to run, it reads the reply from the Redis server (related to the last command executed), and writes a new command. The command is now in the loopback interface buffer, but in order to be read by the server, the kernel should schedule the server process (currently blocked in a system call) to run, and so forth. So in practical terms the loopback interface still involves network-like latency, because of how the kernel scheduler works.

Basically a busy loop benchmark is the silliest thing that can be done when metering performances on a networked server. The wise thing is just avoiding benchmarking in this way.

Redis keyspace notifications
Monitor changes to Redis keys and values in real time

Keyspace notifications allow clients to subscribe to Pub/Sub channels in order to receive events affecting the Redis data set in some way.

Examples of events that can be received are:

All the commands affecting a given key.
All the keys receiving an LPUSH operation.
All the keys expiring in the database 0.
Note: Redis Pub/Sub is fire and forget; that is, if your Pub/Sub client disconnects, and reconnects later, all the events delivered during the time the client was disconnected are lost.

Type of events
Keyspace notifications are implemented by sending two distinct types of events for every operation affecting the Redis data space. For instance a DEL operation targeting the key named mykey in database 0 will trigger the delivering of two messages, exactly equivalent to the following two PUBLISH commands:

PUBLISH __keyspace@0__:mykey del
PUBLISH __keyevent@0__:del mykey
The first channel listens to all the events targeting the key mykey and the other channel listens only to del operation events on the key mykey

The first kind of event, with keyspace prefix in the channel is called a Key-space notification, while the second, with the keyevent prefix, is called a Key-event notification.

In the previous example a del event was generated for the key mykey resulting in two messages:

The Key-space channel receives as message the name of the event.
The Key-event channel receives as message the name of the key.
It is possible to enable only one kind of notification in order to deliver just the subset of events we are interested in.

Configuration
By default keyspace event notifications are disabled because while not very sensible the feature uses some CPU power. Notifications are enabled using the notify-keyspace-events of redis.conf or via the CONFIG SET.

Setting the parameter to the empty string disables notifications. In order to enable the feature a non-empty string is used, composed of multiple characters, where every character has a special meaning according to the following table:

K     Keyspace events, published with __keyspace@<db>__ prefix.
E     Keyevent events, published with __keyevent@<db>__ prefix.
g     Generic commands (non-type specific) like DEL, EXPIRE, RENAME, ...
$     String commands
l     List commands
s     Set commands
h     Hash commands
z     Sorted set commands
t     Stream commands
d     Module key type events
x     Expired events (events generated every time a key expires)
e     Evicted events (events generated when a key is evicted for maxmemory)
m     Key miss events (events generated when a key that doesn't exist is accessed)
n     New key events (Note: not included in the 'A' class)
A     Alias for "g$lshztxed", so that the "AKE" string means all the events except "m" and "n".
At least K or E should be present in the string, otherwise no event will be delivered regardless of the rest of the string.

For instance to enable just Key-space events for lists, the configuration parameter must be set to Kl, and so forth.

You can use the string KEA to enable most types of events.

Events generated by different commands
Different commands generate different kind of events according to the following list.

APPEND generates an append event.
COPY generates a copy_to event.
DEL generates a del event for every deleted key.
EXPIRE and all its variants (PEXPIRE, EXPIREAT, PEXPIREAT) generate an expire event when called with a positive timeout (or a future timestamp). Note that when these commands are called with a negative timeout value or timestamp in the past, the key is deleted and only a del event is generated instead.
HDEL generates a single hdel event, and an additional del event if the resulting hash is empty and the key is removed.
HEXPIRE and all its variants (HEXPIREAT, HPEXPIRE, HPEXPIREAT) generate hexpire events. Furthermore, hexpired events are generated when fields expire.
HINCRBYFLOAT generates an hincrbyfloat event.
HINCRBY generates an hincrby event.
HPERSIST generates an hpersist event.
HSET, HSETNX and HMSET all generate a single hset event.
INCRBYFLOAT generates an incrbyfloat events.
INCR, DECR, INCRBY, DECRBY commands all generate incrby events.
LINSERT generates an linsert event.
LMOVE and BLMOVE generate an lpop/rpop event (depending on the wherefrom argument) and an lpush/rpush event (depending on the whereto argument). In both cases the order is guaranteed (the lpush/rpush event will always be delivered after the lpop/rpop event). Additionally a del event will be generated if the resulting list is zero length and the key is removed.
LPOP generates an lpop event. Additionally a del event is generated if the key is removed because the last element from the list was popped.
LPUSH and LPUSHX generates a single lpush event, even in the variadic case.
LREM generates an lrem event, and additionally a del event if the resulting list is empty and the key is removed.
LSET generates an lset event.
LTRIM generates an ltrim event, and additionally a del event if the resulting list is empty and the key is removed.
MIGRATE generates a del event if the source key is removed.
MOVE generates two events, a move_from event for the source key, and a move_to event for the destination key.
MSET generates a separate set event for every key.
PERSIST generates a persist event if the expiry time associated with key has been successfully deleted.
RENAME generates two events, a rename_from event for the source key, and a rename_to event for the destination key.
RESTORE generates a restore event for the key.
RPOPLPUSH and BRPOPLPUSH generate an rpop event and an lpush event. In both cases the order is guaranteed (the lpush event will always be delivered after the rpop event). Additionally a del event will be generated if the resulting list is zero length and the key is removed.
RPOP generates an rpop event. Additionally a del event is generated if the key is removed because the last element from the list was popped.
RPUSH and RPUSHX generates a single rpush event, even in the variadic case.
SADD generates a single sadd event, even in the variadic case.
SETRANGE generates a setrange event.
SET and all its variants (SETEX, SETNX,GETSET) generate set events. However SETEX will also generate an expire events.
SINTERSTORE, SUNIONSTORE, SDIFFSTORE generate sinterstore, sunionstore, sdiffstore events respectively. In the special case the resulting set is empty, and the key where the result is stored already exists, a del event is generated since the key is removed.
SMOVE generates an srem event for the source key, and an sadd event for the destination key.
SORT generates a sortstore event when STORE is used to set a new key. If the resulting list is empty, and the STORE option is used, and there was already an existing key with that name, the result is that the key is deleted, so a del event is generated in this condition.
SPOP generates an spop event, and an additional del event if the resulting set is empty and the key is removed.
SREM generates a single srem event, and an additional del event if the resulting set is empty and the key is removed.
XADD generates an xadd event, possibly followed an xtrim event when used with the MAXLEN subcommand.
XDEL generates a single xdel event even when multiple entries are deleted.
XGROUP CREATECONSUMER generates an xgroup-createconsumer event.
XGROUP CREATE generates an xgroup-create event.
XGROUP DELCONSUMER generates an xgroup-delconsumer event.
XGROUP DESTROY generates an xgroup-destroy event.
XGROUP SETID generates an xgroup-setid event.
XSETID generates an xsetid event.
XTRIM generates an xtrim event.
ZADD generates a single zadd event even when multiple elements are added.
ZDIFFSTORE, ZINTERSTORE and ZUNIONSTORE respectively generate zdiffstore, zinterstore and zunionstore events. In the special case the resulting sorted set is empty, and the key where the result is stored already exists, a del event is generated since the key is removed.
ZINCRBY generates a zincr event.
ZREMRANGEBYRANK generates a single zrembyrank event. When the resulting sorted set is empty and the key is generated, an additional del event is generated.
ZREMRANGEBYSCORE generates a single zrembyscore event. When the resulting sorted set is empty and the key is generated, an additional del event is generated.
ZREM generates a single zrem event even when multiple elements are deleted. When the resulting sorted set is empty and the key is generated, an additional del event is generated.
Every time a key with a time to live associated is removed from the data set because it expired, an expired event is generated.
Every time a key is evicted from the data set in order to free memory as a result of the maxmemory policy, an evicted event is generated.
Every time a new key is added to the data set, a new event is generated.
IMPORTANT all the commands generate events only if the target key is really modified. For instance an SREM deleting a non-existing element from a Set will not actually change the value of the key, so no event will be generated.

If in doubt about how events are generated for a given command, the simplest thing to do is to watch yourself:

$ redis-cli config set notify-keyspace-events KEA
$ redis-cli --csv psubscribe '__key*__:*'
Reading messages... (press Ctrl-C to quit)
"psubscribe","__key*__:*",1
At this point use redis-cli in another terminal to send commands to the Redis server and watch the events generated:

"pmessage","__key*__:*","__keyspace@0__:foo","set"
"pmessage","__key*__:*","__keyevent@0__:set","foo"
...
Timing of expired events
Keys with a time to live associated are expired by Redis in two ways:

When the key is accessed by a command and is found to be expired.
Via a background system that looks for expired keys in the background, incrementally, in order to be able to also collect keys that are never accessed.
The expired events are generated when a key is accessed and is found to be expired by one of the above systems, as a result there are no guarantees that the Redis server will be able to generate the expired event at the time the key time to live reaches the value of zero.

If no command targets the key constantly, and there are many keys with a TTL associated, there can be a significant delay between the time the key time to live drops to zero, and the time the expired event is generated.

Expired (expired) events are generated when the Redis server deletes the key and not when the time to live theoretically reaches the value of zero.

Events in a cluster
Every node of a Redis cluster generates events about its own subset of the keyspace as described above. However, unlike regular Pub/Sub communication in a cluster, events' notifications are not broadcasted to all nodes. Put differently, keyspace events are node-specific. This means that to receive all keyspace events of a cluster, clients need to subscribe to each of the nodes.

@history

>= 6.0: Key miss events were added.
>= 7.0: Event type new added

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