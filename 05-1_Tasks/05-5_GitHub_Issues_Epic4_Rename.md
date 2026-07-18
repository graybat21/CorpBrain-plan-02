# GitHub Issue Specifications - Epic 4: 파일명 일괄 변경 (Batch Rename)

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] RN-CMD-01: LLM 추천 네이밍 템플릿 호출 및 Diff 임시 저장"
labels: 'feature, backend, priority:high, llm'
assignees: ''
---

## :dart: Summary
- 기능명: [RN-CMD-01] 파일명 추천 Diff 생성 (Command)
- 목적: 파일 메타와 컨텍스트를 LLM에 전달하여 일관된 규칙의 파일명 추천안을 받고, 기존 이름과 새 이름이 매핑된 Diff 상태를 임시로 DB에 저장한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: `REQ-FUNC-016`
- DTO 명세: `API-003`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 파일 메타데이터(현재 이름, 확장자, 뎁스) 기반 Rename 프롬프트 구성 로직 작성
- [ ] LLM(Option A/B) 호출 및 JSON Array 형태의 결과(원래 이름, 제안된 이름) 수신/파싱
- [ ] 수신된 Diff 매핑 데이터를 `Rename_History` 혹은 임시 테이블에 `status='pending'` 상태로 Insert

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 올바른 Diff 포맷 수신
- Given: 무작위 이름의 파일 3개가 주어짐
- When: Rename 추천을 요청함
- Then: 3개 파일에 대한 규칙적인 네이밍 제안이 담긴 매핑 리스트가 임시 상태로 DB에 저장된다.

## :construction: Dependencies & Blockers
- Depends on: API-003, LLM-CMD-01
- Blocks: RN-QRY-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] RN-QRY-01 / RN-FE-01: 파일명 Diff 매핑 리스트 반환 및 미리보기 테이블 렌더링"
labels: 'feature, fullstack, priority:medium'
assignees: ''
---

## :dart: Summary
- 목적: 백엔드에서 생성된 임시 Diff 상태(Before/After) 리스트를 프론트엔드로 전달하고, 사용자 확인을 위한 시각적 테이블을 렌더링한다.

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] (백엔드) `status='pending'` 상태인 Rename 매핑 데이터를 SELECT 및 DTO 반환
- [ ] (프론트엔드) 반환받은 리스트를 기반으로 왼쪽(Old, 붉은색), 오른쪽(New, 초록색) 테이블 UI 컴포넌트 개발

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: Diff 데이터 UI 표출
- Given: 백엔드에 5건의 Rename Pending 내역이 있음
- When: 프론트엔드에서 데이터를 조회(Query)함
- Then: 테이블에 5개의 Before/After 행이 렌더링되며, 확장자가 유지되는지 시각적으로 확인된다.

## :construction: Dependencies & Blockers
- Depends on: RN-CMD-01

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] RN-CMD-02: 승인된 Diff 기반 OS 레벨 물리 파일 Rename 및 내역 확정"
labels: 'feature, backend, priority:high, os'
assignees: ''
---

## :dart: Summary
- 기능명: [RN-CMD-02] 파일명 변경 실행 (Apply Command)
- 목적: 사용자가 제안을 승인하면 실제로 OS 레벨에서 파일 이름을 변경하고(Rename), DB 기록을 확정(Applied)한다.

## :link: References (Spec & Context)
- SRS 문서: `REQ-FUNC-018`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] OS `os.rename()` 또는 `shutil.move()` (Python/Node.js fs 모듈) API를 호출하여 물리 파일명 변경
- [ ] 이름 변경 중 권한 부족이나 파일 열림 에러(Lock) 발생 시 해당 항목 Skip 및 Rollback 플래그 처리
- [ ] 모두 성공 혹은 일부 성공 상태를 `Rename_History` 테이블에 업데이트(`status='applied'`)

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 정상적인 파일 이름 일괄 변경
- Given: 승인된 Diff 리스트가 존재하며 파일 잠금이 없음
- When: Apply 명령을 호출함
- Then: OS 폴더에서 실제 파일 이름이 변경되고 DB 상태가 적용됨(Applied)으로 바뀐다.

## :gear: Technical & Non-Functional Constraints
- 트랜잭션/OS 에러 방어: 중간에 OS 에러가 나더라도 앱이 크래시되지 않도록 `try-catch` 필수

## :checkered_flag: Definition of Done (DoD)
- [ ] OS 레벨 파일명 변경 엣지 케이스 처리 완료

## :construction: Dependencies & Blockers
- Depends on: RN-QRY-01
- Blocks: RN-CMD-03

<br><br>

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] RN-CMD-03 / RN-TEST-01: `Rename_History` 기반 100% 원복(Undo) 및 무결성 테스트"
labels: 'feature, backend, priority:high'
assignees: ''
---

## :dart: Summary
- 기능명: [RN-CMD-03] 실행 취소 (Undo Command)
- 목적: 사용자가 변경된 파일명을 되돌리고 싶을 때, DB 히스토리를 기반으로 100% 원복을 수행한다.

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 최근 변경된 `Rename_History` 항목 조회 (역순)
- [ ] OS `rename` 함수를 호출해 새로운 이름에서 옛 이름으로 되돌리기
- [ ] 해당 History 상태를 `status='reverted'`로 변경
- [ ] (RN-TEST-01) Diff를 통한 변경 -> Undo를 통한 원복 시 물리 경로와 해시값이 정확히 일치하는지 확인하는 통합 테스트 구현

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: Undo를 통한 100% 롤백
- Given: 방금 Rename 기능으로 이름이 변경된 상태임
- When: Undo(원복) 기능을 실행함
- Then: 물리적 파일 이름이 기존과 100% 동일하게 되돌아온다.

## :construction: Dependencies & Blockers
- Depends on: RN-CMD-02
