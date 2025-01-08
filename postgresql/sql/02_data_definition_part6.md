# 스키마 (Schemas)

## 1. 스키마 개요

### 1.1 스키마의 목적
- 데이터베이스 객체 구조화
- 이름 충돌 방지
- 보안 경계 설정
- 다중 사용자 환경 지원

### 1.2 스키마 종류
- 사용자 정의 스키마
- public 스키마
- pg_catalog 스키마
- information_schema

## 2. 스키마 관리

### 2.1 스키마 생성
```sql
CREATE SCHEMA myschema;
CREATE SCHEMA myschema AUTHORIZATION myuser;
```

### 2.2 스키마 삭제
```sql
DROP SCHEMA myschema;
DROP SCHEMA IF EXISTS myschema CASCADE;
```

### 2.3 스키마 변경
```sql
ALTER SCHEMA myschema RENAME TO newschema;
ALTER SCHEMA myschema OWNER TO newowner;
```

## 3. 스키마 검색 경로

### 3.1 검색 경로 설정
```sql
SET search_path TO myschema, public;
```

### 3.2 기본 검색 경로
- $user: 현재 사용자와 동일한 이름의 스키마
- public: 공용 스키마

### 3.3 스키마 한정자
```sql
myschema.mytable
```

## 4. 스키마 사용 패턴

### 4.1 다중 사용자 환경
- 사용자별 스키마
- 공유 스키마
- 권한 관리

### 4.2 애플리케이션 분리
- 기능별 스키마
- 모듈식 구조
- 버전 관리

### 4.3 보안 모델
- 스키마 수준 권한
- 역할 기반 접근
- 격리된 환경

## 5. 스키마 모범 사례

### 5.1 명명 규칙
- 일관된 접두사
- 의미있는 이름
- 버전 관리 고려

### 5.2 구조화 전략
- 논리적 그룹화
- 의존성 관리
- 확장성 고려

### 5.3 유지보수
- 정기적인 정리
- 사용하지 않는 객체 제거
- 문서화
