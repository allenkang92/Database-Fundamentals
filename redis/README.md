# Redis 학습 프로젝트

이 프로젝트는 Redis의 기본 개념부터 고급 기능까지 다루는 학습 자료를 포함하고 있습니다.

## 디렉토리 구조

- `/docs`: Redis 관련 학습 문서
  - 기본 개념
  - 설치 가이드
  - 명령어
  - 캐싱
  - RAG (Retrieval-Augmented Generation)
  - 벡터 검색
  - 데이터 타입
  - AI 활용
  - 고급 기능
  - 운영
  - Keyspace 관리

- `/docker`: Docker 관련 파일
  - Dockerfile
  - docker-compose 설정 (향후 추가 예정)

## 시작하기

1. Docker를 사용한 Redis 실행:
```bash
cd docker
docker build -t my-redis .
docker run -d --name redis-server -p 6379:6379 my-redis
```

2. Redis CLI 접속:
```bash
docker exec -it redis-server redis-cli
```
