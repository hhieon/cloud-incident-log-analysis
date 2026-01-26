# cloud-incident-log-analysis
클라우드 침해 로그 분석

## 1. 프로젝트 개요
본 프로젝트는 AWS CloudTrail 로그를 활용하여 
클라우드 환경에서 발생하는 침해 행위를 사후 분석하는 팀 프로젝트이다.

CloudTrail에 기록되는 관리 이벤트(API Call)를 기반으로  
권한 상승, 계정 악용 등 침해 행위의 로그 증거를 수집·분석한다.

## 2. 프로젝트 목적
본 프로젝트는 AWS CloudTrail 로그를 기반으로 
클라우드 환경에서 발생하는 침해 행위의 흔적을 식별하고 
로그 증거를 체계적으로 정리하는 것을 목표로 한다.

기존 연구가 사후 수집된 로그 분석에 초점을 맞췄다면,  
본 프로젝트는 실제 공격을 재현하여 로그를 직접 생성하고,  
해당 로그가 침해 행위의 증거로 활용 가능한지를 실험적으로 검증한다.

## 3. 프로젝트 범위
## 4. 전체 구조

## 5. 프로젝트 구성 및 역할
### ta3h1. Cloud Security Engineer
- AWS 환경 구성
- CloudTrail 설정
- 로그 수집 파이프라인 구축

### hyehyej. Detection Engineer
- CloudTrail 로그 구조 분석
- 이벤트 정규화
- 탐지 룰 구현

### hhieon. Cloud Attack & System Analyst
- 공격 시나리오 설계
- 침해 흐름 모델링
- 탐지 기준 정의

## 6. 실험 시나리오 개요
## 7. 로그 및 증거물 관리 방식
## 8. 탐지 로직 개요
## 9. 재현 방법
## 10. 결과 요약
## 11. 한계 및 향후 과제

## 12. 참고 문헌
Investigating Advanced Persistent Threat Tactics in Cloud Environments: 
A Forensic Study of AWS CloudTrail Log Data