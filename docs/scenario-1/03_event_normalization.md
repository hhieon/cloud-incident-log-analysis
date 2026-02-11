# Scenario 1 - Event Normalization

## 탐지 관점 이벤트 정규화 설계

---

## 1. 문서 목적

본 문서는 Scenario 1 (Root Account Persistence)에서
관측된 CloudTrail 이벤트를 탐지 시스템에서 활용 가능한
공통 포맷으로 정규화하는 기준을 정의한다.

정규화의 목적은 다음과 같다

- 서로 다른 AWS 이벤트를 공통 action 체계로 통합
- 탐지 룰에서 재사용 가능한 actor / target / time 구조 정의
- 단일 이벤트 탐지를 체인 탐지로 확장 가능하도록 설계

---

## 2. 정규화 필드 정의

정규화는 다음 4가지 핵심 필드를 기준으로 수행한다

| 필드 | 의미 |
| --- | --- |
| action | 행위의 의미 단위 |
| actor | 행위를 수행한 주체 |
| target | 영향을 받은 대상 |
| time | 이벤트 발생 시각 |

필요 시 result / privilege / credential 등
보조 필드를 추가적으로 포함한다.

---

## 3. Action 매핑 정의

| CloudTrail eventName | Normalized action |
| --- | --- |
| ConsoleLogin | user_login |
| CreateUser | iam_user_create |
| AttachUserPolicy | iam_privilege_escalation |
| CreateAccessKey | iam_credential_create |

---

## 4. 이벤트별 정규화 예시

### 4.1 ConsoleLogin → user_login

```json
{
  "action": "user_login",
  "actor": {
    "type": "root",
    "id": "<ACCOUNT_ID>"
  },
  "target": {
    "type": "account",
    "id": "<ACCOUNT_ID>"
  },
  "result": "success",
  "auth": {
    "mfa_used": true
  },
  "time": "2026-02-05T05:12:11Z"
}
```

### 4.2 CreateUser → iam_user_create

```json
{
	"action": "iam_user_create",
	"actor": {
	"type": "root",
	"id": "<ACCOUNT_ID>"
	},
	"target": {
		"type": "iam_user",
		"name": "scenario1-admin"
	},
	"time": "2026-02-05T05:17:21Z"
}
```

### 4.3 AttachUserPolicy → iam_privilege_escalation

```json
{
  "action": "iam_privilege_escalation",
  "actor": {
    "type": "root",
    "id": "<ACCOUNT_ID>"
  },
  "target": {
    "type": "iam_user",
    "name": "scenario1-admin"
  },
  "privilege": {
    "policy": "AdministratorAccess",
    "level": "high"
  },
  "time": "2026-02-05T05:17:21Z"
}
```

### 4.4 CreateAccessKey → iam_credential_create

```json
{
  "action": "iam_credential_create",
  "actor": {
    "type": "root",
    "id": "<ACCOUNT_ID>"
  },
  "target": {
    "type": "iam_user",
    "name": "scenario1-admin"
  },
  "credential": {
    "type": "access_key",
    "status": "active"
  },
  "time": "2026-02-05T05:21:35Z"
}
```

## 5. 정규화 결과 해석

정규화 이후 이벤트 흐름은 다음과 같이 해석된다:

1. user_login (Root, success, MFA)
2. iam_user_create (scenario1-admin)
3. iam_privilege_escalation (AdministratorAccess)
4. iam_credential_create (Active Access Key)

이는 다음과 같은 체인 탐지 문장으로 변환 가능하다
<Root 계정 로그인 이후, 신규 IAM User 생성 → 관리자 권한 부여 → 장기 자격증명 생성이 연속 발생>
