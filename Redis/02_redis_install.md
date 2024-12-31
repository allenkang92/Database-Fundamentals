# Redis Docker 설치 가이드

## 1. Docker를 이용한 Redis 설치

### 1.1 Redis 이미지 다운로드
```bash
# 최신 버전 Redis 이미지 다운로드
docker pull redis

# 특정 버전 Redis 이미지 다운로드 (예: 6.2)
docker pull redis:6.2
```

### 1.2 Redis 컨테이너 실행
```bash
# 기본 실행 (휘발성)
docker run --name redis-container -d -p 6379:6379 redis

# 데이터 영구 저장을 위한 볼륨 마운트
docker run --name redis-container -d \
  -p 6379:6379 \
  -v redis-data:/data \
  redis redis-server --appendonly yes
```

### 1.3 Redis CLI 접속
```bash
# Docker 컨테이너의 Redis CLI 접속
docker exec -it redis-container redis-cli

# 호스트에서 Redis CLI 접속
redis-cli -h localhost -p 6379
```

## 2. Docker Compose 설정

### 2.1 docker-compose.yml 파일 생성
```yaml
version: '3'
services:
  redis:
    image: redis:latest
    container_name: redis-container
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes
    restart: always
    networks:
      - redis-network

volumes:
  redis-data:
    driver: local

networks:
  redis-network:
    driver: bridge
```

### 2.2 Docker Compose 명령어
```bash
# 컨테이너 시작
docker-compose up -d

# 컨테이너 중지
docker-compose down

# 로그 확인
docker-compose logs
```

## 3. Redis 컨테이너 관리

### 3.1 기본 관리 명령어
```bash
# 컨테이너 상태 확인
docker ps -a | grep redis

# 컨테이너 중지
docker stop redis-container

# 컨테이너 시작
docker start redis-container

# 컨테이너 재시작
docker restart redis-container

# 컨테이너 로그 확인
docker logs redis-container
```

### 3.2 Redis 설정 확인
```bash
# Redis 설정 확인
docker exec -it redis-container redis-cli CONFIG GET *

# 메모리 사용량 확인
docker stats redis-container
```

## 4. 보안 설정

### 4.1 Redis 비밀번호 설정
```bash
# 비밀번호가 설정된 Redis 실행
docker run --name redis-container -d \
  -p 6379:6379 \
  -v redis-data:/data \
  redis redis-server --requirepass your_password

# 비밀번호로 Redis CLI 접속
docker exec -it redis-container redis-cli -a your_password
```

### 4.2 네트워크 설정
```bash
# Redis 전용 네트워크 생성
docker network create redis-network

# 네트워크에 Redis 컨테이너 연결
docker network connect redis-network redis-container
```

## 5. 문제 해결

### 5.1 컨테이너 문제 해결
```bash
# Redis 로그 확인
docker logs redis-container

# Redis 컨테이너 상세 정보 확인
docker inspect redis-container

# Redis 프로세스 확인
docker exec redis-container ps aux
```

### 5.2 데이터 백업
```bash
# Redis 데이터 디렉토리 백업
docker cp redis-container:/data /backup/redis-data

# 볼륨 백업
docker run --rm -v redis-data:/data -v /backup:/backup \
  ubuntu tar czf /backup/redis-backup.tar.gz /data