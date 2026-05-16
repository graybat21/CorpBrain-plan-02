---
name: Feature Task
title: "[Feature] QA-002: 보안 취약점 점검 (OWASP Top 10 스캐닝)"
labels: 'feature, priority:high, qa'
---

## 🎯 Summary
- 기능명: [QA-002] 보안 취약점 점검 (OWASP Top 10 스캐닝)
- 목적: 서비스 출시 전 어플리케이션 전반에 걸친 보안 취약점(XSS, SQL Injection, CSRF 등)을 식별하고 패치하여 B2B 컴플라이언스를 충족한다.

## 🔗 References (Spec & Context)
- SRS 문서: §4.2.3 (REQ-NF-015)

## 📝 Task Breakdown
- [ ] Trivy를 활용한 컨테이너/코드 정적 보안 테스트(SAST) CI 파이프라인 연동 (`trivy fs --severity HIGH,CRITICAL .`).
- [ ] ZAP(Zed Attack Proxy) Baseline Scan을 활용한 동적 보안 스캐닝(DAST)으로 주요 API 엔드포인트 점검 (`zap-baseline.py -t <target_url>`).
- [ ] `npm audit --audit-level=high`을 통한 서드파티 패키지(Dependencies)의 취약성 점검.
- [ ] 스캐닝 결과 발견된 Critical / High 등급의 보안 결함 수정 패치 반영.

## ✅ Acceptance Criteria (BDD)
- **Given**: 애플리케이션 코드가 Release Candidate(RC) 상태로 도달했을 때
- **When**: 전체 보안 취약점 스캐닝 스크립트(`npm run security:scan`)를 실행하면
- **Then**: 리포트 결과 상 Critical 및 High 등급의 취약점이 "0"건이어야 한다.
- **Then**: SQL Injection이나 XSS 등 OWASP Top 10 주요 위협 항목에서 방어 로직이 정상 작동함이 확인되어야 한다.

## 🛡️ Constraints (NFRs)
- 비즈니스 제약: 이 스캐닝 테스트를 통과하지 못한 코드는 어떤 상황에서도 프로덕션(Production) 브랜치로 병합(Merge)될 수 없다 (Hard Constraint).
