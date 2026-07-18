# GitHub Issue Specifications - Epic 5: 딥링크 (Trust-Anchor)

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] DL-CMD-01: 위키 문장과 `File_Meta` 간 매핑(Anchor) 식별자 DB Update"
labels: 'feature, backend, priority:medium'
assignees: ''
---

## :dart: Summary
- 기능명: [DL-CMD-01] 딥링크 식별자 매핑 (Command)
- 목적: LLM이 생성한 위키 마크다운 내부의 인용구(출처)와 실제 로컬 시스템 상의 `File_Meta`를 연결하는 고유 식별자(Anchor)를 삽입/저장한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: `REQ-FUNC-020`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 마크다운 내 특정 패턴(예: `[[file_id:123]]`) 파싱 로직 구현
- [ ] 추출된 식별자를 DB의 `File_Meta`와 조인하여 유효성 확인
- [ ] 식별자 매핑 정보를 DB에 최종 확정

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 유효한 딥링크 생성
- Given: `[[file_id:UUID_1]]` 태그가 포함된 위키 본문이 주어짐
- When: 딥링크 매핑 함수를 실행함
- Then: 정상적으로 파싱되어, 해당 파일 메타데이터와 관계(Relation)가 맺어진다.

## :construction: Dependencies & Blockers
- Depends on: ANA-CMD-03
- Blocks: DL-QRY-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] DL-CMD-02: IPC 기반 `os.startfile` 호출 로직 구현"
labels: 'feature, backend, priority:high, os'
assignees: ''
---

## :dart: Summary
- 기능명: [DL-CMD-02] 로컬 앱 열기 (Command)
- 목적: 프론트엔드에서 딥링크를 클릭했을 때 브라우저 샌드박스를 우회하여 IPC 통신을 통해 OS 기본 프로그램(워드, PDF 뷰어 등)으로 파일을 직접 연다.

## :link: References (Spec & Context)
- SRS 문서: `REQ-FUNC-021`
- DTO 명세: `API-003`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] IPC(또는 로컬 HTTP 서버) 커맨드 수신 라우터 구현
- [ ] 수신된 파일 ID 기반 물리 경로(Path) 조회
- [ ] Windows API(`os.startfile()` 또는 `subprocess.Popen`)를 호출하여 애플리케이션 실행

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 외부 문서 오픈
- Given: 바탕화면의 `test.docx` 경로가 매핑된 파일 ID가 주어짐
- When: IPC로 파일 열기 명령을 호출함
- Then: 백엔드에서 에러 없이 실행 함수가 호출되고 MS Word가 구동되어 파일이 열린다.

## :gear: Technical & Non-Functional Constraints
- 보안: 악의적인 경로(예: `cmd.exe`) 주입 차단을 위해, 사전에 스캔된 `File_Meta` 테이블 내의 화이트리스트 경로만 허용

## :construction: Dependencies & Blockers
- Depends on: API-003
- Blocks: DL-FE-02, STAT-CMD-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] DL-QRY-01 / DL-TEST-01: Broken Link 실시간 검증 및 단위 테스트"
labels: 'feature, test, backend, priority:medium'
assignees: ''
---

## :dart: Summary
- 목적: 위키에 표시될 파일이 현재 물리 디스크 상에서 지워졌거나 이동되었는지(Broken) 검증하고 결과를 반환한다.

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] `File_Meta`에 등록된 경로에 대해 `os.path.exists()` 런타임 체크 (Query)
- [ ] 유효한 딥링크는 `true`, 끊어진 링크는 `false`로 포함하여 응답
- [ ] (DL-TEST-01) 파일 이동/삭제 시뮬레이션 후 Broken Link 플래그 테스트 확인

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 끊어진 딥링크 감지
- Given: 위키가 생성된 후 사용자가 OS 상에서 파일을 삭제함
- When: 위키 문서 조회를 수행함 (Query)
- Then: 반환되는 DTO에 해당 딥링크가 `is_broken: true` 상태로 전달된다.

## :construction: Dependencies & Blockers
- Depends on: DL-CMD-01
- Blocks: DL-FE-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] DL-FE-01/02: 위키 뷰어 내 딥링크 배지 UI 및 IPC 클릭 차단 연동"
labels: 'feature, frontend, priority:high'
assignees: ''
---

## :dart: Summary
- 목적: 마크다운 렌더러에서 딥링크 식별자를 인터랙티브한 배지(Badge) 버튼으로 시각화하고, 클릭 시 IPC 호출을 연동한다.

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] (DL-FE-01) 마크다운 파서 커스텀 플러그인(Remark 등)을 구현하여 식별자를 Button 컴포넌트로 치환
- [ ] Broken 링크일 경우 회색 처리 및 툴팁(Tooltip: '파일을 찾을 수 없습니다') 구현
- [ ] (DL-FE-02) 클릭 이벤트 `e.preventDefault()` 적용으로 브라우저 기본 링크 이동을 차단하고 IPC/API 호출로 우회 전송

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 딥링크 클릭 차단
- Given: 정상적인 딥링크 배지가 렌더링됨
- When: 사용자가 배지를 마우스로 클릭함
- Then: 브라우저가 새 탭을 열지 않고 백엔드 API 단으로 요청(Command)을 보낸다.

## :construction: Dependencies & Blockers
- Depends on: DL-QRY-01, DL-CMD-02
