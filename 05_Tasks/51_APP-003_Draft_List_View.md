---
name: Feature Task
title: "[Feature] APP-003: HITL 대시보드 - 초안 리스트 뷰 및 필터 컴포넌트"
labels: 'feature, priority:medium, frontend'
---

## 🎯 Summary
- 기능명: [APP-003] HITL 대시보드 - 초안 리스트 뷰 및 필터 컴포넌트
- 목적: 실무자가 자신이 검토하고 승인해야 할 문서 초안들의 목록을 효율적으로 탐색하고 상태별로 분류할 수 있는 대시보드 메인 화면을 구현한다.

## 🔗 References (Spec & Context)
- SRS 문서: §4.3 UI/UX Design (DES-003 HITL Dashboard)

## 📝 Task Breakdown
- [ ] Next.js Server Component를 활용하여 API-R002(초안 목록 조회 API)를 호출하는 메인 대시보드 페이지(`/drafts`) 구현.
- [ ] 리스트 아이템 렌더링: 각 초안 카드는 제목(또는 요약), 타임스탬프, 연동 채널 아이콘(Slack/Jira), 상태(`Pending`, `Approved`, `Rejected`)를 직관적인 Badge(APP-002)로 표시.
- [ ] 상단 필터바 컴포넌트 작성: 상태별 탭 필터링 및 생성일자 범위(Date Picker) 검색 기능.
- [ ] SWR 또는 React Query(혹은 Next.js 14 네이티브 캐싱/재검증 로직)를 적용하여 목록 페이징 및 무한 스크롤(또는 페이지 번호) 처리.
- [ ] SSE(API-M002)를 구독하여 새로운 초안이 생성되었을 때 알림(Toast)을 띄우고 리스트를 갱신(Refetch)하는 로직 구현.

## ✅ Acceptance Criteria (BDD)
- **Given**: HITL 대시보드 메인 화면에 접속한 사용자가
- **When**: '대기 중(Pending)' 상태 탭을 클릭하면
- **Then**: 승인 대기 중인 초안 목록만 필터링되어 화면에 렌더링되어야 한다.
- **When**: 백그라운드에서 새로운 초안 생성이 완료되면
- **Then**: 화면 우측 상단에 "새 초안 생성 완료" 토스트 메시지가 나타나고, '새로고침' 버튼이 노출되어야 한다.

## 🛡️ Constraints (NFRs)
- 성능 제약: 리스트 렌더링 시 이미지 로딩이나 불필요한 재렌더링을 방지하여 목록 스크롤 프레임 레이트를 60fps로 부드럽게 유지해야 한다.
