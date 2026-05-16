---
name: Feature Task
title: "[Feature] QA-001: K6 기반 10,000건 동시 접속(Webhook) 부하 테스트"
labels: 'feature, priority:medium, qa'
---

## 🎯 Summary
- 기능명: [QA-001] K6 기반 10,000건 동시 접속(Webhook) 부하 테스트
- 목적: 고객사의 피크 타임에 대량의 Slack/Jira 이벤트가 동시에 쏟아지는 상황을 시뮬레이션하여, 시스템이 지연 없이 안정적으로 트래픽을 처리하는지 검증한다.

## 🔗 References (Spec & Context)
- SRS 문서: §4.2.1 (REQ-NF-001)

## 📝 Task Breakdown
- [ ] 오픈소스 부하 테스트 도구인 Grafana K6 스크립트(`load-test.js`) 작성.
- [ ] 10,000건의 가상 Webhook Payload(Slack 포맷)를 생성하여 `/webhook/slack` 엔드포인트로 전송하는 시나리오 구성.
- [ ] Vus(Virtual Users)를 점진적으로 늘려가는 Ramp-up/Ramp-down 설정 (예: 최대 1,000 Vus).
- [ ] 테스트 실행 중 서버 측 CPU, Memory, Redis Queue 길이를 Datadog(INF-004)으로 모니터링.
- [ ] K6 결과 리포트를 통해 목표 p95 Latency 도달 여부 측정.

## ✅ Acceptance Criteria (BDD)
- **Given**: 스테이징 환경이 프로덕션과 동일한 스펙으로 프로비저닝된 상태에서
- **When**: K6 스크립트를 통해 초당 500건 이상의 동시 Webhook 트래픽(총 10,000건)을 주입하면
- **Then**: 95% 이상의 요청(p95)이 2초 이내에 `200 OK` 응답을 받아야 한다.
- **Then**: Redis 인스턴스가 다운되거나 메모리 누수(OOM) 현상이 발생하지 않아야 한다.

## 🛡️ Constraints (NFRs)
- 운영 제약: 부하 테스트는 반드시 운영(Production) 환경과 분리된 완전히 격리된 스테이징(Staging) 환경에서만 수행되어야 실제 고객 데이터나 서비스에 영향을 주지 않는다.
