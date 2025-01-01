# Redis Vector 데이터베이스 가이드

## 1. Redis Vector 소개

### 1.1 벡터 데이터베이스란?
- 벡터 유사도 검색 지원
- 고차원 벡터 데이터 저장
- 실시간 검색 및 추천
- 머신러닝 모델 통합

### 1.2 주요 활용 사례
- 시맨틱 검색
- 추천 시스템
- 이미지 검색
- 유사 문서 검색
- 이상 탐지

## 2. Redis Vector 설정

### 2.1 설치 및 구성
```bash
# Redis Stack 설치 (Vector 검색 포함)
docker run -d --name redis-stack -p 6379:6379 -p 8001:8001 redis/redis-stack

# 또는 모듈로 설치
redis-server --loadmodule /path/to/redisearch.so
```

### 2.2 인덱스 생성
```bash
# 벡터 인덱스 생성
FT.CREATE idx ON HASH PREFIX 1 item: SCHEMA vec VECTOR FLAT 6 TYPE FLOAT32 DIM 512 DISTANCE_METRIC COSINE

# 복합 인덱스 생성 (메타데이터 포함)
FT.CREATE idx ON HASH PREFIX 1 item: SCHEMA 
    vec VECTOR FLAT 6 TYPE FLOAT32 DIM 512 DISTANCE_METRIC COSINE 
    title TEXT 
    category TAG
```

## 3. 벡터 데이터 관리

### 3.1 벡터 저장
```python
import redis
import numpy as np
from redis.commands.search.field import VectorField
from redis.commands.search.query import Query

# Redis 연결
r = redis.Redis(host='localhost', port=6379)

# 벡터 데이터 생성
vector = np.random.rand(512).astype(np.float32)

# 벡터 저장
r.hset('item:1',
    mapping={
        'vec': vector.tobytes(),
        'title': '제품 1',
        'category': 'electronics'
    }
)
```

### 3.2 벡터 검색
```python
# KNN 검색
query_vector = np.random.rand(512).astype(np.float32)
q = Query(f'*=>[KNN 10 @vec $vec AS score]')\
    .sort_by('score')\
    .return_fields('title', 'category', 'score')\
    .dialect(2)
    
results = r.ft('idx').search(q, query_params={'vec': query_vector.tobytes()})

# 하이브리드 검색 (벡터 + 텍스트)
q = Query('(@category:{electronics} @title:(smartphone))=>[KNN 10 @vec $vec AS score]')
```

## 4. 벡터 인덱스 최적화

### 4.1 인덱스 파라미터
```bash
# FLAT 인덱스 설정
FT.CREATE idx ON HASH PREFIX 1 item: SCHEMA 
    vec VECTOR FLAT 
        6           # Initial cap
        TYPE FLOAT32 
        DIM 512 
        DISTANCE_METRIC COSINE

# HNSW 인덱스 설정
FT.CREATE idx ON HASH PREFIX 1 item: SCHEMA 
    vec VECTOR HNSW 
        12         # M
        50         # EF_CONSTRUCTION
        TYPE FLOAT32 
        DIM 512 
        DISTANCE_METRIC COSINE
```

### 4.2 성능 튜닝
```python
# 배치 처리
pipe = r.pipeline()
for i in range(1000):
    vector = np.random.rand(512).astype(np.float32)
    pipe.hset(f'item:{i}',
        mapping={
            'vec': vector.tobytes(),
            'title': f'제품 {i}'
        }
    )
pipe.execute()

# 인덱스 정보 확인
r.ft('idx').info()
```

## 5. 실시간 벡터 처리

### 5.1 텍스트 임베딩
```python
from transformers import AutoTokenizer, AutoModel
import torch

# 모델 로드
tokenizer = AutoTokenizer.from_pretrained('bert-base-multilingual-cased')
model = AutoModel.from_pretrained('bert-base-multilingual-cased')

def get_embedding(text):
    # 토큰화
    inputs = tokenizer(text, return_tensors='pt', padding=True, truncation=True)
    
    # 임베딩 생성
    with torch.no_grad():
        outputs = model(**inputs)
        embeddings = outputs.last_hidden_state.mean(dim=1)
    
    return embeddings[0].numpy().astype(np.float32)

# 텍스트 임베딩 저장
text = "이것은 샘플 텍스트입니다."
vector = get_embedding(text)
r.hset('doc:1',
    mapping={
        'vec': vector.tobytes(),
        'text': text
    }
)
```

### 5.2 이미지 임베딩
```python
from torchvision import models, transforms
from PIL import Image

# 모델 로드
model = models.resnet50(pretrained=True)
model = torch.nn.Sequential(*(list(model.children())[:-1]))
model.eval()

def get_image_embedding(image_path):
    # 이미지 전처리
    transform = transforms.Compose([
        transforms.Resize(256),
        transforms.CenterCrop(224),
        transforms.ToTensor(),
        transforms.Normalize(mean=[0.485, 0.456, 0.406],
                           std=[0.229, 0.224, 0.225])
    ])
    
    image = Image.open(image_path)
    image = transform(image).unsqueeze(0)
    
    # 임베딩 생성
    with torch.no_grad():
        embedding = model(image)
        embedding = embedding.squeeze()
    
    return embedding.numpy().astype(np.float32)

# 이미지 임베딩 저장
vector = get_image_embedding('image.jpg')
r.hset('img:1',
    mapping={
        'vec': vector.tobytes(),
        'path': 'image.jpg'
    }
)
```

