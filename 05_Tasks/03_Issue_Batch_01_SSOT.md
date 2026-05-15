# Batch 1: 데이터 계약 (SSOT)

본 문서는 CorpBrain MVP 개발을 위한 GitHub Issue 명세서 중 첫 번째 배치(Data Contract - SSOT)에 해당하는 3건의 태스크 명세를 포함합니다.

---

## Issue 1: SSOT-001

```markdown
---
name: Feature Task
title: "[Feature] SSOT-001: API Contract/DTO 스키마 정의 (OpenAPI Spec)"
labels: 'feature, priority:high'
---

## 🎯 Summary
- 기능명: [SSOT-001] API Contract/DTO 스키마 정의 (OpenAPI Spec)
- 목적: 전체 기능 개발의 기준점(SSOT)이 되는 REST API Endpoint의 입출력 규격 및 DTO 모델을 OpenAPI 스펙으로 정의하여 프론트엔드와 백엔드의 병렬 개발을 지원한다.

## 🔗 References (Spec & Context)
- SRS 문서: §6.1 API Endpoint List, §3.3.1 Internal Core API

## 📝 Task Breakdown
- [ ] Next.js (App Router) 환경의 Route Handlers에 맞춰 `api/v1` 구조의 설계 문서 작성.
- [ ] `/webhook/slack` 및 `/webhook/jira` 인바운드 수신용 Endpoint 스키마 정의 (Request Payload 형식 검증 로직 포함).
- [ ] `POST /api/v1/drafts/generate` Endpoint의 Request DTO(`workspace_id`, `time_range`, `channels`, `masking_enabled`) 및 Response DTO 스키마 명세 작성.
- [ ] `PATCH /api/v1/drafts/{draft_id}/review` Endpoint의 Request DTO(`action`, `edits`, `publish_to`) 및 Response DTO 스키마 명세 작성.
- [ ] `POST /api/v1/masking-rules` 및 조회 API(`GET /api/v1/drafts/{id}`, `GET /api/v1/reports/roi`, `GET /api/v1/health`)의 입출력 DTO 명세 작성.
- [ ] OpenAPI 3.0(또는 3.1) 포맷의 YAML/JSON 파일 생성 (또는 Swagger UI 연동 설정 파일 작성).

## ✅ Acceptance Criteria (BDD)
- **Given**: 프론트엔드 개발자가 API 문서를 참조하려는 상황에서
- **When**: OpenAPI 스펙 파일(또는 Swagger/Redoc UI)을 열람하면
- **Then**: 총 8개의 코어 API Endpoint에 대한 Request/Response 스키마, 데이터 타입, 필수 여부, 그리고 에러 응답 포맷(400, 401, 403, 429 등)이 명확히 정의되어 있어야 한다.

## 🛡️ Constraints (NFRs)
- 보안 제약: 각 API 경로에 필요한 인증 헤더(Bearer Token) 및 필요 권한이 명시되어야 한다.
- 아키텍처 제약: P1(데이터·계약 우선) 원칙에 따라 이 스키마가 승인되기 전에는 백엔드 비즈니스 로직(AIE, API Core) 구현을 시작할 수 없다.
```

---

## Issue 2: SSOT-002

