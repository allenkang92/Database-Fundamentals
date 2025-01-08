# Redis RAG (Retrieval Augmented Generation) 활용 가이드

## 1. RAG 개요

### 1.1 RAG란?
RAG(Retrieval Augmented Generation)는 대규모 언어 모델(LLM)의 성능을 향상시키기 위해 외부 데이터를 활용하는 기술입니다.
Redis의 벡터 데이터베이스 기능을 사용하여 효율적인 RAG 시스템을 구축할 수 있습니다.

### 1.2 RAG의 장점
- 최신 정보 활용 가능
- 도메인 특화 지식 반영
- 환각(Hallucination) 현상 감소
- 답변의 신뢰성 향상

## 2. Redis를 활용한 RAG 구현

### 2.1 기본 아키텍처
```python
from redis import Redis
from redis.commands.search.field import VectorField
from redis.commands.search.query import Query
import numpy as np

# Redis 연결 설정
redis_client = Redis(host='localhost', port=6379, db=0)

# 벡터 검색을 위한 인덱스 생성
def create_index():
    schema = (
        VectorField("embedding", "FLOAT32", "FLAT", {
            "TYPE": "FLOAT32",
            "DIM": 384,
            "DISTANCE_METRIC": "COSINE"
        }),
    )
    redis_client.ft("doc_idx").create_index(schema)
```

### 2.2 문서 임베딩 저장
```python
def store_document(doc_id, content, embedding):
    # 문서 데이터 저장
    doc_key = f"doc:{doc_id}"
    redis_client.hset(doc_key,
        mapping={
            "content": content,
            "embedding": embedding.tobytes()
        }
    )
```

### 2.3 유사 문서 검색
```python
def search_similar_docs(query_embedding, top_k=5):
    # 벡터 검색 쿼리 구성
    q = Query(
        f"*=>[KNN {top_k} @embedding $vec_param AS score]"
    ).sort_by("score")
    
    # 검색 실행
    query_params = {
        "vec_param": query_embedding.tobytes()
    }
    results = redis_client.ft("doc_idx").search(q, query_params)
    
    return [
        {
            "id": doc.id,
            "content": doc.content,
            "score": doc.score
        }
        for doc in results.docs
    ]
```

## 3. RAG 파이프라인 구현

### 3.1 전체 파이프라인
```python
from transformers import AutoTokenizer, AutoModel

class RAGPipeline:
    def __init__(self):
        self.tokenizer = AutoTokenizer.from_pretrained("sentence-transformers/all-MiniLM-L6-v2")
        self.model = AutoModel.from_pretrained("sentence-transformers/all-MiniLM-L6-v2")
        self.redis_client = Redis(host='localhost', port=6379, db=0)
    
    def generate_embedding(self, text):
        inputs = self.tokenizer(text, return_tensors="pt", padding=True, truncation=True)
        outputs = self.model(**inputs)
        return outputs.last_hidden_state.mean(dim=1).detach().numpy()
    
    def process_query(self, query, top_k=3):
        # 1. 쿼리 임베딩 생성
        query_embedding = self.generate_embedding(query)
        
        # 2. 유사 문서 검색
        relevant_docs = search_similar_docs(query_embedding, top_k)
        
        # 3. 컨텍스트 구성
        context = "\n".join([doc["content"] for doc in relevant_docs])
        
        # 4. LLM 프롬프트 구성
        prompt = f"""
        Context: {context}
        
        Question: {query}
        
        Answer based on the context above:
        """
        
        return prompt
```

### 3.2 문서 색인 예제
```python
def index_documents(documents):
    pipeline = RAGPipeline()
    
    for doc_id, content in documents.items():
        # 임베딩 생성
        embedding = pipeline.generate_embedding(content)
        
        # Redis에 저장
        store_document(doc_id, content, embedding)
```

### 3.3 질의응답 예제
```python
def answer_question(query):
    pipeline = RAGPipeline()
    
    # RAG 프로세스 실행
    prompt = pipeline.process_query(query)
    
    # LLM API 호출 (예: OpenAI)
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": prompt}
        ]
    )
    
    return response.choices[0].message.content
```

## 4. 성능 최적화

### 4.1 벡터 검색 최적화
```python
# 하이브리드 검색 구현
def hybrid_search(query, embedding, top_k=5):
    # 키워드 검색과 벡터 검색 결합
    q = Query(
        f"(@content:{query})=>[KNN {top_k} @embedding $vec_param AS vector_score]"
    ).sort_by("vector_score")
    
    return redis_client.ft("doc_idx").search(q, {
        "vec_param": embedding.tobytes()
    })
```

### 4.2 캐싱 적용
```python
def cached_search(query, ttl=3600):
    # 캐시 키 생성
    cache_key = f"cache:query:{hash(query)}"
    
    # 캐시 확인
    cached_result = redis_client.get(cache_key)
    if cached_result:
        return json.loads(cached_result)
    
    # 검색 실행
    result = answer_question(query)
    
    # 결과 캐싱
    redis_client.setex(cache_key, ttl, json.dumps(result))
    
    return result
```

## 5. 모니터링 및 평가

### 5.1 성능 메트릭 수집
```python
def log_rag_metrics(query, response_time, num_docs_retrieved):
    redis_client.hincrby("rag:metrics", "total_queries", 1)
    redis_client.hincrbyfloat("rag:metrics", "avg_response_time",
        (response_time - float(redis_client.hget("rag:metrics", "avg_response_time") or 0)) /
        float(redis_client.hget("rag:metrics", "total_queries"))
    )
    redis_client.hincrby("rag:metrics", "total_docs_retrieved", num_docs_retrieved)
```

### 5.2 품질 평가
```python
def evaluate_response(query, response, feedback):
    redis_client.hincrby("rag:feedback", "total_feedback", 1)
    if feedback > 0:
        redis_client.hincrby("rag:feedback", "positive_feedback", 1)
    
    # 피드백 저장
    redis_client.hset(f"rag:feedback:{query}", mapping={
        "response": response,
        "score": feedback,
        "timestamp": time.time()
    })