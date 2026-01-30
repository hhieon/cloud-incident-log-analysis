# Scenario 1 재현 절차

## 사전 조건
- Root 계정 접근 가능
- CloudTrail 활성화 및 로그 수집 설정 완료

## 공격 단계
1. Root 계정으로 AWS 콘솔에 로그인한다.
2. 새로운 IAM User를 생성한다.
3. 생성한 사용자에게 관리자 권한을 부여한다.
4. 해당 사용자에 대해 Access Key를 발급한다.

## 기대 결과
- 각 단계에 대응하는 CloudTrail 관리 이벤트가 생성된다.