```markdown
---
name: Feature Task
title: "[Feature] SSOT-002: 개발용 Mock 데이터 Seed/Fixture 생성"
labels: 'feature, priority:high'
---

## 🎯 Summary
- 기능명: [SSOT-002] 개발용 Mock 데이터 Seed/Fixture 생성
- 목적: UI 조립(APP) 및 데이터 수집/조회 테스트에 즉시 활용할 수 있도록, 도메인 모델에 부합하는 정교한 Mock 데이터(Seed)를 생성하여 병목을 제거한다.

## 🔗 References (Spec & Context)
- SRS 문서: §6.2 Entity & Data Model

## 📝 Task Breakdown
- [ ] 데이터베이스 스키마(WORKSPACE, INTEGRATION, RAW_EVENT, MASKING_RULE, SEMANTIC_DRAFT) 구조에 맞춘 JSON 기반 Mock Data Fixture 작성.
- [ ] `WORKSPACE` 데이터 생성: 'active', 'suspended', 'trial' 상태별 워크스페이스(plan_type 포함) 3건 이상 생성.
- [ ] `RAW_EVENT` 데이터 생성: Slack, Jira로부터 수집된 가상의 원시 메시지(deeplink_url 포함) 20건 이상 생성.
- [ ] `MASKING_RULE` 데이터 생성: 정규식 타입(pii_regex) 및 커스텀 키워드(custom_keyword) 타입 각각 2건 이상 생성.
- [ ] `SEMANTIC_DRAFT` 데이터 생성: 병합된 초안, `deeplink_map` (100% 매핑), pending/approved 상태의 데이터, edit_history 객체 포함 데이터 생성.
- [ ] 개발 환경용 DB에 해당 Mock 데이터를 자동 주입할 수 있는 Seed 스크립트 작성.

## ✅ Acceptance Criteria (BDD)
- **Given**: 빈 개발용 데이터베이스가 준비된 상태에서
- **When**: 개발자가 `npm run seed` (또는 동등한 Seed 명령어)를 실행하면
- **Then**: WORKSPACE, INTEGRATION, RAW_EVENT, MASKING_RULE, SEMANTIC_DRAFT 테이블에 관계(Foreign Key)가 올바르게 매핑된 Mock 데이터가 즉시 적재되어야 한다.
- **Then**: 프론트엔드에서 목록 조회 및 상세 조회 테스트를 수행할 때 정상적인 뷰가 렌더링될 수 있는 형태의 데이터여야 한다.

## 🛡️ Constraints (NFRs)
- 보안 제약: Mock 데이터에는 실제 운영 환경의 PII(개인식별정보)나 민감한 회사 정보가 절대 포함되어서는 안 되며, 모두 가상의 더미 데이터(faker 등 활용)로 구성해야 한다.
- 아키텍처 제약: PostgreSQL의 제약 조건(NOT NULL, UNIQUE 등) 및 JSONB 컬럼 포맷에 완벽하게 일치해야 한다.
```

---

## Issue 3: SSOT-003

```markdown
---
name: Feature Task
title: "[Feature] SSOT-003: Webhook 인바운드 Payload 정규화 스키마 정의 (Slack/Jira)"
labels: 'feature, priority:high'
---

## 🎯 Summary
- 기능명: [SSOT-003] Webhook 인바운드 Payload 정규화 스키마 정의 (Slack/Jira)
- 목적: 서로 다른 형태의 Slack, Jira Webhook Payload를 내부 파이프라인에서 공통으로 처리할 수 있도록 단일화된 내부 정규화 스키마(Adapter DTO)를 정의한다.

## 🔗 References (Spec & Context)
- SRS 문서: §3.1 External Systems, §6.1 API Endpoint (#1, #2)

## 📝 Task Breakdown
- [ ] Slack Events API의 `message.channels` Payload 구조 분석 및 타입 정의.
- [ ] Jira Webhooks의 `issue_created`, `issue_updated`, `comment_created` Payload 구조 분석 및 타입 정의.
- [ ] 이기종 플랫폼의 Payload를 공통 형식으로 변환할 `NormalizedEventDTO` 인터페이스 정의 (필수 필드: `integration_id`, `author_id`, `author_name`, `raw_content`, `source_deeplink_url`, `timestamp`, `channel_name`).
- [ ] Slack/Jira 각 플랫폼에서 `source_deeplink_url`을 추출하거나 조립하는 규칙 문서화.
- [ ] 데이터 파서(Parser) 모듈(Adapter 패턴)이 참조할 입출력 규격 명세 작성.

## ✅ Acceptance Criteria (BDD)
- **Given**: Slack과 Jira에서 각각 다른 형태의 JSON Webhook 이벤트가 들어오는 상황에서
- **When**: 해당 페이로드를 정규화 스키마(`NormalizedEventDTO`)에 매핑할 때
- **Then**: 정보의 누락 없이 공통 필드들(`author_name`, `raw_content`, `source_deeplink_url`, `timestamp`)이 완벽히 채워져야 한다.
- **Then**: TypeScript (또는 사용 언어) 환경에서 두 데이터 소스 모두 컴파일 타임 에러 없이 단일 큐 데이터 모델로 변환될 수 있어야 한다.

## 🛡️ Constraints (NFRs)
- 성능 제약: Payload 변환 및 정규화 과정의 지연 시간은 오버헤드가 거의 없어야 한다.
- 아키텍처 제약: 데이터 수집 태스크(ING-001~006) 개발 전 선행되어야 하며, 데이터 유실을 방지하기 위해 Null 허용 여부(Nullable)가 명확히 선언되어야 한다.
```
