# GitHub Issue Specifications - Epic 0: 데이터 모델 & 통신 계약 (Data & Contract)

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] DB-001: SQLite `corpbrain_meta.db` 스키마 생성 및 마이그레이션"
labels: 'feature, backend, priority:high, database'
assignees: ''
---

## :dart: Summary
- 기능명: [DB-001] SQLite 스키마 생성 및 마이그레이션
- 목적: 애플리케이션의 메타데이터(Workspace, FileMeta 등)를 관리하기 위한 RDBMS 스키마를 정의하고 마이그레이션 환경을 구축한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: `/docs/SRS_v0.md#6.2`
- 데이터 모델 (ERD): `/docs/erd.md#Workspace_FileMeta`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 마이그레이션 도구(예: Alembic 또는 Prisma) 초기화
- [ ] DB 엔티티 모델 정의 (Workspace, FileMeta, Rename_History, Analytics 등 최소 7개 테이블)
- [ ] 초기 DB 마이그레이션 스크립트 작성 및 적용
- [ ] DB Connection Pool 및 세션 팩토리(Session Factory) 로직 구현
- [ ] SQLite WAL(Write-Ahead Logging) 모드 활성화 설정 적용

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 스키마 초기화 및 생성
- Given: 데이터베이스 파일이 존재하지 않는 빈 환경이 주어짐
- When: 마이그레이션 실행 명령어를 호출함
- Then: `corpbrain_meta.db` 물리 파일이 생성되고, 명세된 7개의 테이블이 누락 없이 생성된다.

Scenario 2: DB 커넥션 및 쿼리 동작 검증
- Given: 정상적으로 초기화된 데이터베이스가 주어짐
- When: Workspace 테이블에 임의의 더미 데이터를 INSERT 한 후 SELECT 함
- Then: 삽입한 데이터가 정확하게 조회되며, 에러 없이 커넥션이 종료된다.

## :gear: Technical & Non-Functional Constraints
- 성능: 쿼리 실행 시간 p95 ≤ 10ms (단순 CRUD 기준)
- 안정성: 멀티스레드 환경 대비 WAL 모드 필수 적용 및 커넥션 풀링 최적화
- 데이터 무결성: 모든 외래키(Foreign Key) 제약조건 활성화 (`PRAGMA foreign_keys = ON`)

## :checkered_flag: Definition of Done (DoD)
- [ ] 모든 Acceptance Criteria를 충족하는가?
- [ ] 테이블 스키마에 대한 인프라 단위 테스트(Unit Test)가 추가되었고 통과하는가?
- [ ] Linter 경고가 없는가?
- [ ] Entity 모델 문서가 최신화되었는가?

## :construction: Dependencies & Blockers
- Depends on: None
- Blocks: API-001, WS-CMD-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] DB-002: ChromaDB / FAISS 벡터 DB 컬렉션 초기화 스크립트 작성"
labels: 'feature, backend, priority:high, vector-db'
assignees: ''
---

## :dart: Summary
- 기능명: [DB-002] 벡터 DB 컬렉션 초기화
- 목적: 파일 파싱 결과물(텍스트 청크)과 임베딩 벡터를 저장하기 위한 로컬 벡터 DB(ChromaDB 또는 FAISS)의 컬렉션을 생성하고 초기화한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: `/docs/SRS_v0.md#6.2`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 벡터 DB 클라이언트 초기화 코드 작성 (임베딩 모델의 차원수 설정)
- [ ] 로컬 영속성(Persistence) 디렉토리 지정 및 컬렉션 생성 (예: `corpbrain_vectors`)
- [ ] 기존 컬렉션 존재 시 로드(Load), 없을 시 신규 생성 분기 처리 로직 작성
- [ ] 기초 CRUD(문서 추가/조회) 테스트 스크립트 작성

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 벡터 DB 컬렉션 생성
- Given: 지정된 벡터 DB 저장 경로가 비어있음
- When: 클라이언트 초기화 함수를 호출함
- Then: 스토리지 폴더에 벡터 DB 파일이 생성되고 지정된 이름의 컬렉션이 활성화된다.

## :gear: Technical & Non-Functional Constraints
- 보안/격리: 로컬 파일 시스템 내부에만 영속 저장되며 외부 클라우드 의존성이 없어야 함.

## :checkered_flag: Definition of Done (DoD)
- [ ] 모든 Acceptance Criteria를 충족하는가?
- [ ] 컬렉션 생성 확인 통합 테스트 통과?
- [ ] Linter 통과?

