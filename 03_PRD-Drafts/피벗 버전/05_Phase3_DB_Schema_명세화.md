# Phase 3: 내부 구조 및 DB 스키마 추적성 확보 (Observability) 초안

단순히 "캐싱한다", "실행 취소(Undo)를 지원한다"는 추상적인 요구사항을 실제 구현 가능한 **데이터 관측성(Data Observability)** 관점에서 설계한 명세입니다.

---

## 1. 상태 관리를 위한 SQLite 로컬 DB 스키마 (핵심 테이블)

앱의 진행 상황을 저장하고 Undo 기능을 지원하기 위해, 로컬에 SQLite(`corpbrain_meta.db`)를 구축하고 아래 두 가지 핵심 테이블을 운영합니다.

### A. `File_Meta` (파일 파싱 및 캐싱 관리 테이블)
파일이 변경되었는지 감지하고, 불필요한 재파싱/LLM API 호출을 막기 위한 메타데이터를 저장합니다.
- `file_id` (PK) : UUID
- `original_path` : 원본 파일 절대 경로
- `file_hash` : 파일 내용의 SHA-256 해시값 (캐싱 적중 여부 판단 기준)
- `last_modified` : OS 상의 수정 시간 (mtime)
- `extracted_text` : 텍스트 추출 결과물 (성공 시 캐시)
- `analysis_status` : 상태 값 (PENDING, PARSING, LLM_PROCESSING, DONE, ERROR)

### B. `Rename_History` (Batch Rename 실행 취소 지원 테이블)
F4(일괄 폴더/파일명 개편) 기능 수행 시 "100% 원복"을 보장하기 위해 변경 이력을 기록합니다.
- `job_id` (PK) : 일괄 변경 작업 단위의 UUID
- `file_id` (FK) : 대상 파일 ID
- `old_path` : 변경 전 절대 경로
- `new_path` : 변경 후 절대 경로
- `status` : 현재 상태 (APPLIED, REVERTED)

## 2. 로컬 캐시 (Cache) 업데이트 정책 (PRD F3 보완)
**[문제 상황]** "변경되지 않은 파일은 재분석하지 않음" 이라는 조건의 판별식 부재.
**[To-Be 방어 로직]**
- **변경 감지 기준:** 옵션 2(심층 분석) 재실행 시, 파일의 `last_modified`가 기존 DB의 값과 다를 경우에만 `file_hash`를 새로 계산하여 비교합니다. (성능 최적화를 위해 매번 무거운 해시 연산을 돌리지 않음)
- 해시값이 기존과 동일하면 LLM에 재질의하지 않고 `File_Meta.extracted_text`를 100% 재활용합니다.

## 3. 로컬 딥링크 프로세스 흐름 (PRD F5 보완)
**[문제 상황]** "앱 내부 뷰어 연결"은 MVP 스펙을 기하급수적으로 부풀리는 위험 요소임.
**[To-Be 방어 로직]**
- **구현 방식:** 보안 및 확장성(오버스펙 방지)을 위해 앱 내부에 별도의 HWP, PDF 뷰어를 탑재하지 않습니다.
- **실행 로직:** 사용자가 위키 마크다운 내부의 `[원문 열기](file:///C:/.../원본.docx)` 딥링크를 클릭하면, 앱(Electron/Tauri)의 OS Native 브릿지 함수(예: `shell.openPath` 또는 Python의 `os.startfile`)를 호출하여 **사용자 PC에 설치된 기본 연결 프로그램(Word, Acrobat 등)으로 문서를 띄웁니다.**
