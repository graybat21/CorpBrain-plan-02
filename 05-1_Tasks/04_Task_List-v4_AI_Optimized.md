# CorpBrain MVP — 개발 태스크 리스트 (v4.0 AI-Optimized)

> **생성일:** 2026-07-16
> **버전:** v4.0 (AI 에이전트 오케스트레이션 최적화 버전)
> **최적화 기준:** 
> 1. Data/Contract 선행 추출 (프론트/백엔드 단일 진실 공급원)
> 2. CQRS 패턴 기반 닫힌 문맥(Read/Write) 격리
> 3. 인수 조건(AC)을 자동화 테스트(Unit/Integration Test) 코드로 변환

---

## 📌 Epic 0: 데이터 모델 & 통신 계약 (Data & Contract)
**목표:** AI 에이전트가 상상력으로 데이터 구조를 짜지 않도록, 백엔드와 프론트엔드의 기준점(SSOT)을 가장 먼저 확립합니다.

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 (H/M/L) |
|---------|--------------|----------------|---------------|--------------------------|---------------|
| DB-001 | Data & Contract | SQLite `corpbrain_meta.db` 스키마 생성 및 마이그레이션 (Workspace, FileMeta 등 7개 테이블) | §6.2 | None | M |
| DB-002 | Data & Contract | ChromaDB / FAISS 벡터 DB 컬렉션 초기화 스크립트 작성 | §6.2 | None | L |
| API-001 | Data & Contract | [API Spec] Workspace 도메인 (생성/삭제/조회) Request/Response DTO 정의 | §6.1 | DB-001 | L |
| API-002 | Data & Contract | [API Spec] Analysis 도메인 (스캔/분석/프로그레스) Request/Response DTO 정의 | §6.1 | API-001 | L |
| API-003 | Data & Contract | [API Spec] LLM, Rename, Watcher, Analytics Request/Response DTO 정의 | §6.1 | API-002 | M |
| MOCK-001| Data & Contract | [Mock] 프론트엔드 UI 독립 개발용 Workspace 및 대시보드 스캔 반환 Mock 서버 세팅 | §6.1 | API-001, API-002 | M |
| MOCK-002| Data & Contract | [Mock] 심층 분석 결과(폴더별 탭) 및 Rename Diff 반환 Mock 서버 세팅 | §6.1 | API-003 | M |

---

## 📌 Epic 1: 워크스페이스 & 대시보드 (Workspace)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 (H/M/L) |
|---------|--------------|----------------|---------------|--------------------------|---------------|
| WS-CMD-01| Workspace | [Command] 2개 이상 로컬 폴더 병합 및 `Workspace_Meta` DB 레코드 삽입 (Insert) | REQ-FUNC-001 | DB-001, API-001 | L |
| WS-QRY-01| Workspace | [Query] 전체 워크스페이스 목록 및 단일 상세 조회 로직 | REQ-FUNC-002 | WS-CMD-01 | L |
| WS-TEST-01| Workspace | [Test] 폴더 병합 비즈니스 로직 단위 테스트 및 앱 재시작 후 영속성(Persistence) 통합 테스트 | REQ-FUNC-001, 002 | WS-CMD-01, WS-QRY-01 | M |
| SCAN-CMD-01| Workspace | [Command] 파일 트리 순회 및 블랙리스트(.git 등) 제외 후 `File_Meta` 벌크 Insert | REQ-FUNC-003, 005 | DB-001, API-002 | M |
| SCAN-CMD-02| Workspace | [Command] 파일 수 10,000개 도달 시 순회 중단 및 Limit Guard 예외 반환 (400) | REQ-FUNC-004 | SCAN-CMD-01 | L |
| SCAN-QRY-01| Workspace | [Query] 스캔된 파일 수, 용량(MB), 예상 소요시간 산출 후 DTO 반환 | REQ-FUNC-003 | SCAN-CMD-01 | L |
| SCAN-TEST-01| Workspace | [Test] 블랙리스트 폴더 및 미지원 포맷이 정확히 제외되는지 검증하는 단위 테스트 | REQ-FUNC-005 | SCAN-CMD-01 | M |
| SCAN-TEST-02| Workspace | [Test] 파일 10,000개 초과 시 Limit Guard 예외 처리(400 에러) 단위 테스트 | REQ-FUNC-004 | SCAN-CMD-02 | L |
| WS-FE-01 | Workspace | [UI] 좌측 히스토리 패널 렌더링 및 워크스페이스 목록 API(Query) 연동 | §3.6 | MOCK-001 | M |
| WS-FE-02 | Workspace | [UI] 신규 워크스페이스 생성 모달 (OS 네이티브 폴더 선택) 및 API(Command) 연동 | REQ-FUNC-001 | MOCK-001 | L |
| WS-FE-03 | Workspace | [UI] 대시보드 통계(Query) 바인딩 및 10K Guard 예외 수신 시 다이얼로그 표시 | REQ-FUNC-003, 004 | MOCK-001 | M |