## :construction: Dependencies & Blockers
- Depends on: None
- Blocks: ANA-CMD-02

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] API-001: [API Spec] Workspace 도메인 Request/Response DTO 정의"
labels: 'feature, api, priority:high'
assignees: ''
---

## :dart: Summary
- 기능명: [API-001] Workspace API 스펙 및 DTO 정의
- 목적: 프론트엔드와 백엔드가 통신할 Workspace 도메인(생성/삭제/목록 조회 등)의 Request, Response DTO(데이터 전송 객체)를 정의하여 단일 진실 공급원(SSOT)을 확보한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: `/docs/SRS_v0.md#6.1`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] CQRS 원칙에 따른 `WorkspaceCreateReq`, `WorkspaceListRes` 등 DTO 클래스(또는 타입 인터페이스) 정의
- [ ] DTO 필드별 타입, 제약조건(Validation 규칙), 기본값 적용
- [ ] OpenAPI(Swagger) 또는 Type 정의 문서(TypeScript Interface 등) 생성 및 내보내기

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: DTO 유효성 검증 로직 작동
- Given: `WorkspaceCreateReq` DTO에 잘못된 폴더 경로(빈 문자열)가 주어짐
- When: Validator(검증 로직)를 실행함
- Then: 유효성 검사 실패 에러(400 Bad Request 형태)가 발생하며 적절한 메시지를 반환한다.

## :gear: Technical & Non-Functional Constraints
- 구조: DTO는 비즈니스 로직(Service) 및 데이터베이스 엔티티(DB Entity)와 완전히 분리되어야 함.

## :checkered_flag: Definition of Done (DoD)
- [ ] 모든 관련 API Endpoint의 Payload 구조가 정의되었는가?
- [ ] 단위 테스트(DTO Validator Test) 작성 및 통과?
- [ ] Swagger 혹은 타입 선언 파일이 생성되었는가?

## :construction: Dependencies & Blockers
- Depends on: DB-001
- Blocks: API-002, MOCK-001, WS-CMD-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] API-002: [API Spec] Analysis 도메인 Request/Response DTO 정의"
labels: 'feature, api, priority:high'
assignees: ''
---

## :dart: Summary
- 기능명: [API-002] Analysis API 스펙 및 DTO 정의
- 목적: 스캔, 파일 파싱, 분석 진행 상태(Progress) 반환과 관련된 Request/Response DTO를 정의한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: `/docs/SRS_v0.md#6.1`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] `ScanProgressRes`, `AnalysisStartReq`, `WikiMarkdownRes` 등 DTO 정의
- [ ] 각 DTO에 적절한 타입 힌팅 및 Validation 룰 부여
- [ ] API 스키마 문서화 추가(OpenAPI 연동)

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: Progress DTO 검증
- Given: 진행 상황 데이터(전체 100건 중 50건 처리, ETA 10초)가 주어짐
- When: `ScanProgressRes` DTO를 인스턴스화 및 직렬화(Serialize)함
- Then: 숫자 및 문자열 타입이 올바르게 JSON 포맷으로 변환되어 출력된다.

## :gear: Technical & Non-Functional Constraints
- 일관성: 모든 필드명 네이밍 컨벤션(CamelCase 등)을 Workspace API와 통일

## :checkered_flag: Definition of Done (DoD)
- [ ] 모든 Acceptance Criteria를 충족하는가?
- [ ] 명세서가 최신화되었는가?

## :construction: Dependencies & Blockers
- Depends on: API-001
- Blocks: API-003, MOCK-001, SCAN-CMD-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] API-003: [API Spec] LLM, Rename, Watcher, Analytics DTO 정의"
labels: 'feature, api, priority:high'
assignees: ''
---

## :dart: Summary
- 기능명: [API-003] 부가 기능 도메인 DTO 정의
- 목적: LLM 엔진 설정, 파일명 일괄 변경(Rename), 실시간 데몬(Watcher) 및 통계(Analytics) 처리를 위한 통신 규격을 정의한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: `/docs/SRS_v0.md#6.1`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] LLM 도메인 (`LlmOptionReq`, `LlmHealthCheckRes`) DTO 정의
- [ ] Rename 도메인 (`RenameDiffRes`, `RenameApplyReq`) DTO 정의
- [ ] Watcher 도메인 (`WatcherStatusRes`, `WatcherConfigReq`) DTO 정의
- [ ] Analytics 도메인 (`AnalyticsDashboardRes`) DTO 정의

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: LlmOptionReq DTO 검증
- Given: 지원되지 않는 엔진 타입(예: `unknown_engine`)이 주어짐
- When: `LlmOptionReq` DTO를 검증함
- Then: Enum Validation 실패 에러가 발생한다.

