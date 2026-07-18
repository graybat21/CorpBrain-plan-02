# GitHub Issue Specifications - Epic 8: 시스템 제약 방어 (NFR & Infra)

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] INF-CMD-01: Windows `MAX_PATH` 초과 및 권한 거부 글로벌 예외 처리"
labels: 'feature, backend, priority:high, infrastructure'
assignees: ''
---

## :dart: Summary
- 기능명: [INF-CMD-01] 글로벌 예외 핸들러 구축
- 목적: 파일 스캔 및 읽기 작업 시 빈번히 발생하는 OS 레벨 에러(긴 경로, 권한 거부, 파일 락)가 애플리케이션 전체 크래시(Crash)로 이어지지 않도록 방어하는 안전망을 구축한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: `REQ-NF-007`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] Windows `MAX_PATH` (260자) 우회를 위한 `\\?\` Prefix 자동 붙임 로직 또는 런타임 Manifest 설정 확인
- [ ] 파일 시스템 접근 시 `PermissionError`, `OSError` 등을 포괄하는 글로벌 예외 캐치(Interceptor) 구현
- [ ] 에러 발생 파일은 조용히 Skip 하고 로깅 시스템에 기록(Warning)하는 로직 적용

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 권한 없는 폴더 스캔
- Given: 스캔 대상 폴더 안에 OS에서 접근을 막아둔 관리자 전용 숨김 폴더가 있음
- When: 워크스페이스 스캔을 실행함
- Then: 애플리케이션이 죽지(Crash) 않고, 해당 폴더만 건너뛴 후 나머지 파일 스캔을 정상적으로 완료한다.

## :gear: Technical & Non-Functional Constraints
- 안정성: 어떠한 파일 시스템 예외도 메인 이벤트 루프를 중단시켜서는 안 됨.

## :construction: Dependencies & Blockers
- Depends on: DB-001
- Blocks: None

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] INF-CMD-02: 로그 파일 로테이션 (50MB/30일) 및 Config 포팅 (JSON)"
labels: 'feature, backend, priority:low, infrastructure'
assignees: ''
---

## :dart: Summary
- 기능명: [INF-CMD-02] 인프라 유틸리티
- 목적: 운영 단계에서 로그 파일이 디스크를 꽉 채우는 것을 방지하기 위해 로테이션 정책을 적용하고, 설정 파일(JSON)을 외부로 분리한다.

## :link: References (Spec & Context)
- SRS 문서: `REQ-NF-014, 015`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 로깅 라이브러리(Winston, Python `logging` 등) 설정에 50MB 제한 및 30일 보관 옵션 부여
- [ ] 하드코딩된 환경 변수들을 `config.json` 이나 `.env`로 포팅
- [ ] 앱 실행 시 Config 파일 누락 검증 부트스트랩 작성

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 로그 로테이션 발동
- Given: 로그 파일 크기가 50MB에 도달함
- When: 새로운 로그가 기록됨
- Then: 기존 로그 파일이 압축 백업(또는 이름 변경)되고, 새로운 빈 로그 파일에 기록이 이어진다.

## :construction: Dependencies & Blockers
- Depends on: None

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] INF-TEST-01: 1,000개 파일 스캔 시 p95 < 5,000ms 성능 부하 테스트"
labels: 'test, backend, priority:medium, performance'
assignees: ''
---

## :dart: Summary
- 목적: 애플리케이션의 핵심 병목 구간인 '로컬 파일 스캔' 시, NFR 명세에 따른 응답 시간을 달성하는지 테스트 코드를 통해 자동 검증한다.

## :link: References (Spec & Context)
- SRS 문서: `REQ-NF-001`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 1,000개의 더미 파일이 담긴 가상 디렉토리 생성 스크립트 작성
- [ ] 해당 디렉토리를 타겟으로 스캔 커맨드(API 혹은 함수) 호출 벤치마크 작성
- [ ] 실행 시간(Duration)을 측정하여 p95 기준 5,000ms를 초과하면 테스트를 실패(Fail)하도록 단언(Assert)

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 스캔 성능 충족
- Given: 1,000개의 Mock 파일 트리
- When: 성능 테스트 스위트를 구동함
- Then: 5초(5000ms) 이내에 `File_Meta` Insert까지 모두 끝나고 테스트가 Pass된다.

## :construction: Dependencies & Blockers
- Depends on: SCAN-CMD-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] INF-TEST-02: 외부 클라우드(Telemetry) 통신 완전 격리 테스트"
labels: 'test, backend, priority:high, security'
assignees: ''
---

## :dart: Summary
- 목적: 기업 보안상 폐쇄망 동작이 보장되어야 하므로, LLM Option B 구동 중일 때 어떠한 외부 분석(Telemetry) 트래픽도 발생하지 않음을 테스트 코드로 검증한다.

## :link: References (Spec & Context)
- SRS 문서: `REQ-NF-005`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 테스트 환경 구동 시 아웃바운드 HTTP/TCP 요청을 모니터링/인터셉트 하는 툴 세팅
- [ ] 워크스페이스 스캔 및 로컬(Ollama) 위키 생성 파이프라인 전체 실행
- [ ] 실행 도중 `127.0.0.1`이나 내부 네트워크를 제외한 외부(인터넷)망으로의 요청이 단 1건이라도 잡히면 Fail 처리하는 Assertion 추가

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 완전 오프라인 모드 검증
- Given: Option B로 설정된 앱 환경과 네트워크 요청 감지기가 켜져 있음
- When: 앱 내의 모든 기능(스캔, 딥링크, Watcher)을 한 바퀴 순회함
- Then: 감지기 로그에 외부망 호출 기록이 전혀 없어야 하며 테스트가 성공한다.

## :construction: Dependencies & Blockers
- Depends on: API-003, LLM-CMD-01
