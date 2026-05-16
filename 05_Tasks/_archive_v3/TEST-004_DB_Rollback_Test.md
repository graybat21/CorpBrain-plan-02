---
name: Feature Task
title: "[Feature] TEST-004: 데이터베이스 마이그레이션 롤백 테스트 파이프라인"
labels: 'feature, priority:low, qa'
---

## 🎯 Summary
- 기능명: [TEST-004] 데이터베이스 마이그레이션 롤백 테스트 파이프라인
- 목적: 잦은 스키마 변경 시 발생할 수 있는 DB 장애를 예방하기 위해, 마이그레이션 `Up` 뿐만 아니라 데이터 손실 없는 `Down(Rollback)` 동작을 검증하는 파이프라인을 구성한다.

## 🔗 References (Spec & Context)
- SRS 문서: §4.2.2 (REQ-NF-011 데이터 복구 파이프라인 보조 검증)

## 📝 Task Breakdown
- [ ] ORM(Prisma 등) 마이그레이션 스크립트를 대상으로 한 독립된 테스트 환경(Test DB Container) 셋업.
- [ ] 신규 DDL(Create/Alter) 적용 후, 더미 레코드 100건 삽입 로직 실행.
- [ ] 롤백(`migrate down` 또는 `revert`) 명령어를 실행하는 스크립트 작성.
- [ ] 롤백 실행 후 스키마 상태가 원상 복구되었는지, 그리고 더미 데이터 삭제 시 제약조건 위반(Foreign Key Constraint Error)과 같은 비정상적인 사이드 이펙트가 없는지 검증하는 Assertion 구성.

## ✅ Acceptance Criteria (BDD)
- **Given**: 스테이징 DB 환경(Docker 컨테이너)에 구버전 스키마가 로드되어 있을 때
- **When**: 신규 마이그레이션 `Up` 후 곧바로 `Down(Rollback)` 명령어를 연달아 실행하면
- **Then**: 어떠한 SQL Syntax Error나 Lock 에러 없이 스키마가 기존 구버전 상태로 완벽히 되돌아와야 한다 (무결점 롤백 보장).

## 🛡️ Constraints (NFRs)
- 인프라 제약: 이 테스트는 실제 운영 DB(Primary DB)에 어떤 사이드 이펙트도 줘서는 안 되므로, 반드시 일회성 격리 컨테이너(Ephemeral Container) 환경에서 실행되어야 한다.
