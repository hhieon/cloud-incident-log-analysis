# Scenario 01 – Root 계정 오남용 타임라인

## Environment

- AWS 리전: ap-northeast-2 (Seoul)
- CloudTrail: 활성화됨 (관리 이벤트 수집)
- 로그 저장소: S3

---

## Timeline

### 1. Root 콘솔 로그인

- 행위 주체: Root 계정
- 행위 내용: AWS Management Console 로그인
- 실행 시각: 2026-02-05 14:12 (KST)

---

### 2. IAM 사용자 생성

- 행위 주체: Root 계정
- 행위 내용: IAM 사용자 생성
- 사용자 이름: scenario1-admin
- 실행 시각: 2026-02-05 14:17 (KST)

---

### 3. 관리자 권한 부여

- 행위 주체: Root 계정
- 행위 내용: IAM 사용자에게 관리자 권한 정책 연결
- 정책 이름: AdministratorAccess
- 대상 사용자: scenario1-admin
- 실행 시각: 2026-02-05 14:17 (KST)

---

### 4. Access Key 생성

- 행위 주체: Root 계정
- 행위 내용: IAM 사용자에 대한 Access Key 생성
- 대상 사용자: scenario1-admin
- 실행 시각: 2026-02-05 14:21 (KST)

---

## Summary

본 시나리오는 Root 계정으로 IAM 사용자를 생성한 뒤 관리자 권한을 부여하고, Access Key를 발급하는 과정을 재현한 것이다.
이러한 행위는 계정 탈취나 권한 남용으로 이어질 수 있어 보안 측면에서 주의가 필요하다.
