---
name: Feature Task
title: "[Feature] TEST-003: Playwright(또는 Cypress) E2E 테스트 - 초안 승인 플로우"
labels: 'feature, priority:medium, test'
---

## 🎯 Summary
- 기능명: [TEST-003] Playwright(또는 Cypress) E2E 테스트 - 초안 승인 플로우
- 목적: 사용자가 브라우저를 열어 로그인하고 초안을 검토한 뒤 최종 승인 및 위키 발행을 누르는 End-to-End 전체 여정이 정상적으로 이루어지는지 자동화된 브라우저 테스트를 통해 보장한다.

## 🔗 References (Spec & Context)
- SRS 문서: §4.2.6 (REQ-NF-024), CI/CD 배포 조건

## 📝 Task Breakdown
- [ ] 프로젝트에 Playwright(또는 Cypress) 설치 및 E2E 브라우저 테스트 환경 구성.
- [ ] E2E 테스트용 `seed` 데이터(테스트용 계정 1개, 대기 중인 초안 1개)를 DB에 초기화하는 Setup 스크립트 작성.
- [ ] 테스트 시나리오 작성:
  1. 로그인 폼 입력 및 대시보드 진입
  2. 첫 번째 초안 클릭 및 상세 화면 이동
  3. Trust-Anchor 클릭 액션 시뮬레이션
  4. '승인 및 Notion 발행' 버튼 클릭
- [ ] 결과 모니터링: 버튼 클릭 후 Success Toast가 뜨는지, URL이 정상 반환되는지 검증(`expect`).

## ✅ Acceptance Criteria (BDD)
- **Given**: 자동화된 Chromium/Webkit 브라우저가 실행된 상태에서
- **When**: 지정된 "초안 승인 플로우" E2E 테스트 스크립트를 실행하면
- **Then**: 로그인부터 최종 승인 버튼 클릭까지 UI 상의 에러(클릭 불가 상태 등) 없이 무사히 완료되어야 하며, 테스트 리포트에 모두 Green(Pass)이 기록되어야 한다.

## 🛡️ Constraints (NFRs)
- 운영 제약: E2E 테스트는 실행 시간이 길기 때문에 매번의 `git push`마다 실행하지 않고, `main` 브랜치 병합(PR)이나 릴리스 버전에 대해서만 실행하도록 CI 워크플로우를 분리한다.
