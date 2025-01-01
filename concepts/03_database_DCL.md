# 데이터 제어어(DCL; Data Control Language)의 개념

## 1. 데이터 제어어의 개념

**데이터 제어어(DCL)**는 데이터베이스 관리자가 데이터 보안, 무결성 유지, 병행 제어, 회복 등을 정의하는데 사용하는 언어입니다.

## 2. 데이터 제어어의 유형

데이터 제어어의 유형에는 GRANT, REVOKE가 있습니다.

### 2.1 GRANT

GRANT는 데이터베이스 관리자(DBA)가 사용자에게 데이터베이스에 대한 권한을 부여하는 명령어입니다.

```sql
GRANT 권한 ON 테이블 TO 사용자;
```

### 2.2 REVOKE

REVOKE는 데이터베이스 관리자(DBA)가 사용자에게 부여했던 권한을 회수하기 위한 명령어입니다.

```sql
REVOKE 권한 ON 테이블 FROM 사용자;
```

## 3. 권한의 종류

### 3.1 시스템 권한

- **CREATE USER**: 사용자를 생성할 수 있는 권한
- **DROP USER**: 사용자를 삭제할 수 있는 권한
- **CREATE TABLE**: 테이블을 생성할 수 있는 권한
- **CREATE VIEW**: 뷰를 생성할 수 있는 권한
- **CREATE PROCEDURE**: 프로시저를 생성할 수 있는 권한

### 3.2 객체 권한

| 권한 | 테이블 | 뷰 | 프로시저 |
|------|--------|-----|----------|
| ALTER | O | X | X |
| DELETE | O | O | X |
| EXECUTE | X | X | O |
| INDEX | O | X | X |
| INSERT | O | O | X |
| REFERENCES | O | X | X |
| SELECT | O | O | X |
| UPDATE | O | O | X |

## 4. ROLE

### 4.1 ROLE의 개념

- 사용자에게 허가할 수 있는 권한들의 집합
- 여러 권한을 하나의 ROLE로 묶어서 관리 가능
- 특정 ROLE을 여러 사용자에게 부여 가능

### 4.2 ROLE 관련 명령어

```sql
-- ROLE 생성
CREATE ROLE role_name;

-- ROLE에 권한 부여
GRANT privilege_name TO role_name;

-- 사용자에게 ROLE 부여
GRANT role_name TO user_name;

-- ROLE 삭제
DROP ROLE role_name;
```

## 5. 사용자 관리

### 5.1 사용자 생성 및 삭제

```sql
-- 사용자 생성
CREATE USER user_name
IDENTIFIED BY password;

-- 사용자 삭제
DROP USER user_name;
```

### 5.2 비밀번호 변경

```sql
-- 비밀번호 변경
ALTER USER user_name
IDENTIFIED BY new_password;
```

## 6. 예시

### 6.1 GRANT 예시

관리자가 사용자 앤드류에게 ‘학생’ 테이블에 대해 UPDATE할 수 있는 권한 부여

```sql
GRANT UPDATE ON 학생 TO 앤드류;
```

### 6.2 REVOKE 예시

관리자가 사용자 앤드류에게 ‘학생’ 테이블에 대해 UPDATE 할 수 있는 권한을 회수

```sql
REVOKE UPDATE ON 학생 FROM 앤드류;