# Scenario 1 - Detection Criteria Refinement

## 탐지 기준 정제 (Single vs Multi Event Detection)

---

## 1. 문서 목적

본 문서는 Scenario 1 (Root Account Persistence)에 대해

단일 이벤트 탐지 기준과 다중 이벤트(체인) 탐지 기준을 명확히 정의한다.

목적은 다음과 같다:

- 경보(Alert)와 공격 판정(Incident)을 구분
- 단일 이벤트 기반 신호와 체인 기반 확정 기준을 분리
- 오탐(False Positive)을 최소화하는 탐지 조건 정의

---

# 2. 단일 이벤트 탐지 기준 (Alert Level)

단일 이벤트 탐지는 “이상 행위 신호”를 빠르게 감지하기 위한 트리거이다.

이는 즉시 공격으로 판정하지 않는다.

---

## 2.1 Root Console Login 탐지

### 탐지 조건

- action = user_login
- actor.type = root
- result = success

### 의미

Root 계정의 콘솔 로그인은 고위험 행위로 간주된다.

### False Positive 고려

- 긴급 운영 작업
- 계정 점검
- 보안 감사 활동

---

## 2.2 Root IAM User 생성

### 탐지 조건

- action = iam_user_create
- actor.type = root

### 의미

Root 계정이 신규 IAM User를 생성하는 행위는 권한 위임의 시작일 수 있다.

### False Positive 고려

- 신규 계정 온보딩
- 인프라 초기 세팅
- 자동화 스크립트 실행

---

## 2.3 관리자 권한 부여

### 탐지 조건

- action = iam_privilege_escalation
- actor.type = root
- privilege.level = high

### 의미

Root 계정이 IAM User에 AdministratorAccess 수준의 권한을 부여하는 행위

### False Positive 고려

- 권한 수정 작업
- 장애 대응 중 권한 임시 부여

---

## 2.4 Access Key 생성

### 탐지 조건

- action = iam_credential_create
- actor.type = root
- credential.status = active

### 의미

IAM User에 장기 자격증명(Access Key)을 생성하는 행위

### False Positive 고려

- 키 로테이션
- 자동화 도구 구성

---

# 3. 다중 이벤트 탐지 기준 (Attack Determination)

다중 이벤트 탐지는 개별 신호가 결합되어

“공격 의도”가 명확해지는 경우를 정의한다.

---

## 3.1 Root Persistence 체인 정의

다음 조건이 모두 충족될 경우 공격으로 판단한다:

1. Root 계정 로그인
2. 신규 IAM User 생성
3. 해당 사용자에 AdministratorAccess 부여
4. 해당 사용자에 Active Access Key 생성

---

## 3.2 체인 판정 조건

- 시간적 응집성
    - 위 이벤트들이 짧은 시간 내 발생 (예: 30분 이내)
- 주체 일관성
    - 모든 이벤트의 actor.type = root
- Target 일관성
    - CreateUser → AttachUserPolicy → CreateAccessKey의
    [target.name](http://target.name/) 이 동일

---

# 4. 탐지 레벨 구분

| 구분 | 목적 | 판단 수준 |
| --- | --- | --- |
| 단일 이벤트 탐지 | 이상 행위 신호 감지 | Alert |
| 다중 이벤트 탐지 | 공격 여부 판정 | High Severity Incident |

---

# 5. 탐지 기준 정제의 의미

이 문서의 목적은 다음을 분리하는 것이다:

- “누가 접속했는가?” → 경보
- “공격 의도가 있었는가?” → 판정

단일 이벤트는 신호(Signal)

다중 이벤트는 공격 확정(Determination)이다.

이 기준은 이후 Detection Rule 설계의 기반이 된다.
