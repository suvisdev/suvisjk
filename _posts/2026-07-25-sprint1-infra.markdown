---
layout: default
title: "Sprint 1 — 기반 구축 완료"
date: 2026-07-25
categories: [sprint, infra]
---

# Sprint 1 — 기반 구축 완료

## 작업 요약

프로젝트 인프라와 인증 체계를 구축했다.

| 항목 | 내용 |
|------|------|
| 백엔드 | FastAPI + Clean Architecture (Hexagonal, Ports & Adapters) |
| DB | PostgreSQL (Neon) + SQLAlchemy 2.0 Async, Redis |
| 프론트엔드 | Next.js 16 (App Router) + TypeScript 5.7 + Tailwind CSS 4 |
| 인프라 | Docker Compose (postgres · redis · pgadmin · cloudflared) |
| 인증 | JWT Bearer + Google · Kakao · Naver OAuth 2.0 |

## 아키텍처 결정

- **모듈러 모놀리식 + Star Topology** 채택 — 단일 배포 단위에서 앱 간 경계를 명확히 분리
- Hub(ontology)가 공통 크롤링·AI 파이프라인·이벤트 버스를 담당하고, Spoke(mova·gildle 등)가 각 도메인 로직을 담당
- `import-linter`로 Spoke 간 직접 의존 금지 규칙을 자동 검사

## 인증 체계

```
클라이언트 → route.ts 프록시 → 백엔드 (JWT 검증)
                                    │
                        ┌───────────┼───────────┐
                        ▼           ▼           ▼
                   Google OAuth  Kakao OAuth  Naver OAuth
```

- `require_admin` 공통 가드로 관리자 전용 API 보호
- RS256 + HS256 이중 검증 폴백 구현
- IDOR 방어를 위한 토큰 기반 소유권 검증 원칙 수립
