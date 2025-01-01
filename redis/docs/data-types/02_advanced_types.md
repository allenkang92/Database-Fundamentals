# Redis 고급 데이터 타입 가이드

## 1. Probabilistic Data Structures

### 1.1 HyperLogLog
- 대규모 카디널리티 추정
- 메모리 효율적 구현
- 사용 예제:
```bash
# 요소 추가
PFADD visitors user1 user2 user3

# 카운트 조회
PFCOUNT visitors

# HyperLogLog 병합
PFMERGE result hll1 hll2
```

### 1.2 Bloom Filter
- 멤버십 테스트
- 거짓 양성 허용
- 사용 예제:
```bash
# Bloom Filter 생성
BF.RESERVE filter 0.01 1000

# 요소 추가
BF.ADD filter "item1"

# 멤버십 확인
BF.EXISTS filter "item1"
```

### 1.3 Cuckoo Filter
- Bloom Filter의 개선 버전
- 요소 삭제 지원
- 사용 예제:
```bash
# Cuckoo Filter 생성
CF.RESERVE mycf 1000

# 요소 추가
CF.ADD mycf "item1"

# 요소 삭제
CF.DEL mycf "item1"
```

## 2. 시계열 데이터 타입

### 2.1 Time Series
- 시계열 데이터 저장 및 분석
- 자동 다운샘플링
- 사용 예제:
```bash
# 시계열 생성
TS.CREATE ts:temperature RETENTION 86400

# 데이터 추가
TS.ADD ts:temperature 1577836800 25.5

# 범위 조회
TS.RANGE ts:temperature 1577836800 1577923200

# 집계
TS.RANGE ts:temperature 1577836800 1577923200 AGGREGATION avg 60
```

### 2.2 RedisTimeSeries 모듈
- 라벨 기반 시계열
- 규칙 기반 정책
- 사용 예제:
```bash
# 라벨이 있는 시계열 생성
TS.CREATE ts:cpu LABELS type cpu_usage node server1

# 규칙 생성
TS.CREATERULE source_key dest_key AGGREGATION avg 60
```

## 3. JSON 데이터 타입

### 3.1 RedisJSON
- 네이티브 JSON 지원
- 경로 기반 연산
- 사용 예제:
```bash
# JSON 저장
JSON.SET user:1 $ '{"name":"John", "age":30, "city":"Seoul"}'

# 경로 기반 조회
JSON.GET user:1 $.name

# 경로 기반 수정
JSON.SET user:1 $.age 31

# 배열 조작
JSON.ARRAPPEND user:1 $.skills "Python"
```

### 3.2 JSON 인덱싱
- 필드 기반 검색
- 복잡한 쿼리 지원
- 사용 예제:
```bash
# JSON 인덱스 생성
FT.CREATE idx:users ON JSON PREFIX 1 user: SCHEMA $.name AS name TEXT $.age AS age NUMERIC

# 인덱스 검색
FT.SEARCH idx:users "@name:John @age:[25 35]"
```

## 4. Graph 데이터 타입

### 4.1 RedisGraph
- 그래프 데이터베이스
- Cypher 쿼리 언어
- 사용 예제:
```bash
# 노드 생성
GRAPH.QUERY social "CREATE (:Person {name:'John', age:30})"

# 관계 생성
GRAPH.QUERY social "MATCH (a:Person), (b:Person) WHERE a.name = 'John' AND b.name = 'Jane' CREATE (a)-[:FOLLOWS]->(b)"

# 경로 검색
GRAPH.QUERY social "MATCH (a:Person)-[:FOLLOWS]->(b:Person) RETURN a.name, b.name"
```

### 4.2 그래프 분석
- 경로 탐색
- 중심성 계산
- 사용 예제:
```bash
# 최단 경로
GRAPH.QUERY social "MATCH p=shortestPath((a:Person)-[*]->(b:Person)) WHERE a.name = 'John' AND b.name = 'Bob' RETURN p"

# 페이지랭크
GRAPH.QUERY social "CALL algo.pageRank('Person')"
```

## 5. Geospatial 데이터 타입

### 5.1 기본 기능
- 위치 저장
- 거리 계산
- 사용 예제:
```bash
# 위치 추가
GEOADD locations 127.0276 37.4979 "Seoul"
GEOADD locations 126.9780 37.5665 "Gangnam"

# 거리 계산
GEODIST locations "Seoul" "Gangnam" km

# 반경 검색
GEORADIUS locations 127.0276 37.4979 10 km
```

### 5.2 고급 기능
- 다각형 검색
- 클러스터링
- 사용 예제:
```bash
# 다각형 영역 검색
GEOSEARCH locations FROMMEMBER "Seoul" BYBOX 10 10 km

# 결과 정렬
GEOSEARCH locations FROMMEMBER "Seoul" BYRADIUS 10 km ASC
```

## 6. Bitmap 데이터 타입

### 6.1 기본 연산
- 비트 설정/해제
- 비트 카운팅
- 사용 예제:
```bash
# 비트 설정
SETBIT online:users 123 1

# 비트 조회
GETBIT online:users 123

# 비트 카운트
BITCOUNT online:users
```

### 6.2 비트 연산
- AND, OR, XOR
- 비트 필드
- 사용 예제:
```bash
# 비트 연산
BITOP AND result online:users:today online:users:yesterday

# 비트 필드
BITFIELD mykey SET u4 0 15
BITFIELD mykey INCRBY u4 0 1
```

## 7. Stream 데이터 타입

### 7.1 메시지 처리
- 메시지 추가/읽기
- 소비자 그룹
- 사용 예제:
```bash
# 메시지 추가
XADD mystream * sensor-id 1234 temperature 19.8

# 메시지 읽기
XREAD COUNT 2 STREAMS mystream 0

# 소비자 그룹 생성
XGROUP CREATE mystream mygroup $
```

### 7.2 스트림 관리
- 트리밍
- 소비자 관리
- 사용 예제:
```bash
# 스트림 트리밍
XTRIM mystream MAXLEN 1000

# 펜딩 메시지 관리
XPENDING mystream mygroup

# 소비자 그룹 삭제
XGROUP DESTROY mystream mygroup
```

## 8. 성능 최적화

### 8.1 메모리 관리
- 메모리 정책
- 압축 옵션
- 사용 예제:
```bash
# 메모리 정책 설정
CONFIG SET maxmemory-policy allkeys-lru

# 메모리 사용량 확인
INFO memory
```

### 8.2 인덱싱 전략
- 보조 인덱스
- 인덱스 최적화
- 사용 예제:
```bash
# 인덱스 생성
FT.CREATE myindex ON HASH PREFIX 1 doc: SCHEMA title TEXT SORTABLE

# 인덱스 최적화
FT.ALTER myindex SCHEMA ADD score NUMERIC SORTABLE
```

## 9. 데이터 타입 선택 가이드

### 9.1 사용 사례별 추천
- 대규모 카운팅: HyperLogLog
- 멤버십 테스트: Bloom Filter
- 시계열 분석: RedisTimeSeries
- 그래프 데이터: RedisGraph
- 위치 기반: Geospatial
- 실시간 분석: Stream

### 9.2 성능 고려사항
- 메모리 사용량
- 연산 복잡도
- 확장성
- 데이터 일관성

## 10. 모범 사례

### 10.1 설계 원칙
- 적절한 데이터 타입 선택
- 메모리 효율성 고려
- 인덱싱 전략 수립
- 확장성 계획

### 10.2 운영 팁
- 모니터링 설정
- 백업 전략
- 성능 튜닝
- 보안 설정