---

## 📌 Epic 2: 하이브리드 LLM 엔진 (LLM Engine)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 (H/M/L) |
|---------|--------------|----------------|---------------|--------------------------|---------------|
| LLM-CMD-01| LLM Engine | [Command] 사용자의 Option A/B 엔진 설정 변경 및 DB 저장 | REQ-FUNC-007 | DB-001, API-003 | L |
| LLM-QRY-01| LLM Engine | [Query] 선택된 엔진(Cloud/Ollama) 연결 상태 확인 (Health Check) 반환 | REQ-FUNC-011 | API-003 | L |
| LLM-CMD-02| LLM Engine | [Command] Option A 전송 전 PII(이메일, 전화번호 등) 정규식/NER 마스킹 인메모리 적용 | REQ-FUNC-008 | API-003 | M |
| LLM-CMD-03| LLM Engine | [Command] Option B 선택 시 Ollama 데몬 무인 설치 및 모델 Pull 백그라운 호출 | REQ-FUNC-010 | API-003 | M |
| LLM-TEST-01| LLM Engine | [Test] PII 마스킹 정규식/NER 단위 테스트 및 마스킹 실패 시 예외(차단) 테스트 | REQ-FUNC-009 | LLM-CMD-02 | H |
| LLM-TEST-02| LLM Engine | [Test] Health Check 상태별(네트워크 단절, Ollama 미응답 등) 반환값 단위 테스트 | REQ-FUNC-011 | LLM-QRY-01 | M |
| LLM-FE-01 | LLM Engine | [UI] LLM 설정 화면 (Option A/B 콤보박스) 및 Health Check 상태(✅/❌) 표시 | REQ-FUNC-007, 011 | API-003 | M |
| LLM-FE-02 | LLM Engine | [UI] Ollama 설치 프로그레스 바(Command 연동) 렌더링 | REQ-FUNC-010 | API-003 | M |

---

## 📌 Epic 3: 분석 파이프라인 (Analysis)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 (H/M/L) |
|---------|--------------|----------------|---------------|--------------------------|---------------|
| ANA-CMD-01| Analysis | [Command] 폴더/파일명 추출 및 고속 분석 중요도 산출 후 DB 업데이트 | REQ-FUNC-012 | API-002, LLM-CMD-01 | M |
| ANA-CMD-02| Analysis | [Command] 문서(docx, pdf, txt, md) 파싱 후 텍스트 청킹(Chunking) 및 벡터 DB Insert | REQ-FUNC-013 | DB-002, API-002 | H |
| ANA-CMD-03| Analysis | [Command] 청크 기반 LLM 위키 마크다운 생성 및 `Wiki_Content` DB Insert | REQ-FUNC-013 | ANA-CMD-02, LLM-CMD-01 | H |
| ANA-QRY-01| Analysis | [Query] 1-Depth 폴더별로 분리 가공된 위키 마크다운 구조 반환 | REQ-FUNC-014 | ANA-CMD-03 | M |
| ANA-QRY-02| Analysis | [Query] 분석 진행 상태(Progress: 처리/전체, ETA) 산출 후 반환 | REQ-FUNC-015 | ANA-CMD-02, 03 | M |
| ANA-TEST-01| Analysis | [Test] 지원 4개 포맷 텍스트 추출 정확성(빈 문자열 검출 등) 단위 테스트 | REQ-FUNC-006 | ANA-CMD-02 | M |
| ANA-TEST-02| Analysis | [Test] 생성된 위키가 1-Depth 폴더 구분을 침범하지 않았는지 검증하는 단위 테스트 | REQ-FUNC-014 | ANA-CMD-03, ANA-QRY-01 | M |
| ANA-FE-01 | Analysis | [UI] 고속 분석 중요도 순 정렬 결과 리스트 렌더링 (Query 연동) | REQ-FUNC-012 | MOCK-002 | M |
| ANA-FE-02 | Analysis | [UI] 1-Depth 폴더 탭 컴포넌트 및 마크다운 뷰어 렌더링 (Query 연동) | REQ-FUNC-014 | MOCK-002 | H |
| ANA-FE-03 | Analysis | [UI] 분석 진행 상태 프로그레스 바 폴링(Polling) 업데이트 | REQ-FUNC-015 | API-002 | L |

