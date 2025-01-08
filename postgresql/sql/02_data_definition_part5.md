# 권한과 보안

## 1. 권한 (Privileges)

### 1.1 기본 권한 종류
- SELECT: 데이터 조회
- INSERT: 새 행 추가
- UPDATE: 기존 행 수정
- DELETE: 행 삭제
- TRUNCATE: 테이블 비우기
- REFERENCES: 외래 키 생성
- TRIGGER: 트리거 생성
- CREATE: 새 객체 생성
- CONNECT: 데이터베이스 연결
- TEMPORARY: 임시 테이블 생성
- EXECUTE: 함수 실행

### 1.2 권한 부여
```sql
GRANT SELECT, UPDATE ON products TO user1;
GRANT ALL ON products TO user2;
```

### 1.3 권한 취소
```sql
REVOKE ALL ON products FROM PUBLIC;
REVOKE SELECT ON products FROM user1;
```

## 2. 행 수준 보안 (Row Level Security)

### 2.1 정책 활성화
```sql
ALTER TABLE products ENABLE ROW LEVEL SECURITY;
```

### 2.2 정책 생성
```sql
CREATE POLICY product_access_policy ON products
    FOR SELECT
    USING (created_by = current_user);
```

### 2.3 정책 유형
- FOR SELECT
- FOR INSERT
- FOR UPDATE
- FOR DELETE
- FOR ALL

### 2.4 정책 조건
- USING: 행 가시성
- WITH CHECK: 수정 가능성
- 현재 사용자 확인
- 역할 기반 접근

## 3. 보안 모범 사례

### 3.1 최소 권한 원칙
- 필요한 최소한의 권한만 부여
- 정기적인 권한 검토
- PUBLIC 권한 제한

### 3.2 역할 기반 접근 제어
- 역할 생성
- 권한 그룹화
- 역할 계층 구조

### 3.3 감사와 모니터링
- 접근 로그 유지
- 권한 변경 추적
- 보안 위반 탐지

## 4. 고급 보안 기능

### 4.1 열 수준 보안
- 열 단위 권한 부여
- 민감한 데이터 보호
- 뷰를 통한 접근 제어

### 4.2 동적 정책
- 세션 변수 활용
- 컨텍스트 기반 접근
- 복잡한 비즈니스 규칙
