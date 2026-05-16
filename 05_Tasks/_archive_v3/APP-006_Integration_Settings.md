---
name: Feature Task
title: "[Feature] APP-006: 연동(Integration) 설정 대시보드"
labels: 'feature, priority:medium, frontend'
---

## 🎯 Summary
- 기능명: [APP-006] 연동(Integration) 설정 대시보드 (Slack/Jira/Notion/Stripe)
- 목적: 외부 SaaS 앱(수집 채널 및 발행 채널)들과의 OAuth 연동 상태를 한눈에 파악하고, 연결(Connect) 및 해제(Disconnect) 액션을 수행할 수 있는 설정 뷰를 제공한다.

## 🔗 References (Spec & Context)
- SRS 문서: §3.1 External Systems

## 📝 Task Breakdown
- [ ] 설정 페이지 내 `앱 연동` 탭 라우팅(`/settings/integrations`) 구성.
- [ ] 각 연동 대상(Slack, Jira, Notion, Confluence)별 설정 카드 UI 컴포넌트 개발 (아이콘, 상태 뱃지, 설명 텍스트 포함).
- [ ] 미연결 상태일 때 '연결하기' 버튼을 클릭 시 각 서비스의 OAuth 인가 서버(Authorization URL)로 리다이렉트하는 플로우 연동.
- [ ] 연결 완료 상태일 때 '연결 해제' 모달 플로우 추가.
- [ ] 현재 워크스페이스의 요금제(Plan) 상태를 확인하고 Stripe 결제 포털(Customer Portal)로 진입할 수 있는 '구독 관리' 섹션 추가.

## ✅ Acceptance Criteria (BDD)
- **Given**: 워크스페이스에 아직 Slack이 연동되지 않은 상태에서
- **When**: 사용자가 앱 연동 화면에서 Slack 카드의 'Connect' 버튼을 클릭하면
- **Then**: Slack OAuth 인증 페이지로 이동하며, 인증 성공 후 다시 돌아오면 카드의 상태가 'Connected(활성)'로 변경되어야 한다.

## 🛡️ Constraints (NFRs)
- UI/UX 제약: 연동 해제(Disconnect)와 같은 파괴적인 액션은 사용자의 실수를 방지하기 위해 이중 확인(Double Confirmation) 모달을 반드시 거쳐야 한다.