---

## 📌 Epic 4: 파일명 일괄 변경 (Batch Rename)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 (H/M/L) |
|---------|--------------|----------------|---------------|--------------------------|---------------|
| RN-CMD-01 | Rename | [Command] LLM 템플릿 추천 호출 및 Diff 결과를 DB에 임시 저장 | REQ-FUNC-016 | DB-001, API-003 | M |
| RN-QRY-01 | Rename | [Query] 생성된 파일명 Diff (Old/New) 매핑 리스트 반환 | REQ-FUNC-017 | RN-CMD-01 | L |
| RN-CMD-02 | Rename | [Command] 승인된 Diff 기반 OS 레벨 물리 파일 Rename 및 `Rename_History` Insert | REQ-FUNC-018 | RN-QRY-01 | M |
| RN-CMD-03 | Rename | [Command] `Rename_History` 기록 기반 OS 파일명 100% 원복(Undo) 실행 | REQ-FUNC-019 | RN-CMD-02 | M |
| RN-TEST-01| Rename | [Test] Diff 생성 정확성 및 Undo 실행 시 파일 경로 100% 복구 무결성 통합 테스트 | REQ-FUNC-019 | RN-CMD-03 | H |
| RN-FE-01  | Rename | [UI] Diff 미리보기 테이블 (빨강/초록) 렌더링 (Query 연동) | REQ-FUNC-017 | MOCK-002 | M |
| RN-FE-02  | Rename | [UI] Apply 및 Undo 버튼 동작 및 실패 모달 렌더링 (Command 연동) | REQ-FUNC-018, 019 | MOCK-002 | M |

---

## 📌 Epic 5: 딥링크 (Trust-Anchor)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 (H/M/L) |
|---------|--------------|----------------|---------------|--------------------------|---------------|
| DL-CMD-01 | Deep-link | [Command] 생성된 위키 문장과 `File_Meta` 간 매핑(Anchor) 식별자 DB Update | REQ-FUNC-020 | ANA-CMD-03 | M |
| DL-CMD-02 | Deep-link | [Command] IPC 기반 `os.startfile` 호출 로직 구현 | REQ-FUNC-021 | API-003 | L |
| DL-QRY-01 | Deep-link | [Query] 위키 내 딥링크 대상 원본 파일의 현재 존재(Broken) 여부 검증 반환 | REQ-FUNC-022 | DL-CMD-01 | L |
| DL-TEST-01| Deep-link | [Test] OS에서 임의로 삭제/이동된 파일에 대해 Broken Link 플래그가 반환되는지 단위 테스트 | REQ-FUNC-022 | DL-QRY-01 | M |
| DL-FE-01  | Deep-link | [UI] 위키 뷰어 내 딥링크 배지 렌더링 및 깨진 링크 시 회색/툴팁 처리 | REQ-FUNC-020, 022| MOCK-002 | M |
| DL-FE-02  | Deep-link | [UI] 딥링크 onClick 시 브라우저 기본 동작 차단 및 IPC(Command) 호출 | REQ-FUNC-021 | API-003 | L |

---

## 📌 Epic 6: 실시간 데몬 (Watcher)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 (H/M/L) |
|---------|--------------|----------------|---------------|--------------------------|---------------|
| WA-CMD-01 | Watcher | [Command] Watcher 설정 모드(수동/실시간/유휴) 변경 및 DB 저장 | REQ-FUNC-023 | DB-001 | L |
| WA-CMD-02 | Watcher | [Command] `watchdog.Observer` 이벤트 감지, 디바운싱 및 타임스탬프 대조 로직 | REQ-FUNC-024 | WA-CMD-01 | H |
| WA-CMD-03 | Watcher | [Command] 내용이 수정된 파일 재분석 및 위키 부분 재생성 후 DB 갱신 | REQ-FUNC-025 | WA-CMD-02, ANA-CMD-03 | H |
| WA-QRY-01 | Watcher | [Query] Watcher 데몬 활성 상태, 모드 및 큐 대기건수 반환 | REQ-FUNC-024, 026 | WA-CMD-02 | L |
| WA-TEST-01| Watcher | [Test] 단순 터치(내용 변경 없음) 시 타임스탬프 대조를 통해 이벤트가 Skip 되는지 단위 테스트 | REQ-FUNC-024 | WA-CMD-02 | M |
| WA-TEST-02| Watcher | [Test] 유휴(Idle) 모드에서 N분간 입력 없을 시 큐가 일괄 처리되는지 통합 테스트 | REQ-FUNC-026 | WA-CMD-02 | M |
| WA-FE-01  | Watcher | [UI] 설정 콤보박스 및 상태 아이콘(Query/Command 연동) 렌더링 | REQ-FUNC-023, 024 | API-003 | M |
| WA-FE-02  | Watcher | [UI] 백그라운드 위키 갱신 성공 시 IPC Toast 알림 렌더링 | REQ-FUNC-025 | API-003 | L |

