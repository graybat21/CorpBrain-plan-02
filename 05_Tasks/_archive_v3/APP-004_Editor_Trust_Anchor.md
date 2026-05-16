---
name: Feature Task
title: "[Feature] APP-004: HITL 에디터 - Diff Viewer 및 Trust-Anchor 클릭 하이라이팅 컴포넌트"
labels: 'feature, priority:high, frontend'
---

## 🎯 Summary
- 기능명: [APP-004] HITL 에디터 - Diff Viewer 및 Trust-Anchor 클릭 하이라이팅 컴포넌트
- 목적: CorpBrain의 핵심 경쟁력인 '할루시네이션(환각) 방지'를 시각적으로 증명하기 위해, 사용자가 초안을 읽으며 의심되는 문장을 클릭할 경우 즉시 원본 메시지로 이동(또는 우측 패널에 원본 노출)하는 에디터를 구현한다.

## 🔗 References (Spec & Context)
- SRS 문서: §4.3 UI/UX Design (DES-001 Trust-Anchor, DES-003), §4.1.2 (REQ-FUNC-008 충돌 Diff)

## 📝 Task Breakdown
- [ ] 상세 화면(`/drafts/[id]`) 레이아웃 구성: 좌측(초안 에디터 패널) / 우측(원본 소스 및 메타데이터 패널) 분할 레이아웃(Split Pane).
- [ ] 본문(Markdown) 렌더링 시, `deeplink_map` 데이터와 매칭되는 텍스트 Span에 시각적 하이라이트(또는 점선 밑줄) 스타일링 컴포넌트(Trust-Anchor) 적용.
- [ ] 하이라이트된 문장 클릭 시: 이벤트 핸들러를 통해 우측 패널에 원본 메시지 전문과 발화자 정보를 띄우거나 외부 딥링크(Slack/Jira) 새 창 열기 액션 수행.
- [ ] AIE-002b에서 반환된 문맥 충돌(Conflict) 발생 구간 감지 시, `monaco-editor`의 Diff 모드 또는 자체적인 Side-by-side Diff Viewer 컴포넌트를 렌더링하여 두 의견(A/B)을 대조.
- [ ] 하단 '승인(Publish)' 및 '반려' Action 바 배치 및 API-W003 통신 연동.

## ✅ Acceptance Criteria (BDD)
- **Given**: 실무자가 상세 에디터 화면에서 AI가 요약한 초안 텍스트를 읽고 있을 때
- **When**: 파란색 점선이 쳐진 특정 문장(Trust-Anchor)을 마우스로 클릭하면
- **Then**: 우측 패널에 해당 요약의 근거가 된 Slack 원본 메시지 작성자, 시간, 채널 링크가 즉각적으로 표시되어야 한다.
- **When**: 문맥 충돌 마커가 있는 부분을 발견했을 때
- **Then**: 두 가지 상반된 의견이 나란히(Diff 형식으로) 시각적으로 렌더링되어 사용자가 어느 쪽을 채택할지 선택할 수 있는 UI가 제공되어야 한다.

## 🛡️ Constraints (NFRs)
- UI/UX 제약: Trust-Anchor를 나타내는 하이라이트 UI는 사용자의 독해를 방해하지 않는 옅은 색상(예: 텍스트에 연한 밑줄)이어야 하며, 마우스 호버(Hover) 시 상호작용 피드백을 제공해야 한다.
