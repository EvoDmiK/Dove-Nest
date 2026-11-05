---
created: 2026-11-06
updated: 2026-11-06
type: reference
category: "관측성"
status: draft
source_project: "Birds-Nest"
source_work_log: "[[Work/Birds-Nest/작업기록/Swallow/2026-11-05-Swallow-SW015-021-운영-배포]]"
tags:
  - dev
  - draft
  - "birds-nest"
---

# Swallow SW-015~021 운영 배포

> 자동 증류 초안 · 원천 [[Work/Birds-Nest/작업기록/Swallow/2026-11-05-Swallow-SW015-021-운영-배포]] · 분류 근거 {"인프라":0,"자동화":7,"관측성":8,"데이터":0}

## 1. 개요 및 목적

사용자 요청으로 NAS에 DB 마이그레이션 002~005, MCP 이미지 `4ef45b3`, 뉴스·시세 n8n 워크플로우 및 Grafana 대시보드를 반영했다. 운영 검증과 잔여 사항은 아래에 구분한다.

| 구분 | 결과 |
| --- | --- |
| 실서비스 배포 | DB migrations 002~005, MCP `4ef45b3`, n8n 49·26노드 반영 |
| 형상 관리 | PR #10, `EvoDmiK/dev-swallow`; Grafana Git Sync도 임시로 해당 브랜치를 추적 |
| 검증 | 시세 예약 75884 success 및 리스크 노드 실행, 뉴스 예약 75885 success, MCP 인증과 포트폴리오 읽기 성공 |
| 자동 청산 | 배포 후 장중 모의 포지션 15건(익절 6·손절 9) 청산. 실제 증권 주문이 아님 |

## 2. 핵심 아키텍처 및 원칙

- 요약
- 작업 및 변경 결과 요약
- 1. 작업 배경 및 문제 분석
- 2. 해결 방안 및 설계
- 3. 주요 구현 내용
- 4. 검증 및 테스트 결과
- 5. 배포 및 형상 관리
- 6. 한계 및 후속 과제
- 관련 문서
- 독립 검증 중간 결과
- 독립 검증 완료 — 10:15 이후

## 3. 재사용 가능한 코드 스니펫 / 설정

_원천 문서에 코드 블록 없음_

## 4. 검증 근거 및 트러블슈팅

사용자 요청으로 NAS에 DB 마이그레이션 002~005, MCP 이미지 `4ef45b3`, 뉴스·시세 n8n 워크플로우 및 Grafana 대시보드를 반영했다. 운영 검증과 잔여 사항은 아래에 구분한다.

| 구분 | 결과 |
| --- | --- |
| 실서비스 배포 | DB migrations 002~005, MCP `4ef45b3`, n8n 49·26노드 반영 |
| 형상 관리 | PR #10, `EvoDmiK/dev-swallow`; Grafana Git Sync도 임시로 해당 브랜치를 추적 |
| 검증 | 시세 예약 75884 success 및 리스크 노드 실행, 뉴스 예약 75885 success, MCP 인증과 포트폴리오 읽기 성공 |
| 자동 청산 | 배포 후 장중 모의 포지션 15건(익절 6·손절 9) 청산. 실제 증권 주문이 아님 |

기존 운영 n8n의 credential과 webhook 식별자를 보존하고 각 워크플로우를 비활성화·기존 실행 종료 확인 후 업데이트·재활성화했다. 오류 시 이전 정의 복원 경로를 마련했다. DB 적용 전 두 DB의 덤프를 확보했다.

마이그레이션 002~005는 각각 트랜잭션으로 순차 적용. MCP는 기존 이미지와 독립된 불변 릴리스로 빌드·배포. Grafana 테스트 파일을 Git Sync 감시 폴더 밖으로 이동한 `4cc06ec`를 push했다.

- NAS 백업 `data.dump`, `n8n.dump`는 `pg_restore -l` 목록 검증을 통과했다. 복원 시험은 하지 않았다.
- 뉴스 첫 예약 실행 75885: success, DB 처리 이력 13 done / 1 retry. 공공데이터 티커 조회 노드에 오류 1건이 있었으며 retry 원인 및 후속 완료는 추가 확인이 필요하다.
- 시세 실행 75884: success, 리스크 평가 준비와 보유 리스크 평가 실행. 관측된 자동 청산 손익률은 익절 +5.08% 이상, 손절 -3.61% 이하.
- MCP 컨테이너 healthy, 인증된 목록 13개 및 포트폴리오 읽기 성공.
- Grafana Git Sync는 `4cc06ec` 기준 success, stock-monitor 패널 8 신규 쿼리 반영. 브라우저 렌더링은 미검증.
- Grafana SQL 테스트 이동 후 8/8 통과.

## 5. 관련 Dev 가이드

- 원천 작업기록 [[Work/Birds-Nest/작업기록/Swallow/2026-11-05-Swallow-SW015-021-운영-배포]]
- [[Dev/index|Dev 인덱스]]