---

## 📌 Epic 7: 분석 통계 및 공통 셸 (Analytics & App Shell)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 (H/M/L) |
|---------|--------------|----------------|---------------|--------------------------|---------------|
| STAT-CMD-01| Analytics | [Command] 딥링크 클릭, Watcher 자동 갱신 발생 시 통계 수치 로깅 DB Insert | REQ-FUNC-028, 030 | DL-CMD-02, WA-CMD-03 | L |
| STAT-QRY-01| Analytics | [Query] WPM 기반 절약 시간, 압축률 등 통계 데이터 집계 후 DTO 반환 | REQ-FUNC-027, 029 | STAT-CMD-01 | M |
| STAT-TEST-01| Analytics | [Test] 250 WPM 공식 기반 절약 시간 수치가 정확히 산출되는지 단위 테스트 | REQ-FUNC-027 | STAT-QRY-01 | L |
| STAT-FE-01 | Analytics | [UI] My Analytics 차트 및 4대 지표 대시보드 렌더링 | REQ-FUNC-027~030 | API-003 | M |
| APP-UI-01  | App Shell | [UI] 전체 앱 레이아웃(React 라우터, 사이드바, 디자인 시스템) 기초 공사 | §3.6 | None | M |

---

## 📌 Epic 8: 시스템 제약 방어 (NFR & Infra)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 (H/M/L) |
|---------|--------------|----------------|---------------|--------------------------|---------------|
| INF-CMD-01| Infra/NFR | [Command] Windows MAX_PATH 초과 및 권한 거부 시 Crash를 막는 글로벌 예외 핸들러 | REQ-NF-007 | DB-001 | M |
| INF-CMD-02| Infra/NFR | [Command] 로그 파일 로테이션 (50MB/30일) 및 Config 포팅 (JSON) | REQ-NF-014, 015 | None | L |
| INF-TEST-01| Infra/NFR | [Test] k6 등을 활용하여 1,000개 파일 스캔 시 p95 < 5,000ms 기준 부하 테스트 작성 | REQ-NF-001 | SCAN-CMD-01 | M |
| INF-TEST-02| Infra/NFR | [Test] 외부 클라우드 통신(Telemetry) 코드가 없는지 확인하는 네트워킹 격리 테스트 | REQ-NF-005 | API-003 | M |

---

## ✅ 요약 및 기대 효과 (v4.0)

| 분류 | 태스크 수 |
|------|----------|
| **DB & API 명세 (Contract)** | 7 |
| **Command 로직 (Write)** | 18 |
| **Query 로직 (Read)** | 9 |
| **자동화 테스트 (AC → Test)** | 14 |
| **UI/UX 프론트엔드 컴포넌트** | 16 |
| **Infra & NFR 방어** | 4 |
| **총합** | **68** |

### 💡 AI 오케스트레이션 향상점
1. **단일 진실 공급원(SSOT) 확보:** 맨 첫 번째 Epic 0에서 DB 스키마와 DTO, Mock API를 완성하므로, FE 에이전트와 BE 에이전트가 동시에 투입되어도 엇갈리지 않습니다.
2. **닫힌 문맥(Closed Context):** 기존 CRUD 통짜 태스크를 Command(저장/수정/삭제)와 Query(조회)로 완벽히 찢어, 에이전트가 단일 함수 작성에만 집중할 수 있게 했습니다. (오작동 확률 최소화)
3. **자동화 테스트 14건 도출:** 사람이 일일이 UI를 클릭해가며 에이전트에게 잔소리할 필요 없이, **"테스트 태스크(e.g., WS-TEST-01)를 먼저 통과시킨 후 다음 작업으로 넘어가라"**라고 지시할 수 있는 기틀이 마련되었습니다.
