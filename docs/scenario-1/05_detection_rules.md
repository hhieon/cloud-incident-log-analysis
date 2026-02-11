# Scenario 1 - Detection Rule Definition

## Root Account Persistence Detection Rules

---

# 1. 문서 목적

본 문서는 Scenario 1 (Root Persistence)에 대한
실제 탐지 룰 정의를 기술한다.

- 단일 이벤트 룰 (Alert Level)
- 다중 이벤트 상관 룰 (Attack Determination Level)

---

# 2. 단일 이벤트 탐지 룰 (Alert Level)

단일 이벤트는 “이상 행위 신호(Alert)”를 발생시키기 위한 룰이다.
공격으로 단정하지 않는다.

---

## 2.1 Rule: Root Console Login

### Rule Name

Root Console Login

### Detection Condition

- action = user_login
- actor.type = root
- result = success

### Time Condition

- 없음 (단일 이벤트)

### Description

Root 계정은 일상적 사용이 금지되어야 하며,
콘솔 로그인 자체가 고위험 보안 이벤트로 간주된다.

이 이벤트는 공격 체인의 시작 신호가 될 수 있다.

### Severity

Medium

---

## 2.2 Rule: Root Creates IAM User

### Rule Name

Root Creates IAM User

### Detection Condition

- action = iam_user_create
- actor.type = root

### Description

Root 계정이 직접 IAM User를 생성하는 행위는
지속성 확보 또는 권한 위임 시도의 초기 단계일 수 있다.

### Severity

Medium

---

## 2.3 Rule: Root Grants Admin Privilege

### Rule Name

Root Grants Admin Privilege

### Detection Condition

- action = iam_privilege_escalation
- actor.type = root
- privilege.level = high

### Description

Root 계정이 IAM User에 AdministratorAccess 수준의
고권한을 부여하는 행위는 공격 체인에서 핵심 단계로 간주된다.

### Severity

High

---

## 2.4 Rule: Root Creates Access Key

### Rule Name

Root Creates Access Key

### Detection Condition

- action = iam_credential_create
- actor.type = root
- credential.status = active

### Description

Access Key는 장기 자격 증명(Long-term Credential)으로,
공격자가 콘솔 접근 없이도 계정을 유지할 수 있도록 한다.

지속성(Persistence) 확보의 중요한 지표이다.

### Severity

High

---

# 3. 다중 이벤트 상관 룰 (Attack Chain Detection)

단일 이벤트는 신호에 불과하다.

아래 조건이 충족되면 공격으로 판정한다.

---

## 3.1 Rule: Root Persistence Chain Detected

### Rule Name

Root Persistence Chain Detected

---

### 사용 이벤트

- Root Console Login
- Root Creates IAM User
- Root Grants Admin Privilege
- Root Creates Access Key

---

## 3.2 Detection Conditions (Correlation Level)

### Primary Condition

- actor.type = root

---

### Required Action Sequence (within 30 minutes)

- action = user_login
    - result = success
- action = iam_user_create
- action = iam_privilege_escalation
    - privilege.level = high
- action = iam_credential_create
    - credential.status = active

---

### Correlation Constraints

- Time Window
    - 모든 이벤트가 30분 이내 발생
- Actor Consistency
    - 모든 이벤트의 actor.type = root
- Target Consistency
    - iam_user_create
    - iam_privilege_escalation
    - iam_credential_create
        
        → target.user가 동일해야 함
        
- Event Order Integrity
    - user_login
        
        → iam_user_create
        
        → iam_privilege_escalation
        
        → iam_credential_create
        
        순서가 유지되어야 함
        

---

# 4. 탐지 구조 요약

| 레벨 | 목적 | 결과 |
| --- | --- | --- |
| 단일 이벤트 룰 | 이상 신호 감지 | Alert |
| 다중 이벤트 룰 | 공격 확정 | High Severity Incident |

---

# 5. 결론

본 탐지 룰은 다음을 분리한다:

- “Root가 로그인했다” → 신호
- “Root가 지속성 계정을 만들었다” → 공격
