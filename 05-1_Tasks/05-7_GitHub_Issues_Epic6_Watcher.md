# GitHub Issue Specifications - Epic 6: 실시간 데몬 (Watcher)

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] WA-CMD-01: Watcher 설정 모드(수동/실시간/유휴) 변경 및 DB 저장"
labels: 'feature, backend, priority:low'
assignees: ''
---

## :dart: Summary
- 기능명: [WA-CMD-01] Watcher 동작 모드 관리 (Command)
- 목적: 파일 변경 사항을 감지하는 Watcher 데몬의 동작 모드(Manual, Real-time, Idle) 설정을 사용자가 변경하고, 이를 영구 저장소에 반영한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: `REQ-FUNC-023`
- DTO 명세: `API-003`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] Watcher Mode Enum 값(Manual, Real-time, Idle) 정의
- [ ] 설정값 수신 시 `Settings_Meta` 테이블 갱신 
- [ ] 갱신 즉시 백그라운드 Watcher 프로세스의 큐 처리 주기를 동적으로 변경하는 Pub/Sub 또는 시그널(Signal) 로직 연동

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 설정 변경 적용
- Given: 현재 Watcher 모드가 '수동(Manual)'임
- When: '실시간(Real-time)'으로 모드 변경을 요청함
- Then: DB에 설정이 반영되며, Watcher 데몬이 재시작되거나 큐 타이머가 즉시 활성화(0초)된다.

## :construction: Dependencies & Blockers
- Depends on: DB-001
- Blocks: WA-CMD-02

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] WA-CMD-02 / WA-TEST-01/02: 이벤트 감지, 디바운싱, 타임스탬프 대조 로직 및 테스트"
labels: 'feature, backend, priority:high, daemon'
assignees: ''
---

## :dart: Summary
- 기능명: [WA-CMD-02] 파일 시스템 이벤트 감지
- 목적: OS 단의 파일 변경 이벤트를 감지(Watch)하되, 무의미한 중복 이벤트(단순 속성 변경 등)를 필터링(Debounce)하고 실 내용 변경 시에만 큐에 적재한다.

## :link: References (Spec & Context)
- SRS 문서: `REQ-FUNC-024, 026`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] Python `watchdog` 라이브러리(혹은 Node.js `chokidar`)를 사용하여 지정된 워크스페이스 디렉토리 감시 옵저버 세팅
- [ ] Modified 이벤트 발생 시 500ms~1000ms 디바운싱(Debouncing) 적용
- [ ] DB의 `File_Meta.updated_at` 타임스탬프와 OS 물리 파일의 수정 시간을 대조(Check)
- [ ] 내용이 실제로 수정되었을 경우 처리 대기 큐(Queue)에 적재
- [ ] (WA-TEST-01) 파일 내용 변경 없이 껍데기만 터치된 경우 필터링(Skip) 되는지 단위 테스트
- [ ] (WA-TEST-02) 유휴(Idle) 모드일 때 N분간 입력(이벤트)이 없을 시 큐가 일괄 처리(Flush) 되는지 시뮬레이션 통합 테스트

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 단순 터치 필터링
- Given: 옵저버가 실행 중이고, 파일 속성만 변경되어 OS 이벤트가 발생함
- When: 디바운서 및 필터 검사가 동작함
- Then: 내용 수정 시간이 이전과 동일하므로 이벤트를 버리고(Skip) 큐에 적재하지 않는다.

## :gear: Technical & Non-Functional Constraints
- 안정성: 무한 루프 이벤트 스트림에 의한 데몬 CPU 과점유 방지

## :construction: Dependencies & Blockers
- Depends on: WA-CMD-01
- Blocks: WA-CMD-03, WA-QRY-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] WA-CMD-03: 내용이 수정된 파일 재분석 및 위키 부분 재생성 후 DB 갱신"
labels: 'feature, backend, priority:high, llm'
assignees: ''
---

## :dart: Summary
- 기능명: [WA-CMD-03] Watcher 기반 증분 분석 (Command)
- 목적: 큐에 적재된 수정된 파일들에 대해 해당 파일만 재파싱하고 관련된 1-Depth 폴더의 위키 마크다운을 부분적으로 갱신(Delta Update)한다.

## :link: References (Spec & Context)
- SRS 문서: `REQ-FUNC-025`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 대상 파일의 기존 벡터(ChromaDB) 삭제 및 새 청크/임베딩 Insert (ANA-CMD-02 로직 재사용)
- [ ] 해당 파일이 속한 1-Depth 폴더 단위의 `Wiki_Content` 재생성 트리거 (ANA-CMD-03 로직 재사용)
- [ ] 재생성 성공 후 DB 갱신 및 프론트엔드로 알림 브로드캐스트

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 부분 재분석 트리거
- Given: `01_Backend` 폴더 내의 한 파일이 수정되어 큐에 들어옴
- When: 큐 프로세서가 작동함
- Then: `01_Backend` 영역의 위키 마크다운 문서만 재생성되어 DB에 업데이트된다.

## :construction: Dependencies & Blockers
- Depends on: WA-CMD-02, ANA-CMD-03
- Blocks: STAT-CMD-01, WA-FE-02

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] WA-QRY-01 / WA-FE-01/02: Watcher 상태 반환 및 프론트엔드 UI/Toast 알림 연동"
labels: 'feature, fullstack, priority:medium'
assignees: ''
---

## :dart: Summary
- 목적: 사용자가 데몬의 동작 모드 및 현재 대기 중인 큐 상태를 UI에서 확인할 수 있도록 API를 제공하고, 갱신 성공 시 시각적 알림을 구현한다.

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] (백엔드) 현재 구동 모드(수동/실시간/유휴)와 메모리 상의 대기 큐 크기(N건)를 직렬화하여 반환하는 Query 구현
- [ ] (프론트엔드) 헤더 또는 설정 창에 Watcher 상태 콤보박스 및 상태등(초록/회색) 렌더링
- [ ] (프론트엔드) 백그라운드 위키 갱신 성공 브로드캐스트(웹소켓 또는 폴링 응답) 수신 시 "위키가 최신화되었습니다" IPC Toast 알림 노출

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 실시간 갱신 알림 렌더링
- Given: 앱이 켜져 있고 실시간 동기화 모드가 켜져 있음
- When: 백그라운드에서 파일이 갱신되어 DB가 업데이트 완료됨
- Then: 프론트엔드 우측 하단에 성공 Toast 알림 UI가 팝업된다.

## :construction: Dependencies & Blockers
- Depends on: WA-CMD-02, API-003
