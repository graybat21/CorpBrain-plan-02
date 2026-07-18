# CorpBrain MVP — 개발 태스크 리스트 (Merged Final)

> **기준 문서:** [SRS-draft_v0.6_OPUS.md](file:///c:/Users/docto/OneDrive/문서/CorpBrain-project-root/CorpBrain-plan-02-1/04_SRS-Drafts/피벗%20버전/SRS-draft_v0.6_OPUS.md)
> **생성일:** 2026-07-16
> **버전:** v2.0 (Gemini + Opus 병합 최종판)

---

## 병합 원칙

| # | 원칙 | 채택 출처 |
|---|------|----------|
| 1 | **도메인 접두사 ID 체계** (`WS-BE-01`, `LLM-FE-02`) — ID만 보고 도메인+레이어 즉시 파악 | Gemini |
| 2 | **Epic 내 BE/FE/UI 물리적 분리** — 스프린트 할당에 적합한 계층 구조 | Opus |
| 3 | **전량 커버리지** — REQ-FUNC 30건 + REQ-NF 17건 = 47건 누락 0 | Opus |
| 4 | **Foundation(Epic 0) / App Shell(Epic 9) / 비기능(Epic 8)** 독립 Epic | Opus |
| 5 | **복잡도** — 두 버전 중 더 현실적인 평가를 채택 (근거 명시) | 선택적 |

---

## 범례

| 구분 | 접미사 | 설명 |
|------|--------|------|
| **BE** | `-BE-` | 백엔드 (Python Core) |
| **FE** | `-FE-` | 프론트엔드 (React Desktop UI) |
| **UI** | `-UI-` | UI/UX 디자인 (와이어프레임, 시각 설계) |
| **INFRA** | `INFRA-` | 인프라 / 패키징 / DB 설계 |
| **NF** | `NF-` | 비기능 요구사항 전담 (성능, 보안, 신뢰성) |
| **APP** | `APP-` | 앱 공통 셸 (레이아웃, 디자인 시스템) |

**복잡도 기준:** H(High)=5일+, M(Medium)=2~4일, L(Low)=1일 이하 (1인 기준)

---

## Epic 0: 프로젝트 기반 (Foundation)

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| INFRA-001 | 프로젝트 구조 초기화 (React + Python 모노레포, IPC 통신 스캐폴딩) | §3.2, §3.6 | None | M |
| INFRA-002 | SQLite 스키마 생성 (`corpbrain_meta.db` — 7개 엔티티) | §6.2 전체 | INFRA-001 | M |
| INFRA-003 | ChromaDB/FAISS 벡터 DB 초기 설정 및 VectorDBManager 구현 | §6.2, §6.4 | INFRA-001 | M |
| INFRA-004 | DatabaseManager 공통 모듈 (connection pool, transaction, execute) | §6.4 | INFRA-002 | M |
| INFRA-005 | 로컬 REST API 서버 셋업 (Python — FastAPI/Flask, IPC 바인딩) | §3.3 API Overview | INFRA-001 | M |
| INFRA-006 | PyInstaller 단일 `.exe` 패키징 파이프라인 구축 | §1.2 S-01, CON-02 | INFRA-001 | H |
| INFRA-007 | 앱 데이터 경로 격리 (`%LocalAppData%\CorpBrain`) | §4.2 REQ-NF-004 | INFRA-001 | L |
| INFRA-008 | 로그 시스템 구축 (로테이션: 50MB / 30일) | §4.2 REQ-NF-014 | INFRA-001 | L |
| INFRA-009 | 예외 처리 프레임워크 (MAX_PATH 260자, 권한 거부 → Skip & Log) | §4.2 REQ-NF-007, CON-04 | INFRA-008 | M |

---

## Epic 1: 워크스페이스 & 대시보드 (F1)

### 백엔드

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| WS-BE-01 | WorkspaceManager — 워크스페이스 CRUD (생성/조회/삭제) | §4.1.1 REQ-FUNC-001~002, §6.1 #1~4 | INFRA-004, INFRA-005 | M |
| WS-BE-02 | FileScanner — 파일 트리 순회 및 File_Meta 벌크 삽입 | §4.1.1 REQ-FUNC-003, §6.4 | WS-BE-01 | M |
| WS-BE-03 | FileScanner — 블랙리스트 폴더 필터링 (.git, node_modules 등) | §4.1.1 REQ-FUNC-005 | WS-BE-02 | L |
| WS-BE-04 | FileScanner — 10,000개 파일 상한 일시정지 (Limit Guard) | §4.1.1 REQ-FUNC-004, §4.2 REQ-NF-012 | WS-BE-02 | L |
| WS-BE-05 | 대시보드 통계 산출 (파일 수, 용량, 예상 소요 시간) | §4.1.1 REQ-FUNC-003, §6.1 #5 | WS-BE-02 | L |
| WS-BE-06 | TextParser — `.docx`, `.pdf`, `.txt`, `.md` 4포맷 파서 구현 | §4.1.1 REQ-FUNC-006, §6.4 TextParser | INFRA-001 | H |
| WS-BE-07 | 워크스페이스 영속성 (앱 재시작 후 히스토리 로드) | §4.1.1 REQ-FUNC-002, §4.2 REQ-NF-008 | WS-BE-01 | L |

### 프론트엔드

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| WS-FE-01 | 좌측 히스토리 패널 (워크스페이스 목록, 생성/삭제 모달) | §4.1.1 REQ-FUNC-001~002, §3.6 | WS-BE-01, WS-UI-01 | M |
| WS-FE-02 | 다중 폴더 선택 다이얼로그 (OS 네이티브 폴더 선택) | §4.1.1 REQ-FUNC-001 | WS-FE-01 | L |
| WS-FE-03 | 오버뷰 대시보드 (파일 수, 용량, 예상 소요 시간 시각화) | §4.1.1 REQ-FUNC-003 | WS-BE-05, WS-UI-02 | M |
| WS-FE-04 | 10K 파일 초과 확인 다이얼로그 | §4.1.1 REQ-FUNC-004 | WS-BE-04 | L |

### UI/UX 디자인

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| WS-UI-01 | 좌측 히스토리 패널 와이어프레임 & 비주얼 디자인 | §3.6 Presentation Layer | APP-UI-01 | M |
| WS-UI-02 | 오버뷰 대시보드 레이아웃 및 차트 디자인 | §4.1.1 REQ-FUNC-003 | WS-UI-01 | M |

---

## Epic 2: 하이브리드 LLM 엔진 (F2)

### 백엔드

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| LLM-BE-01 | LLMRouter — Option A/B 라우팅 디스패처 공통 인터페이스 | §4.1.2 REQ-FUNC-007, §6.4 LLMRouter | INFRA-005 | H |
| LLM-BE-02 | Option A: Cloud LLM API 클라이언트 (OpenAI/Anthropic 통합) | §4.1.2 REQ-FUNC-007, §3.1 | LLM-BE-01 | M |
| LLM-BE-03 | Option B: Ollama 로컬 LLM 클라이언트 및 데몬 연동 | §4.1.2 REQ-FUNC-007, §3.1 | LLM-BE-01 | M |
| LLM-BE-04 | PIIFilter — 정규식 기반 PII 탐지 (이메일, 전화번호, 주민번호) | §4.1.2 REQ-FUNC-008, §6.3.3 1단계 | INFRA-001 | M |
| LLM-BE-05 | PIIFilter — NER 기반 고유명사 탐지 (인명, 기관명) | §4.1.2 REQ-FUNC-008, §6.3.3 2단계 | LLM-BE-04 | H |
| LLM-BE-06 | PIIFilter — 무결성 검증 및 Fail-Safe (전송 차단) | §4.1.2 REQ-FUNC-009, §6.3.3 3~4단계 | LLM-BE-04, LLM-BE-05 | M |
| LLM-BE-07 | Ollama 원클릭 온보딩 (Silent Install + 모델 Pull + 진행률) | §4.1.2 REQ-FUNC-010, §6.3.2 | None | H |
| LLM-BE-08 | LLM Health Check (API 키 유효성, Ollama 데몬 응답) | §4.1.2 REQ-FUNC-011, §6.1 #11 | LLM-BE-01 | L |

### 프론트엔드

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| LLM-FE-01 | LLM 설정 화면 (Option A/B 전환, API 키 입력 콤보박스) | §4.1.2 REQ-FUNC-007, §3.5 UC-05 | LLM-BE-01, LLM-UI-01 | M |
| LLM-FE-02 | Ollama 온보딩 프로그레스 바 UI (설치 + 모델 다운로드) | §4.1.2 REQ-FUNC-010 | LLM-BE-07, LLM-UI-01 | M |
| LLM-FE-03 | LLM 상태 인디케이터 (✅/❌ 헬스체크 아이콘) | §4.1.2 REQ-FUNC-011 | LLM-BE-08 | L |

### UI/UX 디자인

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| LLM-UI-01 | LLM 설정 화면 및 온보딩 플로우 UI 디자인 | §4.1.2, §6.3.2 | APP-UI-02 | M |

---

## Epic 3: 다단계 시맨틱 분석 파이프라인 (F3)

### 백엔드

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| ANA-BE-01 | AnalysisEngine — 고속 분석 (파일명/경로 기반 중요도 점수화) | §4.1.3 REQ-FUNC-012, §3.4.3, §6.1 #6 | WS-BE-02, LLM-BE-01 | H |
| ANA-BE-02 | AnalysisEngine — 심층 분석: 텍스트 청킹(Chunking) 엔진 | §4.1.3 REQ-FUNC-013, §6.4 TextParser.chunk | WS-BE-06 | M |
| ANA-BE-03 | AnalysisEngine — 심층 분석: 벡터 임베딩 저장 (ChromaDB/FAISS) | §4.1.3 REQ-FUNC-013, §3.4.1 | ANA-BE-02, INFRA-003 | M |
| ANA-BE-04 | AnalysisEngine — 심층 분석: LLM 기반 위키 마크다운 생성 | §4.1.3 REQ-FUNC-013, §3.4.1 | ANA-BE-03, LLM-BE-01 | H |
| ANA-BE-05 | 1-Depth 폴더별 탭 분리 위키 구조화 | §4.1.3 REQ-FUNC-014 | ANA-BE-04 | M |
| ANA-BE-06 | 분석 진행 상태 추적 API (task_id, 진행률, ETA) | §4.1.3 REQ-FUNC-015, §6.1 #8 | ANA-BE-01, ANA-BE-04 | M |

### 프론트엔드

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| ANA-FE-01 | 고속 분석 결과 화면 (중요도 순 파일 리스트, 상단 하이라이트) | §4.1.3 REQ-FUNC-012 | ANA-BE-01, ANA-UI-01 | M |
| ANA-FE-02 | 심층 분석 위키 뷰어 (마크다운 렌더링, 폴더별 독립 탭) | §4.1.3 REQ-FUNC-013~014 | ANA-BE-05, ANA-UI-01 | H |
| ANA-FE-03 | 분석 프로그레스 바 (처리 N/M, 잔여 시간 폴링) | §4.1.3 REQ-FUNC-015 | ANA-BE-06, ANA-UI-01 | M |

### UI/UX 디자인

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| ANA-UI-01 | 고속 분석 리스트 + 위키 뷰어 + 폴더 탭 레이아웃 디자인 | §4.1.3 | APP-UI-02 | H |

---

## Epic 4: 일괄 폴더/파일명 개편 (F4)

### 백엔드

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| RN-BE-01 | RenameManager — AI Naming 템플릿 추천 (LLM 프롬프트 엔지니어링 포함) | §4.1.4 REQ-FUNC-016, §6.4 | ANA-BE-04, LLM-BE-01 | H |
| RN-BE-02 | RenameManager — Diff 생성 (old/new 매핑 리스트) | §4.1.4 REQ-FUNC-017, §6.3.1 | RN-BE-01 | L |
| RN-BE-03 | RenameManager — OS 레벨 일괄 Rename 실행 및 Rename_History 기록 | §4.1.4 REQ-FUNC-018, §6.1 #12 | RN-BE-02 | M |
| RN-BE-04 | RenameManager — Rename Undo (100% 원복, Rename_History 참조) | §4.1.4 REQ-FUNC-019, §4.2 REQ-NF-009, §6.1 #13 | RN-BE-03 | M |

### 프론트엔드

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| RN-FE-01 | Rename Diff 미리보기 테이블 (빨강/초록 비교, 개별/전체 승인) | §4.1.4 REQ-FUNC-017 | RN-BE-02, RN-UI-01 | M |
| RN-FE-02 | 일괄 적용(Apply) + 실행 취소(Undo) IPC 연동 및 실패 팝업 | §4.1.4 REQ-FUNC-018~019 | RN-FE-01, RN-BE-03 | M |

### UI/UX 디자인

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| RN-UI-01 | Naming 템플릿 선택 + Diff 미리보기 + 결과 화면 디자인 | §4.1.4, §6.3.1 | APP-UI-02 | M |

---

## Epic 5: 로컬 딥링크 Trust-Anchor (F5)

### 백엔드

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| DL-BE-01 | DeepLinkBridge — 딥링크 매핑 생성 (위키 단락별 → 원문 파일 경로 자동 매핑) | §4.1.5 REQ-FUNC-020, §6.4 | ANA-BE-04 | H |
| DL-BE-02 | DeepLinkBridge — `os.startfile` 호출로 원본 파일 열기 (IPC 브릿지) | §4.1.5 REQ-FUNC-021, §6.4 | None | L |
| DL-BE-03 | DeepLinkBridge — 깨진 링크 탐지 (파일 존재 여부 확인) | §4.1.5 REQ-FUNC-022, §6.4 | DL-BE-01 | L |

### 프론트엔드

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| DL-FE-01 | 위키 내 딥링크 아이콘 렌더링 (문장별 출처 표시) | §4.1.5 REQ-FUNC-020 | DL-BE-01, DL-UI-01, ANA-FE-02 | M |
| DL-FE-02 | 딥링크 클릭 핸들러 (브라우저 기본 동작 차단 + IPC 파일 열기) | §4.1.5 REQ-FUNC-021 | DL-FE-01, DL-BE-02 | L |
| DL-FE-03 | 깨진 링크 시각적 표시 (회색 비활성화 + "원본 파일을 찾을 수 없습니다" 툴팁) | §4.1.5 REQ-FUNC-022 | DL-BE-03, DL-FE-01 | L |

### UI/UX 디자인

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| DL-UI-01 | 딥링크 아이콘 및 파일 유실 상태별 툴팁 UI 디자인 | §4.1.5 | APP-UI-02 | L |

---

## Epic 6: 실시간 Watcher (F6)

### 백엔드

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| WA-BE-01 | WatcherDaemon — Watcher 모드 설정 CRUD (수동/실시간/유휴/끄기) | §4.1.6 REQ-FUNC-023, §6.1 #14 | INFRA-004 | L |
| WA-BE-02 | WatcherDaemon — `watchdog.Observer` 기반 실시간 파일 이벤트 감지 데몬 | §4.1.6 REQ-FUNC-024, §3.4.2 | WA-BE-01, WS-BE-01 | H |
| WA-BE-03 | WatcherDaemon — 이벤트 디바운싱 및 `last_modified` 타임스탬프 대조 | §4.1.6 REQ-FUNC-024~025, §3.4.2 | WA-BE-02 | M |
| WA-BE-04 | WatcherDaemon — 변경분 재파싱 + 청크 무효화 + 위키 부분 갱신(Merge) | §4.1.6 REQ-FUNC-025, §3.4.2 | WA-BE-03, ANA-BE-04 | H |
| WA-BE-05 | WatcherDaemon — 유휴시간(Idle) 모드 (키보드/마우스 감지 + 누적 일괄 처리) | §4.1.6 REQ-FUNC-026, §6.3.5 | WA-BE-02 | H |
| WA-BE-06 | WatcherDaemon — 유휴 시 리소스 최소화 (CPU <1%, RAM <100MB) | §4.2 REQ-NF-002, CON-05 | WA-BE-02 | M |

### 프론트엔드

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| WA-FE-01 | Watcher 모드 설정 UI (4개 모드 선택 드롭다운/토글) | §4.1.6 REQ-FUNC-023 | WA-BE-01, WA-UI-01 | L |
| WA-FE-02 | Watcher 상태 표시 (is_running, 큐 이벤트 수, last_event) | §4.1.6 REQ-FUNC-024, §6.1 #15 | WA-BE-02 | L |
| WA-FE-03 | 위키 자동 갱신 Toast 알림 + 해당 탭 위키 리렌더링 | §4.1.6 REQ-FUNC-025 | WA-BE-04, WA-UI-01 | M |

### UI/UX 디자인

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| WA-UI-01 | Watcher 설정 + 상태 표시 + Toast 알림 UI 디자인 | §4.1.6 | APP-UI-02 | L |

---

## Epic 7: My Analytics (F7)

### 백엔드

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| STAT-BE-01 | AnalyticsService — 절약된 시간 산출 (토큰 → WPM 환산) | §4.1.7 REQ-FUNC-027, §6.3.4 | INFRA-004 | M |
| STAT-BE-02 | AnalyticsService — 팩트체크율 추적 (딥링크 클릭 누적 카운트) | §4.1.7 REQ-FUNC-028, §6.3.4 | DL-BE-02 | L |
| STAT-BE-03 | AnalyticsService — 지식 압축률 시각화 데이터 산출 | §4.1.7 REQ-FUNC-029, §6.3.4 | ANA-BE-04 | L |
| STAT-BE-04 | AnalyticsService — 자동화 기여도 수치화 (Watcher 자동 갱신 횟수) | §4.1.7 REQ-FUNC-030, §6.3.4 | WA-BE-04 | L |
| STAT-BE-05 | Analytics 집계 API (period 기반 필터링: week/month/all) | §6.1 #16 | STAT-BE-01~04 | M |

### 프론트엔드

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| STAT-FE-01 | My Analytics 대시보드 (4개 지표 카드 + 차트 + 텍스트 렌더링) | §4.1.7 REQ-FUNC-027~030 | STAT-BE-05, STAT-UI-01 | H |

### UI/UX 디자인

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| STAT-UI-01 | My Analytics 4대 지표 패널 레이아웃 및 차트 UI 디자인 | §4.1.7, §6.3.4 | APP-UI-02 | M |

---

## Epic 8: 보안 & 비기능 (Cross-Cutting)

| Task ID | Category | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|----------|-----------------|---------------|------------|--------|
| NF-001 | Security | Telemetry 차단 코드 레벨 검증 (외부 무단 통신 원천 배제) | §4.2 REQ-NF-005, CON-03 | INFRA-001 | M |
| NF-002 | Security | PII 마스킹 소켓 연결 전 메모리 상 100% 완료 보장 | §4.2 REQ-NF-006 | LLM-BE-06 | M |
| NF-003 | Performance | 파일 스캔 p95 <5,000ms (1,000개 기준) 최적화 | §4.2 REQ-NF-001 | WS-BE-02 | M |
| NF-004 | Performance | 심층 분석 100파일(평균 5KB) 300초 이내 처리량 보장 | §4.2 REQ-NF-003 | ANA-BE-04 | M |
| NF-005 | Reliability | Graceful Degradation (LLM 미연결 시 기존 위키·딥링크·대시보드 정상 동작) | §4.2 REQ-NF-010 | LLM-BE-01, ANA-FE-02 | M |
| NF-006 | Reliability | RPO/RTO 보장 (비정상 종료 후 30초 이내 복구, 마지막 DB 커밋 보존) | §4.2 REQ-NF-011 | INFRA-004, WS-BE-07 | M |
| NF-007 | Scalability | 10,000개 파일 전체 파이프라인 (스캔→분석→위키) 동작 검증 | §4.2 REQ-NF-012 | WS-BE-02, ANA-BE-04 | H |
| NF-008 | Scalability | 50개+ 워크스페이스 히스토리 보존 및 전환 <2초 | §4.2 REQ-NF-013 | WS-BE-01 | L |
| NF-009 | Maintainability | 사용자 설정(LLM, Watcher) Export/Import (JSON/TOML) | §4.2 REQ-NF-015 | INFRA-004 | L |
| NF-010 | Cost | Option A 파일당 평균 API 비용 산출 및 누적 비용 표시 | §4.2 REQ-NF-016 | LLM-BE-02, STAT-BE-05 | M |
| NF-011 | Monitoring | 내부 Health Metrics 로그 (분석 성공/실패, 평균 소요, Watcher 건수) | §4.2 REQ-NF-017 | INFRA-008 | L |

---

## Epic 9: 앱 공통 UI 셸 (App Shell)

| Task ID | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 | 복잡도 |
|---------|-----------------|---------------|------------|--------|
| APP-UI-01 | 전체 앱 레이아웃 디자인 (좌측 패널 + 메인 콘텐츠 영역) | §3.6 Presentation Layer | None | M |
| APP-UI-02 | 디자인 시스템 수립 (컬러, 타이포그래피, 아이콘, 컴포넌트 규격) | §3.6 | None | H |
| APP-FE-01 | React 앱 셸 구현 (라우팅, 레이아웃 프레임, 사이드바) | §3.6 | APP-UI-01, APP-UI-02, INFRA-001 | M |
| APP-FE-02 | 공통 UI 컴포넌트 라이브러리 (버튼, 입력, 모달, 토스트, 프로그레스 바) | §3.6 | APP-UI-02 | M |

---

## 전체 요약

| 구분 | 태스크 수 |
|------|----------|
| INFRA (인프라/기반) | 9 |
| BE (백엔드) | 39 |
| FE (프론트엔드) | 19 |
| UI/UX (디자인) | 8 |
| NF (비기능) | 11 |
| APP (앱 셸) | 4 |
| **합계** | **90** |

---

## 복잡도 분포 & 공수 추정

| 복잡도 | 태스크 수 | 추정 공수 (1인 기준) |
|--------|----------|---------------------|
| **H** (5일+) | 16 | 80~96일 |
| **M** (2~4일) | 47 | 94~188일 |
| **L** (1일 이하) | 27 | 13~27일 |
| **합계** | **90** | **187~311일 (약 9~15개월, 1인 기준)** |

> 2인 풀타임 기준 약 **5~8개월**, 3인 기준 약 **3~5개월** 예상.

---

## SRS Traceability 역추적 (누락 검증)

### 기능 요구사항 (REQ-FUNC)

| SRS ID | 매핑된 Task ID | ✓ |
|--------|---------------|---|
| REQ-FUNC-001 | WS-BE-01, WS-FE-01, WS-FE-02 | ✅ |
| REQ-FUNC-002 | WS-BE-01, WS-BE-07, WS-FE-01 | ✅ |
| REQ-FUNC-003 | WS-BE-02, WS-BE-05, WS-FE-03 | ✅ |
| REQ-FUNC-004 | WS-BE-04, WS-FE-04 | ✅ |
| REQ-FUNC-005 | WS-BE-03 | ✅ |
| REQ-FUNC-006 | WS-BE-06 | ✅ |
| REQ-FUNC-007 | LLM-BE-01, LLM-BE-02, LLM-BE-03, LLM-FE-01 | ✅ |
| REQ-FUNC-008 | LLM-BE-04, LLM-BE-05 | ✅ |
| REQ-FUNC-009 | LLM-BE-06 | ✅ |
| REQ-FUNC-010 | LLM-BE-07, LLM-FE-02 | ✅ |
| REQ-FUNC-011 | LLM-BE-08, LLM-FE-03 | ✅ |
| REQ-FUNC-012 | ANA-BE-01, ANA-FE-01 | ✅ |
| REQ-FUNC-013 | ANA-BE-02, ANA-BE-03, ANA-BE-04, ANA-FE-02 | ✅ |
| REQ-FUNC-014 | ANA-BE-05, ANA-FE-02 | ✅ |
| REQ-FUNC-015 | ANA-BE-06, ANA-FE-03 | ✅ |
| REQ-FUNC-016 | RN-BE-01 | ✅ |
| REQ-FUNC-017 | RN-BE-02, RN-FE-01 | ✅ |
| REQ-FUNC-018 | RN-BE-03, RN-FE-02 | ✅ |
| REQ-FUNC-019 | RN-BE-04, RN-FE-02 | ✅ |
| REQ-FUNC-020 | DL-BE-01, DL-FE-01 | ✅ |
| REQ-FUNC-021 | DL-BE-02, DL-FE-02 | ✅ |
| REQ-FUNC-022 | DL-BE-03, DL-FE-03 | ✅ |
| REQ-FUNC-023 | WA-BE-01, WA-FE-01 | ✅ |
| REQ-FUNC-024 | WA-BE-02, WA-BE-03, WA-FE-02 | ✅ |
| REQ-FUNC-025 | WA-BE-04, WA-FE-03 | ✅ |
| REQ-FUNC-026 | WA-BE-05 | ✅ |
| REQ-FUNC-027 | STAT-BE-01, STAT-FE-01 | ✅ |
| REQ-FUNC-028 | STAT-BE-02, STAT-FE-01 | ✅ |
| REQ-FUNC-029 | STAT-BE-03, STAT-FE-01 | ✅ |
| REQ-FUNC-030 | STAT-BE-04, STAT-FE-01 | ✅ |

### 비기능 요구사항 (REQ-NF)

| SRS ID | 매핑된 Task ID | ✓ |
|--------|---------------|---|
| REQ-NF-001 | NF-003 | ✅ |
| REQ-NF-002 | WA-BE-06 | ✅ |
| REQ-NF-003 | NF-004 | ✅ |
| REQ-NF-004 | INFRA-007 | ✅ |
| REQ-NF-005 | NF-001 | ✅ |
| REQ-NF-006 | NF-002 | ✅ |
| REQ-NF-007 | INFRA-009 | ✅ |
| REQ-NF-008 | WS-BE-07 | ✅ |
| REQ-NF-009 | RN-BE-04 | ✅ |
| REQ-NF-010 | NF-005 | ✅ |
| REQ-NF-011 | NF-006 | ✅ |
| REQ-NF-012 | NF-007 | ✅ |
| REQ-NF-013 | NF-008 | ✅ |
| REQ-NF-014 | INFRA-008 | ✅ |
| REQ-NF-015 | NF-009 | ✅ |
| REQ-NF-016 | NF-010 | ✅ |
| REQ-NF-017 | NF-011 | ✅ |

> **검증 결과:** REQ-FUNC 30건 + REQ-NF 17건 = **총 47건 전량 매핑 완료. 누락 0건.**
