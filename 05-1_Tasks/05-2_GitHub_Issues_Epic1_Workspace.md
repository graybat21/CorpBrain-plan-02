# GitHub Issue Specifications - Epic 1: 워크스페이스 & 대시보드 (Workspace)

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] WS-CMD-01: 2개 이상 로컬 폴더 병합 및 `Workspace_Meta` DB 레코드 삽입"
labels: 'feature, backend, priority:high'
assignees: ''
---

## :dart: Summary
- 기능명: [WS-CMD-01] 로컬 폴더 병합 및 워크스페이스 생성 (Command)
- 목적: 사용자가 OS에서 선택한 한 개 이상의 로컬 폴더 경로를 입력받아 새로운 가상의 워크스페이스로 매핑하고 DB에 영속화한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: `REQ-FUNC-001`
- DTO 명세: `API-001`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 여러 폴더 경로를 입력받아 병합(Merge)하는 비즈니스 로직(Service) 구현
- [ ] OS 물리 경로의 존재 여부 및 접근 권한 검증 로직 추가
- [ ] 유효한 경로들을 `Workspace_Meta` DB 테이블에 Insert (트랜잭션 적용)
- [ ] 생성된 워크스페이스의 UUID 반환

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 다중 폴더 병합 생성
- Given: 존재하는 로컬 폴더 경로 A(`C:\docs`)와 B(`D:\assets`)가 주어짐
- When: 병합하여 워크스페이스 생성(Command)을 요청함
- Then: DB에 워크스페이스 메타 레코드가 생성되고, 폴더 A와 B가 하위 경로로 매핑되며 201 Created 응답이 반환된다.

Scenario 2: 유효하지 않은 경로 포함 시 생성 실패
- Given: 존재하지 않는 경로(`Z:\fake_path`)가 포함되어 주어짐
- When: 생성(Command)을 요청함
- Then: 404 Not Found 에러가 발생하며 워크스페이스 생성이 취소된다(DB Rollback).

## :gear: Technical & Non-Functional Constraints
- 성능: 트랜잭션 처리 속도 p95 ≤ 50ms
- 운영체제: Windows 환경의 `MAX_PATH` 제약을 고려한 경로 파싱

## :checkered_flag: Definition of Done (DoD)
- [ ] Acceptance Criteria 충족 및 단위 테스트 (WS-TEST-01) 연계 통과 여부 확인
- [ ] 트랜잭션(Rollback) 엣지 케이스 확인 완료

## :construction: Dependencies & Blockers
- Depends on: DB-001, API-001
- Blocks: WS-QRY-01, SCAN-CMD-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] WS-QRY-01: 전체 워크스페이스 목록 및 단일 상세 조회 로직"
labels: 'feature, backend, priority:medium'
assignees: ''
---

## :dart: Summary
- 기능명: [WS-QRY-01] 워크스페이스 조회 (Query)
- 목적: 프론트엔드 히스토리 패널 렌더링을 위해 전체 워크스페이스 목록을 불러오고, 특정 워크스페이스의 메타 정보를 단일 조회한다.

## :link: References (Spec & Context)
- SRS 문서: `REQ-FUNC-002`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] DB에서 최근 생성/수정일 기준 내림차순(DESC)으로 `Workspace_Meta` 목록 SELECT
- [ ] UUID를 기반으로 단일 워크스페이스 정보 SELECT
- [ ] 조회 데이터를 `WorkspaceListRes` DTO 형식으로 매핑하여 반환

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 빈 워크스페이스 목록 조회
- Given: 생성된 워크스페이스가 없음
- When: 전체 목록 조회를 요청함
- Then: 빈 배열 `[]` 과 함께 200 OK 응답이 반환된다.

## :gear: Technical & Non-Functional Constraints
- 성능: DB Index를 활용하여 리스트 조회 p95 ≤ 30ms 달성

## :checkered_flag: Definition of Done (DoD)
- [ ] 정렬 조건(수정일 최신순) 적용 및 테스트 통과
- [ ] API 반환값이 DTO 명세와 일치하는가?

