---
name: Feature Task
title: "[Feature] APP-001: Next.js App Router 기반 프로젝트 초기화 및 컴포넌트 아키텍처"
labels: 'feature, priority:high, frontend'
---

## 🎯 Summary
- 기능명: [APP-001] Next.js App Router 기반 프로젝트 초기화 및 컴포넌트 아키텍처
- 목적: CorpBrain의 통합 풀스택 프레임워크인 Next.js(App Router) 프로젝트를 초기화하고, 프론트엔드와 백엔드(Server Actions/Route Handlers)가 유기적으로 통신할 수 있는 기본 디렉토리 구조를 확립한다.

## 🔗 References (Spec & Context)
- SRS 문서: 프롬프트 13-9 추가 항목 (C-TEC-001, C-TEC-002)

## 📝 Task Breakdown
- [ ] `create-next-app@latest` 명령어를 통해 최신 Next.js 프로젝트 초기화 (TypeScript, App Router, Tailwind CSS 옵션 활성화).
- [ ] 프로젝트 디렉토리 구조 셋업: `app/` (라우팅), `components/` (UI), `lib/` (유틸리티/DB), `actions/` (Server Actions), `api/` (Route Handlers).
- [ ] `tsconfig.json` 절대 경로(`@/*`) 설정 및 ESLint / Prettier 초기 룰 구성 (INF-003 CI 기준에 부합하도록).
- [ ] 전역 상태 관리(Zustand 또는 React Context)를 위한 스토어 기본 템플릿(세션/워크스페이스 정보) 작성.
- [ ] P2(CQRS) 패턴 준수를 위해 Server Actions(Write)와 Client Fetching(Read) 간의 데이터 흐름 가이드라인(README) 작성.

## ✅ Acceptance Criteria (BDD)
- **Given**: 개발자가 초기화된 프로젝트 저장소를 클론(Clone) 받았을 때
- **When**: `npm run dev` 명령어를 실행하면
- **Then**: Next.js 기본 인덱스 페이지가 에러 없이 로드되어야 하며, 절대 경로(`@/components/...`) 임포트가 정상적으로 작동해야 한다.

## 🛡️ Constraints (NFRs)
- 아키텍처 제약: C-TEC-001에 명시된 대로 프론트엔드와 백엔드를 별도의 리포지토리로 분리하지 않으며, 단일 모노레포(Monorepo) 형태의 폴더 구조를 유지해야 한다.
- 개발 환경 제약: 로컬 개발 환경의 DB 연동은 Prisma + SQLite를 사용할 수 있도록 초기 `.env.local` 템플릿을 제공해야 한다 (C-TEC-003).
