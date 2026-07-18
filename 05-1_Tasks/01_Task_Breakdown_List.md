# CorpBrain MVP 개발 태스크 (Task Breakdown)

본 문서는 `SRS-draft_v0.6_OPUS.md` 문서의 요구사항을 분석하여 실제 개발이 가능한 최소 단위의 Feature로 분리한 실행 계획입니다. UI/UX 디자인, 프론트엔드(FE), 백엔드(BE) 및 인프라(SYS) 작업을 명확히 분리하여 병렬적 개발이 가능하도록 설계했습니다.

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 (H/M/L) |
|---|---|---|---|---|---|
| **WS-UI-01** | 워크스페이스 (Workspace) | 워크스페이스 생성/관리 및 파일 스캔 대시보드 UI/UX 설계 | 4.1.1 F1 | None | M |
| **WS-BE-01** | 워크스페이스 (Workspace) | SQLite 메타데이터 DB 스키마 구축 및 %LocalAppData% 격리 환경 구성 | 4.1.1 F1, 4.2 NF-004 | None | M |
| **WS-BE-02** | 워크스페이스 (Workspace) | 워크스페이스 생성 및 다중 폴더 목록(CRUD) 관리 API 개발 | 4.1.1 F1 | WS-BE-01 | M |
| **WS-BE-03** | 워크스페이스 (Workspace) | 로컬 파일 트리 스캔, 블랙리스트 필터링 및 10,000개 상한 방어 로직 | 4.1.1 F1 | WS-BE-02 | M |
| **WS-BE-04** | 워크스페이스 (Workspace) | 지원 파일 포맷(`.docx`, `.pdf`, `.txt`, `.md`) 텍스트 파서 모듈 개발 | 4.1.1 F1 | None | H |
| **WS-FE-01** | 워크스페이스 (Workspace) | 좌측 히스토리 패널 워크스페이스 목록 연동 및 생성 모달 구현 | 4.1.1 F1 | WS-UI-01, WS-BE-02 | M |
| **WS-FE-02** | 워크스페이스 (Workspace) | 파일 스캔 실행 버튼 연동 및 스캔 통계(파일 수, 용량, 예상시간) 렌더링 | 4.1.1 F1 | WS-UI-01, WS-BE-03 | M |
| **LLM-UI-01** | 하이브리드 LLM (LLM) | 하이브리드 LLM(Option A/B) 설정 및 상태 조회 패널 UI/UX 설계 | 4.1.2 F2 | None | L |
| **LLM-BE-01** | 하이브리드 LLM (LLM) | Hybrid LLM Router 및 추론 요청 공통 인터페이스 설계 | 4.1.2 F2 | None | H |
| **LLM-BE-02** | 하이브리드 LLM (LLM) | 클라우드 API(Option A) 연동 및 네트워크 전송 전 PII 마스킹 모듈 | 4.1.2 F2, 4.2 NF-006 | LLM-BE-01 | H |
| **LLM-BE-03** | 하이브리드 LLM (LLM) | 로컬 Ollama 데몬(Option B) 연동 및 Health Check 검증 모듈 | 4.1.2 F2 | LLM-BE-01 | M |
| **LLM-BE-04** | 하이브리드 LLM (LLM) | Ollama 미설치 환경 대응 원클릭 백그라운드 다운로드/설치 로직 구현 | 4.1.2 F2 | None | H |
| **LLM-FE-01** | 하이브리드 LLM (LLM) | 환경 설정(A/B) 콤보박스 및 상태 조회 API 기반 헬스체크 아이콘 연동 | 4.1.2 F2 | LLM-UI-01, LLM-BE-03 | M |
| **LLM-FE-02** | 하이브리드 LLM (LLM) | Ollama 원클릭 설치 프로그레스 바 UI 렌더링 | 4.1.2 F2 | LLM-UI-01, LLM-BE-04 | M |
| **ANA-UI-01** | 분석 파이프라인 (Analysis) | 고속 분석 리스트, 심층 분석 위키(폴더 탭) 및 진행률 표시 UI/UX 설계 | 4.1.3 F3 | None | M |
| **ANA-BE-01** | 분석 파이프라인 (Analysis) | 폴더/파일명 기반 고속 분석 중요도 점수화 API 로직 개발 | 4.1.3 F3 | WS-BE-03, LLM-BE-01 | H |
| **ANA-BE-02** | 분석 파이프라인 (Analysis) | 문서 텍스트 청킹(Chunking) 및 Vector DB(Chroma/FAISS) 임베딩 모듈 | 4.1.3 F3 | WS-BE-04 | H |
| **ANA-BE-03** | 분석 파이프라인 (Analysis) | LLM 일괄 전송 및 1-Depth 폴더 단위 구조화 위키 마크다운 생성 API | 4.1.3 F3 | ANA-BE-02, LLM-BE-01 | H |
| **ANA-BE-04** | 분석 파이프라인 (Analysis) | 처리 중인 분석(Task) 파일 진척도 기반 프로그레스 및 예상 잔여 시간 계산 API | 4.1.3 F3 | ANA-BE-01, ANA-BE-03 | M |
| **ANA-FE-01** | 분석 파이프라인 (Analysis) | 고속 분석 결과 리스트 상단 하이라이트 뷰어 구현 | 4.1.3 F3 | ANA-UI-01, ANA-BE-01 | M |
| **ANA-FE-02** | 분석 파이프라인 (Analysis) | 1-Depth 폴더별 독립 탭 구조 기반 마크다운 위키 뷰어 컴포넌트 개발 | 4.1.3 F3 | ANA-UI-01, ANA-BE-03 | H |
| **ANA-FE-03** | 분석 파이프라인 (Analysis) | 프로그레스 바 상태 폴링 API 연동 및 UI 프로그레스 표시 | 4.1.3 F3 | ANA-UI-01, ANA-BE-04 | M |
| **RN-UI-01** | 일괄 파일명 개편 (Rename) | AI Naming 템플릿 선택 및 텍스트 Diff 미리보기 테이블 UI/UX 설계 | 4.1.4 F4 | None | M |
| **RN-BE-01** | 일괄 파일명 개편 (Rename) | 파일/폴더 컨텍스트 기반 AI Naming 추천 템플릿 로직 | 4.1.4 F4 | ANA-BE-03 | H |
| **RN-BE-02** | 일괄 파일명 개편 (Rename) | 물리적 OS 파일명 변경 실행 및 Rename_History DB 매핑 API 구현 | 4.1.4 F4 | None | M |
| **RN-BE-03** | 일괄 파일명 개편 (Rename) | Rename_History 참조 기반 100% 실행 취소(Undo) 롤백 API 개발 | 4.1.4 F4, 4.2 NF-009 | RN-BE-02 | M |
| **RN-FE-01** | 일괄 파일명 개편 (Rename) | 기존/신규 이름 텍스트 Diff 비교 테이블 컴포넌트 렌더링 | 4.1.4 F4 | RN-UI-01, RN-BE-01 | M |
| **RN-FE-02** | 일괄 파일명 개편 (Rename) | 일괄 적용(Apply) 및 실행 취소(Undo) IPC 인터페이스 및 실패 팝업 연동 | 4.1.4 F4 | RN-FE-01, RN-BE-02 | M |
| **DL-UI-01** | 딥링크 (Trust-Anchor) | 위키 렌더러 내 딥링크 아이콘 및 파일 유실 툴팁 상태별 UI/UX 설계 | 4.1.5 F5 | None | L |
| **DL-BE-01** | 딥링크 (Trust-Anchor) | 위키 마크다운 단락별 원문 로컬 파일 절대 경로 자동 매핑 및 보존 로직 | 4.1.5 F5 | ANA-BE-03 | H |
| **DL-BE-02** | 딥링크 (Trust-Anchor) | IPC 통신 수신 시 OS 브릿지(`os.startfile`)를 통한 원문 열기 프로세스 | 4.1.5 F5 | None | M |
| **DL-FE-01** | 딥링크 (Trust-Anchor) | 마크다운 내 딥링크 컴포넌트 렌더링 및 파일 유실 시 회색 비활성 처리 | 4.1.5 F5 | DL-UI-01, DL-BE-01 | M |
| **DL-FE-02** | 딥링크 (Trust-Anchor) | 딥링크 클릭 시 브라우저 기본 동작 차단 및 IPC 파일 열기 이벤트 바인딩 | 4.1.5 F5 | DL-FE-01, DL-BE-02 | L |
| **WA-UI-01** | 와처 모듈 (Watcher) | 백그라운드 위키 자동 업데이트 Toast 노티피케이션 UI/UX 설계 | 4.1.6 F6 | None | L |
| **WA-BE-01** | 와처 모듈 (Watcher) | `watchdog` 기반 OS 파일 추가/수정 이벤트 실시간 백그라운드 감지 데몬 | 4.1.6 F6 | WS-BE-01 | H |
| **WA-BE-02** | 와처 모듈 (Watcher) | 사용자 키보드/마우스 이벤트를 체크하는 Idle 모드 판단 로직 및 처리 분기 | 4.1.6 F6 | WA-BE-01 | H |
| **WA-BE-03** | 와처 모듈 (Watcher) | 변경 파일 대상 재파싱, DB 기존 청크 무효화, 부분 위키 재요약 및 병합 자동화 | 4.1.6 F6 | WA-BE-01, ANA-BE-03 | H |
| **WA-FE-01** | 와처 모듈 (Watcher) | 실시간/유휴/수동/끄기 모드 설정 변경 인터페이스 연동 | 4.1.6 F6 | LLM-FE-01 | L |
| **WA-FE-02** | 와처 모듈 (Watcher) | 백그라운드 갱신 성공 IPC 알림 수신 시 Toast 출력 및 해당 탭 위키 리렌더링 | 4.1.6 F6 | WA-UI-01, WA-BE-03 | M |
| **STAT-UI-01**| 생산성 통계 (Analytics)| My Analytics 주요 4대 지표 패널 레이아웃 및 차트 UI/UX 설계 | 4.1.7 F7 | None | M |
| **STAT-BE-01**| 생산성 통계 (Analytics)| WPM 기반 절약 시간, 팩트체크 횟수, 압축률, Watcher 갱신 건수 산출 로직 | 4.1.7 F7 | ANA-BE-03, DL-BE-02 | M |
| **STAT-BE-02**| 생산성 통계 (Analytics)| 시각화 및 클라이언트에 반환하기 위한 Summary 데이터 Aggregation API | 4.1.7 F7 | STAT-BE-01 | M |
| **STAT-FE-01**| 생산성 통계 (Analytics)| 대시보드 그래프 컴포넌트 라이브러리 연동 및 지표 텍스트 렌더링 | 4.1.7 F7 | STAT-UI-01, STAT-BE-02| M |
| **SYS-01**    | 공통 인프라 (Infra) | PyInstaller 등 단일 `.exe` 로컬 데스크톱 애플리케이션 번들링 설정 | 1.5.1 CON-01, CON-02 | WS-FE-01, WS-BE-01 | M |
| **SYS-02**    | 공통 인프라 (Infra) | 앱 비정상 크래시 대비 MAX_PATH(260자) 접근 예외 전역 방어 처리 및 로깅 | 4.2 NF-007 | None | M |
| **SYS-03**    | 공통 인프라 (Infra) | 로컬 파일 로그 최대 50MB 또는 30일 초과 시 자동 Rotation 스크립트 작성 | 4.2 NF-014 | None | L |
| **SYS-04**    | 공통 인프라 (Infra) | 사용자 세팅(LLM, Watcher) Export/Import(JSON/TOML) 기능 구현 | 4.2 NF-015 | WS-BE-01 | M |