## :construction: Dependencies & Blockers
- Depends on: WS-CMD-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] WS-TEST-01: 폴더 병합 단위 테스트 및 DB 영속성 통합 테스트"
labels: 'test, backend, priority:medium'
assignees: ''
---

## :dart: Summary
- 기능명: [WS-TEST-01] Workspace 테스트 스위트
- 목적: WS-CMD-01 및 WS-QRY-01에서 구현한 로직이 안정적으로 동작하고 앱 재시작 환경에서도 영속성이 유지됨을 검증한다.

## :link: References (Spec & Context)
- 관련 태스크: WS-CMD-01, WS-QRY-01

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 폴더 병합 비즈니스 로직(중복 폴더 제거 등) 단위 테스트 작성 (Jest / PyTest 등)
- [ ] 테스트용 DB 환경에서 데이터를 Insert한 뒤 애플리케이션(커넥션)을 재시작하고 다시 Select하는 영속성 통합 테스트 작성

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 앱 재시작 후 영속성 유지
- Given: 테스트 스크립트에서 워크스페이스 데이터를 생성함
- When: DB 커넥션을 닫고 다시 염(Simulate App Restart)
- Then: 생성했던 데이터가 그대로 존재하고 정상적으로 조회되어야 한다.

## :checkered_flag: Definition of Done (DoD)
- [ ] 단위/통합 테스트 코드 추가 및 CI 환경 통과

## :construction: Dependencies & Blockers
- Depends on: WS-CMD-01, WS-QRY-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] SCAN-CMD-01: 파일 트리 순회 및 블랙리스트 제외 후 `File_Meta` 벌크 Insert"
labels: 'feature, backend, priority:high'
assignees: ''
---

## :dart: Summary
- 기능명: [SCAN-CMD-01] 파일 트리 스캔 및 추출 (Command)
- 목적: 매핑된 워크스페이스의 실제 디렉토리를 순회하며 블랙리스트(`/node_modules`, `.git` 등)를 제외한 유효 파일(.md, .docx, .pdf, .txt)의 메타데이터를 추출해 DB에 벌크 저장한다.

## :link: References (Spec & Context)
- SRS 문서: `REQ-FUNC-003, 005`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] OS 파일 시스템 순회 (재귀적 탐색) 로직 구현
- [ ] 확장자 화이트리스트 및 디렉토리 블랙리스트 필터링 구현
- [ ] 파일 메타(경로, 크기, 수정일 등)를 추출하여 `File_Meta` 테이블에 Bulk Insert (청크 단위로 DB I/O 최적화)

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 블랙리스트 파일 필터링
- Given: `.git` 폴더와 `test.md`가 포함된 경로가 주어짐
- When: 파일 트리 스캔을 실행함
- Then: `.git` 안의 파일들은 완전히 무시되고 `test.md`의 메타데이터만 추출되어 반환/저장된다.

## :gear: Technical & Non-Functional Constraints
- 성능: 1,000개 파일 스캔 및 필터링 처리를 2초 이내에 완료 (비동기 IO 또는 스레드풀 활용 고려)

## :checkered_flag: Definition of Done (DoD)
- [ ] Bulk Insert를 통한 성능 최적화가 적용되었는가?

## :construction: Dependencies & Blockers
- Depends on: DB-001, WS-CMD-01
- Blocks: SCAN-CMD-02, SCAN-QRY-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] SCAN-CMD-02: 파일 수 10,000개 도달 시 순회 중단 및 Limit Guard 예외 반환"
labels: 'feature, backend, priority:medium'
assignees: ''
---

## :dart: Summary
- 기능명: [SCAN-CMD-02] 스캔 Limit Guard (Command)
- 목적: 워크스페이스 내 파일이 10,000개를 초과할 경우 시스템 과부하를 막기 위해 즉시 스캔을 중단하고 예외를 발생시킨다.

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 순회 과정 중 누적 파일 수를 카운팅하는 로직 추가
- [ ] Count가 10,000 도달 시 즉시 Break하고 400 Bad Request(Payload Too Large) 커스텀 예외 반환

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: Limit Guard 발동
- Given: 워크스페이스 내부에 10,005개의 유효 파일이 존재함
- When: 파일 스캔을 시작함
- Then: 10,000개를 만나는 순간 순회가 멈추고 파일 갯수 초과 에러 메시지가 리턴된다.