## 6. 고급 검색 기능

### 6.1 범위 검색
```python
# 거리 기반 필터링
q = Query(f'*=>[KNN 10 @vec $vec AS score]')\
    .filter('score <= 0.5')

# 복합 조건 검색
q = Query('(@category:{electronics} @price:[0 1000])=>[KNN 10 @vec $vec AS score]')
```

### 6.2 집계 검색
```python
# 그룹화 및 집계
q = Query('*')\
    .group_by('@category',
        reduce='count(0) as count',
        sort_by='-count')

# 벡터 기반 클러스터링
def vector_clustering(vectors, n_clusters):
    from sklearn.cluster import KMeans
    kmeans = KMeans(n_clusters=n_clusters)
    clusters = kmeans.fit_predict(vectors)
    return clusters
```

## 7. 성능 최적화

### 7.1 캐싱 전략
```python
# 결과 캐싱
from functools import lru_cache

@lru_cache(maxsize=1000)
def cached_search(query_vector):
    q = Query(f'*=>[KNN 10 @vec $vec AS score]')
    return r.ft('idx').search(q, query_params={'vec': query_vector.tobytes()})

# 파이프라인 처리
def batch_vector_search(query_vectors):
    pipe = r.pipeline()
    for vec in query_vectors:
        q = Query(f'*=>[KNN 10 @vec $vec AS score]')
        pipe.ft('idx').search(q, query_params={'vec': vec.tobytes()})
    return pipe.execute()
```

### 7.2 인덱스 최적화
```bash
# 인덱스 최적화 설정
FT.CREATE idx ON HASH PREFIX 1 item: SCHEMA 
    vec VECTOR HNSW 
        12         # M (그래프 연결성)
        50         # EF_CONSTRUCTION (인덱스 품질)
        TYPE FLOAT32 
        DIM 512 
        DISTANCE_METRIC COSINE 
    MAXTEXTFIELDS 
    NOOFFSETS 
    NOHL

# 인덱스 통계 확인
FT.INFO idx
```

## 8. 모니터링 및 관리

### 8.1 성능 모니터링
```python
# 검색 성능 측정
import time

def measure_search_performance(query_vector, n_iterations=100):
    times = []
    q = Query(f'*=>[KNN 10 @vec $vec AS score]')
    
    for _ in range(n_iterations):
        start = time.time()
        r.ft('idx').search(q, query_params={'vec': query_vector.tobytes()})
        times.append(time.time() - start)
    
    return {
        'avg_time': np.mean(times),
        'std_time': np.std(times),
        'min_time': np.min(times),
        'max_time': np.max(times)
    }
```

### 8.2 인덱스 관리
```bash
# 인덱스 삭제
FT.DROPINDEX idx

# 인덱스 재생성
FT.CREATE idx ON HASH PREFIX 1 item: SCHEMA ...

# 메모리 사용량 확인
INFO memory
MEMORY USAGE item:1
```

## 9. 확장 및 통합

### 9.1 분산 처리
```python
# 샤딩 구성
def distribute_vectors(vectors, num_shards):
    shards = {}
    for i, vec in enumerate(vectors):
        shard_id = i % num_shards
        if shard_id not in shards:
            shards[shard_id] = []
        shards[shard_id].append(vec)
    return shards

# 병렬 검색
from concurrent.futures import ThreadPoolExecutor

def parallel_search(query_vector, redis_clients):
    with ThreadPoolExecutor() as executor:
        futures = []
        for client in redis_clients:
            futures.append(
                executor.submit(
                    search_in_shard,
                    client,
                    query_vector
                )
            )
        results = [f.result() for f in futures]
    return merge_results(results)
```

### 9.2 외부 시스템 통합
```python
# Elasticsearch 통합
from elasticsearch import Elasticsearch

def hybrid_search(query_text, query_vector):
    # Redis 벡터 검색
    redis_results = vector_search(query_vector)
    
    # Elasticsearch 텍스트 검색
    es = Elasticsearch()
    es_results = es.search(
        index="products",
        body={
            "query": {
                "match": {
                    "description": query_text
                }
            }
        }
    )
    
    # 결과 통합
    return merge_search_results(redis_results, es_results)
```

## 10. 모범 사례

### 10.1 설계 원칙
- 적절한 벡터 차원 선택
- 효율적인 인덱스 구성
- 배치 처리 활용
- 캐싱 전략 수립
- 정기적인 성능 모니터링

### 10.2 보안 고려사항
- 접근 제어 설정
- 데이터 암호화
- 백업 전략 수립
- 모니터링 및 감사
