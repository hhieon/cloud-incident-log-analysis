# Scenario 1 - Root 계정 오남용

## evidence-map과 실제 로그 비교

---

## 1. 문서 목적

본 문서는 Root Persistence  공격 시나리오에 대해

사전에 정의한 evidence-map과 실제 AWS CloudTrail 로그를 비교, 검증한 결과를 정리한다

이를 통해 다음을 검증한다:

- 가설에 포함된 행위들이 실제 CloudTrail 로그로 관측 가능한지
- 각 행위를 탐지 관점에서 어떤 필드로 식별할 수 있는지
- 개별 이벤트들이 체인으로 연결 가능한지

---

## 2. 시나리오 개요

Root Persistence 시나리오

1. Root 계정 콘솔 로그인
2. 신규 IAM User 생성
3. 해당 IAM User에 관리자 권한 부여
4. IAM User에 활성화된 Access Key 생성

시나리오 설명: 공격자가 Root 권한을 이용해 장기 접근 수단을 확보하는 전형적이 지속성 시나리오이다.

---

## 3. 로그 검증 요약

| 단계 | 이벤트 | 검증 결과 |
| --- | --- | --- |
| 1 | ConsoleLogin | 관측됨 |
| 2 | CreateUser | 관측됨 |
| 3 | AttachUserPolicy | 관측됨 |
| 4 | CreateAccessKey | 관측됨 |

모든 가설 단계가 실제 CloudTrail 로그에서 확인 됨으로 체인 연결 가능하다.

---

## 4. 이벤트별 로그 분석

### 4.1 ConsoleLogin - Root 콘솔 로그인 성공 여부

로그 개요

- eventName: ConsoleLogin
- eventSource: signin.amazonaws.com

핵심 필드

| 필드 | 값 | 의미 |
| --- | --- | --- |
| userIdentity,type | Root | 행위 주체가 Root 계정 |
| responseElements.ConsoleLogin | Success | 로그인 성공 |
| additionalEventData.MFAUsed | Yes | MFA 사용 여부 |
| recipientAccountId | <ACCOUNT_ID> | 로그인 대상 계정 |

탐지 관점 해석

- Root 계정이 MFA를 사용하여 콘솔 로그인에 성공함
- Rioot 콘솔 로그인 자체가 고위험 행위
- MFA 사용은 위험도를 낮추지만 이후 행위와 결합시 여전히 중요 신호

### 4.2 CreateUser - IAM User 생성

로그 개요

- eventName: CreateUser
- eventSource: iam.amazonaws.com

핵심필드

| 필드 | 값 | 의미 |
| --- | --- | --- |
| userIdentity.type | Root | 행위 주체가 Root 계정 |
| requestParameters.userName | scenario1-admin | 생성된 IAM User |
| responseElements.user.arn | arn:aws:iam::… | 생성 결과 확인 |

탐지 관점 해석

- Root 계정이 신규 IAM User를 생성함
- Root가 직접 IAM User를 생성하는 행위는 지속성 시도의 초기 단계
- 이후 권한 부여 및 자격증면 생성 이벤트와 target 기준으로 연결 가능

### 4.3 AttachUserPolicy - 관리자 권한 부여

로그 개요

- eventName: AttachUserPolicy
- eventSource: iam.amazonaws.com

 핵심 필드

| 필드 | 값 | 의미 |
| --- | --- | --- |
| requestParameters.userName | scenario1-admin | 권한 부여 대상 |
| requestParameters.policyArn | AdministratorAccess | 최고 권한 정책 |
| userIdentity.type | Root | 행위 주체 |

탐지 관점 해석

- Root 계정이 IAM User에게 관리자 권한을 부여함
- AdministratorAccess는 최고 수준 권한
- IAM User가 곧 Root와 동등한 권한을 갖게 됨
- 권한 상승의 명확한 신호

### 4.4 CreateAccessKey - 장기 자격증명 생성

로그 개요

- eventName: CreateAccessKey
- eventSource: iam.amasonaws.com

 핵심 필드

| 필드 | 값 | 의미 |
| --- | --- | --- |
| requestParameters.userName | scenario1-admin | 자격증명 대상 |
| responseElements.accessKey.status | Active | 사용 가능한 상태 |
| userIdentity.type | Root | 행위 주체 |

탐지 관점 해석

- IAM User에 활성화된 Access Key 생성
- 콘솔 접근 없이 API 접근 가능
- 장기 지속성 확보의 결정적 신호
- 이전 이벤트들과 동일 target 기준 체인 완성

---

## 5. 체인 연결 검증

연결 기준

- Actor 일관성: 모든 이벤트의 userIdentity.type = Root
- Target 일관성: scenario1-admin IAM User
- 시간 흐름: 로그인 → 생성 → 권한 부여 → 자격 증명

체인 요약

 ConsoleLogin(Root) → CreateUser(scenario1 - admin) → AttachUserPolicy(AdministratorAccess) → CreateAccessKey(Active)

- 단일 이벤트가 아닌 연속 행위로 해석 가능
- 탐지 기준 및 룰 정의에 사용 가능한 체인 구조 확인

---

## 6. 결론

- Root Persistence 공격 가설은 실제 CloudTrail 로그에서 재현 가능
- 모든 핵심 행위는 명확한 필드 기반으로 식별 가능
- actor / target / time 기준으로 체인 탐지가 가능함

본 문서는 이후 이벤트 정규화, 탐지 기준 정제, 탐지 룰 정의 단계의 근거 자료로 활용된다.
