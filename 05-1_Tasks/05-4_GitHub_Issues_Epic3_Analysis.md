# GitHub Issue Specifications - Epic 3: 분석 파이프라인 (Analysis)

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] ANA-CMD-01: 폴더/파일명 추출 및 고속 분석 중요도 산출 후 DB 업데이트"
labels: 'feature, backend, priority:high'
assignees: ''
---

## :dart: Summary
- 기능명: [ANA-CMD-01] 구조 기반 고속 분석
- 목적: 파일 내용을 열기 전, 폴더 트리 구조와 파일 이름 패턴(정규식 등)만으로 문서의 중요도와 컨텍스트 가중치를 산출하여 DB를 업데이트한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: `REQ-FUNC-012`
- API 명세: `API-002`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 파일 경로 뎁스(Depth) 및 확장자에 따른 기본 가중치 산정 로직 작성
- [ ] 키워드(예: '기획', '설계', '완료') 매칭에 따른 중요도 가점(Bonus) 부여 로직 구현
- [ ] 산출된 중요도 점수(Score)를 `File_Meta` 테이블에 일괄 업데이트

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 이름 기반 중요도 산출
- Given: 스캔이 완료된 `File_Meta` 테이블에 '최종_기획서.docx' 와 '임시_메모.txt' 가 있음
- When: 고속 분석 프로세스가 트리거됨
- Then: '최종_기획서.docx'의 중요도 점수가 '임시_메모.txt'보다 높게 산출되어 DB에 저장된다.

## :construction: Dependencies & Blockers
- Depends on: SCAN-CMD-01
- Blocks: ANA-FE-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] ANA-CMD-02 / ANA-TEST-01: 문서 파싱 후 텍스트 청킹(Chunking) 및 벡터 DB Insert"
labels: 'feature, backend, priority:high'
assignees: ''
---

## :dart: Summary
- 기능명: [ANA-CMD-02] 심층 분석 - 파싱 및 청킹
- 목적: 물리적 파일(docx, pdf, txt, md)을 열어 텍스트를 추출하고, LLM 처리 한계(Token)에 맞게 의미 단위 청크(Chunk)로 나눈 후 벡터 DB에 저장한다.

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 포맷별 텍스트 추출 어댑터 구현 (`pdfplumber` 등 외부 라이브러리 연동)
- [ ] 문단/구문 단위로 텍스트 분할(Chunking) 알고리즘 적용 (예: 500 토큰 단위, Overlap 50 설정)
- [ ] 텍스트 청크를 임베딩(Embedding)하고 ChromaDB/FAISS에 Insert
- [ ] (ANA-TEST-01) 4개 포맷에 대해 빈 문자열이나 깨진 인코딩 없이 정확히 텍스트가 추출되는지 확인하는 단위 테스트 작성

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: PDF 문서 추출 및 벡터 DB 삽입
- Given: 텍스트가 포함된 유효한 3페이지 짜리 PDF 파일이 주어짐
- When: 심층 파싱을 지시함
- Then: 텍스트가 추출되고 설정된 청크 크기 단위로 쪼개져 벡터 DB 스토리지에 총 N개의 레코드로 저장된다.

## :gear: Technical & Non-Functional Constraints
- 성능: 대용량 PDF 파싱 시 메모리 릭(Memory Leak) 방지를 위해 제너레이터 활용 및 스트리밍 파싱 적용

## :construction: Dependencies & Blockers
- Depends on: DB-002, SCAN-CMD-01
- Blocks: ANA-CMD-03

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] ANA-CMD-03 / ANA-TEST-02: 청크 기반 LLM 위키 마크다운 생성 및 DB Insert"
labels: 'feature, backend, priority:high, llm'
assignees: ''
---

## :dart: Summary
- 기능명: [ANA-CMD-03] 위키 마크다운 생성
- 목적: 벡터 DB에 모인 컨텍스트를 바탕으로 1-Depth 최상위 폴더별로 LLM 프롬프트를 조합하여 체계화된 위키 문서를 작성하고 DB에 영속화한다.

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 1-Depth 폴더를 기준으로 그룹화(Group-by)하여 관련된 청크 문맥(Context) 조회
- [ ] RAG 프롬프트 템플릿 구성 (요약, 추출 구조화 등)
- [ ] LLM(클라우드 또는 Ollama)에 문맥과 프롬프트를 전송하여 Markdown 결과 수신
- [ ] 수신된 Markdown 텍스트를 `Wiki_Content` 테이블에 Insert
- [ ] (ANA-TEST-02) 생성된 위키가 서로 다른 1-Depth 폴더의 컨텍스트를 침범하지 않는지(격리) 검증

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 정상 위키 문서 생성
- Given: 1-Depth 폴더 '01_Frontend'에 연관된 텍스트 청크들이 벡터 DB에 존재함
- When: 위키 생성 커맨드를 호출함
- Then: 프론트엔드 관련 내용만 요약된 마크다운 문서가 LLM을 통해 생성되고 DB에 저장된다.

## :construction: Dependencies & Blockers
- Depends on: ANA-CMD-02, LLM-CMD-01, LLM-CMD-02

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] ANA-QRY-01: 1-Depth 폴더별로 분리 가공된 위키 마크다운 구조 반환"
labels: 'feature, backend, priority:medium'
assignees: ''
---

## :dart: Summary
- 기능명: [ANA-QRY-01] 생성 위키 조회 (Query)
- 목적: 프론트엔드에서 폴더 탭별로 위키 문서를 렌더링할 수 있도록 1-Depth 디렉토리 단위로 구조화된 마크다운을 묶어 반환한다.

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] `Wiki_Content` 테이블에서 특정 워크스페이스 ID를 기준으로 전체 위키 SELECT
- [ ] 폴더명(키) - 마크다운 본문(값) 형태의 JSON 객체/DTO로 데이터 변환 가공

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 다중 탭 위키 응답
- Given: DB에 '01_FE', '02_BE' 두 개의 위키 문서가 있음
- When: 위키 문서 조회를 요청함
- Then: 배열 또는 맵(Dictionary) 형태로 두 폴더명과 내용이 포함된 JSON 데이터를 반환한다.

## :construction: Dependencies & Blockers
- Depends on: ANA-CMD-03
- Blocks: ANA-FE-02

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] ANA-QRY-02 / ANA-FE-03: 분석 진행 상태(Progress) 산출, 반환 및 프론트엔드 렌더링"
labels: 'feature, fullstack, priority:medium'
assignees: ''
---

## :dart: Summary
- 기능명: 진행률 계산 및 폴링 UI 업데이트
- 목적: 비동기로 길게 진행되는 파싱 및 LLM 생성 태스크의 현재 진행 현황(처리 수, 전체 수)을 제공하여 UI에 표시한다.

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] (백엔드) 처리 중인 파일 수 / 전체 스캔 파일 수 비례 계산 로직 작성 (메모리 큐 또는 DB 트래킹)
- [ ] (백엔드) ETA 산출 및 Progress DTO 직렬화 API(Query) 구현
- [ ] (프론트엔드) 주기적(예: 3초)으로 API를 폴링(Polling)하거나 SSE를 통해 값을 받아 프로그레스 바 렌더링
- [ ] (프론트엔드) 작업 완료 상태 수신 시 완료 모달 전환 처리

## :construction: Dependencies & Blockers
- Depends on: ANA-CMD-02, ANA-CMD-03
