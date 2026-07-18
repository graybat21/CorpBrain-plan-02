# GitHub Issue Specifications - Epic 7: 분석 통계 및 공통 셸 (Analytics & App Shell)

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] APP-UI-01: 전체 앱 레이아웃 및 디자인 시스템 기초 공사"
labels: 'feature, frontend, priority:highest'
assignees: ''
---

## :dart: Summary
- 기능명: [APP-UI-01] App Shell 및 라우팅 설정
- 목적: 애플리케이션의 최상위 진입점이자 뼈대가 되는 공통 레이아웃(좌측 사이드바, 우측 컨텐츠 영역 등), 클라이언트 라우터, 그리고 디자인 시스템(Color/Typography)을 구축한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: `§3.6 (UX/UI 디자인 방향성)`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] React/Next.js 라우터(Router) 초기화 및 `/, /dashboard, /workspace/:id` 등 경로 정의
- [ ] 글로벌 CSS(혹은 Tailwind/CSS-in-JS) 토큰 및 컬러 팔레트 세팅
- [ ] 레이아웃 컴포넌트(Sidebar, Top Nav, Main Section) 구조화
- [ ] 일렉트론(Electron) 또는 데스크탑 WebView 특성을 고려한 `-webkit-app-region: drag` 타이틀바 적용

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 기본 레이아웃 렌더링
- Given: 빌드된 프론트엔드 환경
- When: 루트 페이지(`/`)에 접근함
- Then: 좌측에 빈 히스토리 패널 영역과 우측 중앙에 "워크스페이스를 선택하세요" 등의 기본 레이아웃 뼈대가 정상적으로 노출된다.

## :gear: Technical & Non-Functional Constraints
- 심미성: 데스크탑 네이티브 앱과 같은 부드러운 전환과 깔끔한 여백(Margin/Padding) 준수

## :checkered_flag: Definition of Done (DoD)
- [ ] Linter/Formatter (Prettier, ESLint) 오류 없는가?

## :construction: Dependencies & Blockers
- Depends on: None
- Blocks: WS-FE-01, ANA-FE-01, LLM-FE-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] STAT-CMD-01: 통계 이벤트 발생 시 수치 로깅 및 DB Insert"
labels: 'feature, backend, priority:low'
assignees: ''
---

## :dart: Summary
- 기능명: [STAT-CMD-01] 사용자 액션 트래킹 (Command)
- 목적: "자동 문서 갱신(Watcher)", "딥링크 기반 빠른 열기" 등의 이벤트가 발생할 때마다 횟수 및 관련 로그를 DB에 적재하여, 추후 '절약 시간' 통계를 내기 위한 원천 데이터를 쌓는다.

## :link: References (Spec & Context)
- SRS 문서: `REQ-FUNC-028, 030`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] `Analytics` 테이블 또는 로그 컬렉션 스키마 확정
- [ ] DL-CMD-02(딥링크 열기) 및 WA-CMD-03(위키 재생성) 완료 직후 통계 기록 트리거(로깅) 추가
- [ ] 비동기 Fire-and-Forget 방식으로 DB Insert 실행 (메인 비즈니스 로직 성능 저하 방지)

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 딥링크 클릭 로깅
- Given: 백엔드 서버가 켜져 있음
- When: 딥링크 실행 커맨드가 정상적으로 처리됨
- Then: `Analytics` 로그 테이블에 액션 타입(`DEEP_LINK_CLICK`), 타임스탬프가 포함된 레코드가 성공적으로 적재된다.

## :construction: Dependencies & Blockers
- Depends on: DL-CMD-02, WA-CMD-03
- Blocks: STAT-QRY-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] STAT-QRY-01 / STAT-TEST-01: WPM 기반 통계 산출 및 테스트"
labels: 'feature, backend, priority:low'
assignees: ''
---

## :dart: Summary
- 기능명: [STAT-QRY-01] 대시보드 통계 집계 (Query)
- 목적: 로깅된 액션과 추출된 문서량을 기반으로 '시간 절약(Time Saved)' 및 '압축률' 지표를 계산하여 프론트엔드로 반환한다.

## :link: References (Spec & Context)
- SRS 문서: `REQ-FUNC-027, 029`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 총 추출 단어 수를 250 WPM 공식으로 나누어 '문서 읽는 데 절약한 시간' 도출 로직 구현
- [ ] "총 텍스트 토큰" 대비 "위키 토큰" 비율을 계산하여 압축률 산정
- [ ] 딥링크 클릭 수, 워처 갱신 수 SUM 쿼리 작성 및 DTO 반환
- [ ] (STAT-TEST-01) WPM 공식(예: 단어 수 25,000 / 250 = 100분)이 정확한 결과값을 반환하는지 검증하는 단위 테스트 작성

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 올바른 WPM 계산
- Given: 총 추출된 텍스트 볼륨이 250,000 단어임
- When: 대시보드 통계를 조회함
- Then: 절약된 시간 필드가 '1,000분' (혹은 환산된 시간)으로 반환된다.

## :construction: Dependencies & Blockers
- Depends on: STAT-CMD-01
- Blocks: STAT-FE-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] STAT-FE-01: My Analytics 차트 및 4대 지표 대시보드 UI 렌더링"
labels: 'feature, frontend, priority:low'
assignees: ''
---

## :dart: Summary
- 목적: 분석된 통계 지표(시간 절약, 압축률 등)를 게이미피케이션 요소가 포함된 직관적인 대시보드 UI로 렌더링한다.

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 통계 DTO를 수신하여 4가지 주요 지표(카드 위젯 형태) 렌더링
- [ ] 필요 시 Recharts 등 경량 차트 라이브러리 연동
- [ ] 데이터 미존재(Empty State) 시의 기본 화면 구현

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 통계 카드 렌더링
- Given: API에서 정상적인 통계 수치가 반환됨
- When: 대시보드 페이지 렌더링이 완료됨
- Then: 4개의 지표 카드(읽기 시간 절약, 문서 압축률 등)가 애니메이션과 함께 표출된다.

## :construction: Dependencies & Blockers
- Depends on: STAT-QRY-01, API-003
