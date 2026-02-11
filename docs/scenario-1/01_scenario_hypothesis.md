# Scenario 1 - Root 계정 오남용

## 가설 검증 및 탐지 관점 분석

---

## 1. 시나리오 가설

### Root 계정 오남용 기반 지속성 확보 시나리오

공격자가 Root 계정을 장악한 후 다음 행위를 수행한다고 가정한다:

1. Root 계정 콘솔 로그인
2. 신규 IAM User 생성
3. 해당 IAM User에 AdministratorAccess 권한 부여
4. IAM User에 Access Key 생성

이는 권한 위임을 통한 장기 접근 수단 확보 시나리오이다.

---

## 2. 타임라인 검증 (실제 로그 기준)

| 순서 | 이벤트 | 발생 시간 (eventTime) | 검증 결과 |
| --- | --- | --- | --- |
| 1 | ConsoleLogin | 2026-02-05T10:11:32Z | 확인됨 |
| 2 | CreateUser | 2026-02-05T10:13:04Z | 확인됨 |
| 3 | AttachUserPolicy | 2026-02-05T10:14:18Z | 확인됨 |
| 4 | CreateAccessKey | 2026-02-05T10:15:02Z | 확인됨 |

※ 위 시간은 실제 CloudTrail 로그에서 확인한 eventTime 기준으로 기입

모든 행위가 짧은 시간 내 연속적으로 발생함을 확인하였다.

---

## 3. 단일 이벤트 탐지의 오탐 가능성 분석

### 3.1 ConsoleLogin (Root)

- 탐지 조건: userIdentity.type = Root AND eventName = ConsoleLogin AND responseElements.ConsoleLogin = Success
- 정상 발생 가능성: 존재 (관리 목적 로그인)
- 오탐 가능성: Medium
- 위험도: High (Root 로그인 자체가 고위험)

---

### 3.2 CreateUser

- 탐지 조건: userIdentity.type = Root AND eventName = CreateUser
- 정상 발생 가능성: 높음 (온보딩, 자동화 등)
- 오탐 가능성: High
- 단독 탐지 시 신뢰도 낮음

---

### 3.3 AttachUserPolicy

- 탐지 조건: eventName = AttachUserPolicy AND requestParameters.policyArn = AdministratorAccess
- 정상 발생 가능성: 존재
- 오탐 가능성: High

---

### 3.4 CreateAccessKey

- 탐지 조건: eventName = CreateAccessKey
- 정상 발생 가능성: 존재 (키 교체 등)
- 오탐 가능성: Medium
- Root 조건이 추가될 경우 빈도 급감

---

## 4. 체인 탐지 최소 조건 정의

### 체인 탐지란?

공격자가 다음 과정을 완성했는지를 판단하는 최소 이벤트 조합을 의미한다

### 4.1 체인 최소 조건

- 주체 일관성
    - 동일 userIdentity.arn (Root)
    - 동일 세션 또는 동일 계정
- 시간적 응집성
    - 짧은 시간 내 발생 (예: 10~30분 이내)
    - eventTime 기준
- 권한 상태 변화
    - 신규 IAM User 생성
    - AdministratorAccess 부여
- 지속 접근 수단 확보
    - CreateAccessKey
    - Active 상태 자격 증명 발급

---

## 5. 체인 예시 구조

1. Root (ConsoleLogin)
2. CreateUser (scenario1-admin)
3. AttachUserPolicy (AdministratorAccess)
4. CreateAccessKey (Active)

이는 권한 위임 기반 지속성 확보 체인으로 해석 가능하다.

---

## 6. 탐지 룰 정의 (초안)

### Rule Draft

다음 조건이 충족될 경우 High Severity Alert 생성:

- userIdentity.type = Root
- ConsoleLogin 성공 이후
- 30분 이내
    - CreateUser
    - AttachUserPolicy (AdministratorAccess)
    - CreateAccessKey
- 동일 target userName 기준 연결

이는 Root 권한을 이용한 지속적 계정 접근 확보 시도로 판단한다.