## :gear: Technical & Non-Functional Constraints
- 보안: LLM API Key 등 민감 정보가 Response DTO에 노출되지 않도록 처리

## :checkered_flag: Definition of Done (DoD)
- [ ] DTO 유효성 단위 테스트 작성 및 통과?
- [ ] Swagger 스펙 업데이트?

## :construction: Dependencies & Blockers
- Depends on: API-002
- Blocks: MOCK-002, LLM-CMD-01, RN-CMD-01, WA-CMD-01, STAT-FE-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] MOCK-001: [Mock] 프론트엔드 UI 독립 개발용 Workspace 및 대시보드 Mock 서버 세팅"
labels: 'feature, mock, priority:medium'
assignees: ''
---

## :dart: Summary
- 기능명: [MOCK-001] Workspace / 대시보드 Mock API 제공
- 목적: 백엔드 비즈니스 로직 완성 전, 프론트엔드가 UI(사이드바, 대시보드 화면) 개발을 병행할 수 있도록 정적 더미 데이터를 반환하는 Mock 서버를 세팅한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev 대기
- API 명세: API-001, API-002

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] Mock 데이터(JSON) 더미 팩토리 생성 (워크스페이스 리스트, 스캔 결과 등)
- [ ] HTTP Mock 서버(MSW, json-server, 또는 Python FastAPI 정적 엔드포인트) 구성
- [ ] 프론트엔드 연동 가이드 문서 작성

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: Mock Workspace 목록 조회
- Given: Mock 서버가 실행 중임
- When: 프론트엔드에서 `/api/v1/workspaces` 로 GET 요청을 보냄
- Then: 200 OK와 함께 미리 정의된 3개의 Workspace 더미 객체 리스트(ID, 경로 등 포함)를 반환한다.

## :gear: Technical & Non-Functional Constraints
- 환경 분리: 개발 환경(`NODE_ENV=development` 등)에서만 Mock 서버가 동작해야 하며 빌드 시 프로덕션 환경에서는 제외되어야 함.

## :checkered_flag: Definition of Done (DoD)
- [ ] 프론트엔드에서 Mock API 호출 시 정상 응답을 받는가?

## :construction: Dependencies & Blockers
- Depends on: API-001, API-002
- Blocks: WS-FE-01, WS-FE-02, WS-FE-03

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] MOCK-002: [Mock] 심층 분석 결과(폴더별 탭) 및 Rename Diff 반환 Mock 서버 세팅"
labels: 'feature, mock, priority:medium'
assignees: ''
---

## :dart: Summary
- 기능명: [MOCK-002] Analysis / Rename 도메인 Mock API 제공
- 목적: 분석 결과(위키 마크다운 내용) 및 파일명 변경 추천 결과(Diff) 화면을 개발하기 위한 Mock API 환경을 구축한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note
- API 명세: API-003

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 마크다운 더미 데이터가 포함된 WikiContent 응답 Mock 구현
- [ ] 구 파일명, 신규 파일명이 쌍으로 매핑된 Rename Diff Mock 구현
- [ ] 기존 Mock 서버 레지스트리에 새로운 엔드포인트 추가

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: Rename Diff Mock 호출
- Given: Mock 서버가 활성화되어 있음
- When: `/api/v1/rename/diff`로 GET 요청을 보냄
- Then: `old_name`과 `new_name`이 매핑된 더미 리스트와 함께 200 OK를 반환한다.

## :gear: Technical & Non-Functional Constraints
- 포맷: 마크다운 텍스트는 이스케이프 처리가 제대로 되어 프론트엔드 뷰어 컴포넌트에서 렌더링 가능해야 함.

## :checkered_flag: Definition of Done (DoD)
- [ ] FE 연동 시 마크다운 및 Diff 테이블이 정상 렌더링되는 더미 데이터를 뿜어내는가?

## :construction: Dependencies & Blockers
- Depends on: API-003, MOCK-001
- Blocks: ANA-FE-01, ANA-FE-02, RN-FE-01, RN-FE-02
