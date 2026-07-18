# GitHub Issue Specifications - Epic 2: 하이브리드 LLM 엔진 (LLM Engine)

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] LLM-CMD-01: LLM 엔진 설정(Option A/B) 변경 및 DB 저장"
labels: 'feature, backend, priority:high'
assignees: ''
---

## :dart: Summary
- 기능명: [LLM-CMD-01] 하이브리드 LLM 설정 관리 (Command)
- 목적: 사용자가 클라우드 API(Option A)와 로컬 프라이빗(Option B - Ollama) 중 선호하는 엔진을 선택하면 이 설정을 데이터베이스에 저장한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: `REQ-FUNC-007`
- DTO 명세: `API-003`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] Option A/B Enum 정의 및 `Settings_Meta` 테이블에 업데이트하는 로직 구현
- [ ] Option B(Ollama) 선택 시 내부적으로 로컬 데몬 구동 여부 플래그를 활성화하도록 연계

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 정상적인 엔진 변경 요청
- Given: 현재 Option A 상태인 시스템
- When: 사용자가 Option B로 설정을 변경하는 요청을 보냄
- Then: DB에 설정이 'Option B'로 반영되고 성공 응답(200 OK)이 반환된다.

## :gear: Technical & Non-Functional Constraints
- 무결성: Enum에 존재하지 않는 엔진 값이 들어올 경우 예외 처리 필수

## :checkered_flag: Definition of Done (DoD)
- [ ] 단위 테스트 작성 및 통과
- [ ] Swagger 스펙상 유효한 값만 전송되도록 Validation 적용

## :construction: Dependencies & Blockers
- Depends on: DB-001, API-003
- Blocks: LLM-CMD-02, LLM-CMD-03

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] LLM-QRY-01 / LLM-TEST-02: 선택된 엔진 연결 상태 확인 (Health Check) 및 테스트"
labels: 'feature, test, backend, priority:medium'
assignees: ''
---

## :dart: Summary
- 기능명: [LLM-QRY-01] LLM 엔진 Health Check
- 목적: 현재 선택된 엔진(Cloud API 또는 Local Ollama)과 정상적으로 통신이 가능한지 핑(Ping)을 날려 상태를 반환한다.

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] Option A인 경우: 외부 네트워크 연결 체크 또는 Cloud API Ping 테스트
- [ ] Option B인 경우: 로컬 호스트(`http://127.0.0.1:11434`)의 Ollama 서버 Ping 동작 확인
- [ ] 상태에 따라 `true/false` DTO 리턴
- [ ] (LLM-TEST-02) 네트워크 단절 및 데몬 미구동 상태에 대한 Mocking 단위 테스트 작성

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: Ollama 데몬 미실행 시 체크
- Given: 설정이 Option B(Ollama)로 지정되어 있으나 백그라운드 프로세스가 내려가 있음
- When: Health Check 쿼리를 실행함
- Then: 연결 시간 초과(Timeout) 등을 감지하여 `status: false` 응답을 반환한다.

## :construction: Dependencies & Blockers
- Depends on: API-003

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] LLM-CMD-02 / LLM-TEST-01: Option A 전송 전 PII 마스킹 인메모리 적용 및 테스트"
labels: 'feature, backend, priority:high, security'
assignees: ''
---

## :dart: Summary
- 기능명: [LLM-CMD-02] 개인정보(PII) 마스킹
- 목적: Option A(클라우드)를 사용할 경우, 추출된 텍스트에 포함된 민감 정보(주민번호, 전화번호, 이메일 등)를 외부로 전송하기 전에 정규식을 이용해 인메모리 상에서 치환(마스킹)한다.

## :link: References (Spec & Context)
- SRS 문서: `REQ-FUNC-008, 009`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 이메일, 전화번호, 주민등록번호 등을 필터링하는 정규표현식(Regex) 모듈 작성
- [ ] 텍스트 전처리 파이프라인(Interceptor) 구현 (인메모리에서만 치환하고 원본 DB나 파일은 수정 금지)
- [ ] (LLM-TEST-01) 다양한 포맷의 PII 문자열에 대한 치환 성공/실패 단위 테스트 촘촘하게 작성
- [ ] 예기치 않은 오류 발생 시 외부 전송 원천 차단(Fail-Safe) 로직 구현

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: PII 마스킹 및 전송 차단 검증
- Given: Option A가 활성화되어 있고, "제 번호는 010-1234-5678 입니다." 라는 청크가 주어짐
- When: LLM API로 텍스트를 전송하기 직전 파이프라인을 통과함
- Then: 문자열이 "제 번호는 ***-****-**** 입니다." 로 치환되어 클라우드에 전송된다.

## :gear: Technical & Non-Functional Constraints
- 보안/성능: 원본 데이터가 영구 수정되지 않도록 얕은 복사/깊은 복사 주의. 정규식 백트래킹 취약점(ReDoS) 대비.

## :checkered_flag: Definition of Done (DoD)
- [ ] LLM-TEST-01 테스트 통과 (다양한 엣지 케이스 포함)

## :construction: Dependencies & Blockers
- Depends on: API-003
- Blocks: ANA-CMD-03

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] LLM-CMD-03: Option B 선택 시 Ollama 데몬 무인 설치 및 백그라운드 모델 Pull"
labels: 'feature, backend, priority:medium'
assignees: ''
---

## :dart: Summary
- 기능명: [LLM-CMD-03] Ollama 무인 설치
- 목적: 완전 오프라인 환경을 위해 사용자가 Option B를 선택하고 데몬이 없을 경우, OS 명령어 레벨에서 백그라운드로 Ollama 설치 및 지정된 경량 모델 풀링(Pull)을 실행한다.

## :link: References (Spec & Context)
- SRS 문서: `REQ-FUNC-010`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 현재 OS(Windows) 타겟 환경에 맞는 설치 여부 확인(명령어 경로 조회)
- [ ] 미설치 시 Installer 다운로드 및 무인(Silent) 실행 스크립트 트리거 로직 구현
- [ ] 설치 후 `ollama pull [model_name]` Subprocess 실행 및 표준 출력(stdout) 파싱
- [ ] 프론트엔드 진행 상황(Progress) 전달용 웹소켓 혹은 Polling 상태 저장

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: Ollama 미설치 환경에서 자동 설치 트리거
- Given: Ollama가 설치되지 않은 Windows 환경
- When: Option B 설정을 저장하고 설치 명령을 백그라운드로 호출함
- Then: 설치 프로세스가 백그라운드에서 실행되며, 진행 상태가 API(상태 조회)를 통해 지속 갱신된다.

## :construction: Dependencies & Blockers
- Depends on: API-003
- Blocks: LLM-FE-02

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] LLM-FE-01/02: LLM 설정 UI 및 설치 프로그레스 화면"
labels: 'feature, frontend, priority:high'
assignees: ''
---

## :dart: Summary
- 목적: Option A/B 엔진 선택 콤보박스와 상태 아이콘, 그리고 Ollama 다운로드 프로그레스 바를 렌더링한다.

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 환경 설정 다이얼로그 혹은 패널 내 Option A/B 선택 UI 구현
- [ ] 5초 주기(또는 SSE)로 서버 상태를 폴링하여 Health Check 아이콘(✅/❌) 토글
- [ ] Option B 선택 시 미설치 상태일 경우 인라인 프로그레스 바(0~100%) 컴포넌트 렌더링

## :construction: Dependencies & Blockers
- Depends on: API-003, LLM-CMD-01, LLM-CMD-03