## :checkered_flag: Definition of Done (DoD)
- [ ] 10K Guard 테스트(SCAN-TEST-02) 작성 및 통과

## :construction: Dependencies & Blockers
- Depends on: SCAN-CMD-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] SCAN-QRY-01: 스캔된 파일 수, 용량(MB), 예상 소요시간 산출 후 반환"
labels: 'feature, backend, priority:low'
assignees: ''
---

## :dart: Summary
- 기능명: [SCAN-QRY-01] 스캔 요약 결과 제공 (Query)
- 목적: 파일 스캔 완료 후 총 갯수, 용량, 분석 예상 소요시간을 계산하여 프론트엔드 대시보드에 표시할 통계 DTO를 반환한다.

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] `File_Meta`에서 워크스페이스 단위로 파일 개수 및 용량(SUM) 집계 (Query)
- [ ] WPM(Words Per Minute) 또는 평균 파일 크기 공식을 활용하여 ETA(예상 소요시간) 산출
- [ ] `ScanSummaryRes` DTO 구조로 반환

## :construction: Dependencies & Blockers
- Depends on: SCAN-CMD-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] SCAN-TEST-01/02: 스캔 필터링 및 Limit Guard 단위 테스트"
labels: 'test, backend, priority:medium'
assignees: ''
---

## :dart: Summary
- 목적: 스캔 엔진의 필터링 규칙(블랙리스트)과 방어 기제(10K Limit)가 완벽히 동작하는지 검증한다.

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 가상 파일 시스템(Mock FS)을 이용해 블랙리스트 파일만 존재하는 폴더 스캔 시도 -> 결과 0건 검증
- [ ] 가상 파일 10,001개를 생성하고 스캔 시도 -> Limit Guard 에러 발생 여부 검증

## :construction: Dependencies & Blockers
- Depends on: SCAN-CMD-01, SCAN-CMD-02

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] WS-FE-01/02/03: Workspace & 대시보드 UI 구현"
labels: 'feature, frontend, priority:high'
assignees: ''
---

## :dart: Summary
- 기능명: Workspace Frontend UI
- 목적: 프론트엔드에서 좌측 패널, 생성 모달, 대시보드 스캔 통계를 사용자에게 시각적으로 제공한다.

## :link: References (Spec & Context)
- SRS 문서: `§3.6`, `REQ-FUNC-001~004`
- API 명세: `API-001, API-002`, `MOCK-001`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] (WS-FE-01) 좌측 사이드바(히스토리 패널) 렌더링 및 `GET /workspaces` (Mock) 연동
- [ ] (WS-FE-02) 신규 생성 버튼 클릭 시 OS 네이티브 디렉토리 선택기 호출 및 생성 API 연동
- [ ] (WS-FE-03) 대시보드 내 전체 통계 데이터 렌더링 및 Limit Guard(400 에러) 수신 시 "파일이 너무 많습니다" 경고 다이얼로그(Toast) 구현

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 10K 예외 모달 노출
- Given: 새 워크스페이스 추가 시 OS 폴더에 10,000개가 넘는 파일이 있음
- When: 추가 버튼을 클릭함
- Then: 서버에서 반환된 400 에러를 캐치하여 사용자에게 에러 다이얼로그를 띄우고 상태를 원복한다.

## :gear: Technical & Non-Functional Constraints
- 성능/UX: 비동기 API 호출 시 로딩 스피너(Skeleton UI) 노출 등 상태(State) 관리 철저

## :checkered_flag: Definition of Done (DoD)
- [ ] Mock API를 이용해 정상/에러 시나리오 렌더링 검증이 완료되었는가?
- [ ] 디자인 시스템(Color, Typography)이 일관성 있게 적용되었는가?

## :construction: Dependencies & Blockers
- Depends on: MOCK-001, APP-UI-01
