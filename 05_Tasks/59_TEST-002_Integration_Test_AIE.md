---
name: Feature Task
title: "[Feature] TEST-002: 시맨틱 병합 모듈(AIE) Mocking 기반 통합 테스트"
labels: 'feature, priority:high, test'
---

## 🎯 Summary
- 기능명: [TEST-002] 시맨틱 병합 모듈(AIE) Mocking 기반 통합 테스트
- 목적: 전처리(AIE-001) → 병합(AIE-002) → 딥링크 매핑(AIE-003)으로 이어지는 AI 파이프라인 연쇄가 실제 DB나 과도한 LLM 비용 소모 없이 정상적으로 동작하는지 통합 검증(Integration Test)한다.

## 🔗 References (Spec & Context)
- SRS 문서: §4.2.6 (REQ-NF-024)

## 📝 Task Breakdown
- [ ] AIE 파이프라인에 주입될 가상의 입력 데이터 세트(더미 Slack/Jira 이벤트 10건) Fixture 작성.
- [ ] 실제 Gemma LLM 추론 서버를 호출하지 않고, 사전에 정의된 '병합 완료 결과물'을 반환하는 Mocking LLM 어댑터 클래스 구현.
- [ ] In-Memory DB(또는 SQLite)를 띄워 Mock 데이터를 적재한 후, `DraftGenerateService` 를 실행.
- [ ] 반환된 결과물이 `SEMANTIC_DRAFT` 포맷에 부합하는지, 특히 `deeplink_map`이 누락 없이 100% 연결되었는지 검증(Assertion).

## ✅ Acceptance Criteria (BDD)
- **Given**: Mocking LLM 모듈이 설정된 테스트 환경에서
- **When**: AIE 통합 파이프라인 실행(Execute Pipeline) 함수가 호출되면
- **Then**: 외부 API 호출(과금)이나 타임아웃 지연 없이 1초 이내에 테스트가 종료되어야 한다.
- **Then**: 출력 결과물의 딥링크 매핑 무결성과 Diff 충돌 감지 마커 포맷이 예상값과 정확히 일치(Pass)해야 한다.

## 🛡️ Constraints (NFRs)
- 아키텍처 제약: 의존성 주입(DI) 또는 인터페이스 분리 원칙을 준수하여, 테스트 환경에서만 LLM Provider를 Mock 클래스로 안전하게 스위칭할 수 있는 구조여야 한다.
