# Scenario 1 단계별 CloudTrail 이벤트 매핑 및 최소 탐지 조건 (가설)

※ 본 문서는 공격 시나리오 설계 단계에서
CloudTrail 로그에 대한 가설을 정리한 것이다.

## Step 1. Root 계정 콘솔 로그인
- 기대 이벤트: ConsoleLogin
- 주요 필드(예):
  - userIdentity.type = Root
  - responseElements.ConsoleLogin = Success

- 최소 탐지 조건:
  - Root 계정 로그인 성공 이벤트 발생 시  비정상 행위로 의심 가능

## Step 2. IAM User 생성
- 기대 이벤트: CreateUser
- 주요 필드(예):
  - userIdentity.type = Root
  - eventName = CreateUser

- 최소 탐지 조건:
  - Root 로그인 이후 일정 시간 내 CreateUser 발생

## Step 3. 관리자 권한 부여
- 기대 이벤트:
  - AttachUserPolicy 또는 PutUserPolicy (권한 부여 방식에 따라 이벤트가 달라질 수 있음)
- 주요 필드(예):
  - policyArn = AdministratorAccess

- 최소 탐지 조건:
  - 사용자 생성 또는 권한 변경 이벤트가 연속적으로 발생

## Step 4. Access Key 발급
- 기대 이벤트: CreateAccessKey
- 주요 필드(예):
  - requestParameters.userName

- 최소 탐지 조건:
  - 권한 부여 이후 Access Key 발급 이벤트까지 이어짐