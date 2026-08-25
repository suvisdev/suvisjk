---
layout: default
title: Eval-ATS — 팀 프로젝트
nav_order: 11
permalink: /ats/
---

# Eval-ATS

**AI 기반 채용 프로세스 자동화 및 지원자 통합 관리 플랫폼**
{: .fs-5 .fw-300 }

팀 SEUK 5인 공동 개발 프로젝트. 이력서 AI 파싱, 칸반 보드, Tool-Calling Agent, RAG 질의응답까지 — 채용 프로세스를 하나의 플랫폼에서 자동화한다.
{: .fs-5 .fw-300 }

---

## 프로젝트 요약

| 항목 | 내용 |
|------|------|
| 사업명 | AI 기반 채용 프로세스 자동화 및 지원자 통합 관리 플랫폼 |
| 시스템명 | Arda (Eval-ATS) |
| 개발 기간 | 2026.08.20 ~ 2026.10.27 (10주, 5스프린트) |
| 개발 인원 | 5명 |
| 개발 방법론 | 애자일 스크럼 (2주 단위 스프린트) |

---

## 나의 역할

| 포지션 | 담당 영역 | 주요 기술 |
|--------|----------|----------|
| AI / Backend | 이력서 파싱 엔진, Tool-Calling Agent, RAG Q&A 엔진, 평가 화면·API 설계 | Python, Claude API, pgvector |

**스프린트별 담당 업무:**

| 스프린트 | 기간 | 담당 업무 |
|---------|------|----------|
| Sprint 1 | 08/20 ~ 09/02 | Docker + EC2 배포, 컨테이너 구성 |
| Sprint 2 | 09/03 ~ 09/16 | JWT 인증 구현, 권한 3종(관리자/채용담당자/면접관) 설계 |
| Sprint 3 | 09/17 ~ 09/30 | 역할별 접근 제어 (RBAC) 적용 |
| Sprint 4 | 10/01 ~ 10/14 | CI/CD 파이프라인 (GitHub Actions) |
| Sprint 5 | 10/15 ~ 10/27 | 헬스체크·모니터링, 최종 배포 |

---

## 주요 기능

| 기능 | 설명 |
|------|------|
| 지원자 칸반 보드 | 카드를 드래그해 단계 이동(지원 접수 → 서류 검토 → 면접 → 최종 합격/불합격). 모든 이동은 단계 이력으로 기록 |
| 단계 변경 자동 메일 | 단계 이동 시 지원자에게 메일 자동 발송. SQS 큐 + 워커로 비동기 처리, 실패 시 재시도 |
| 공고 관리 · 공개 지원 링크 | 채용 공고 등록·관리. 지원자는 로그인 없이 외부 공개 링크로 지원서 제출 |
| 이력서 S3 업로드 | presigned URL로 브라우저에서 S3에 직접 업로드 — 파일이 API 서버를 거치지 않는다 |
| AI 이력서 파싱 | PDF/DOCX에서 이름·학력·경력·기술스택을 구조화 데이터로 자동 추출 |
| Tool-Calling Agent | 자연어 명령으로 지원자 검색·일정 조회·통계 요청을 처리하는 AI 에이전트 |
| RAG 질의응답 | 이력서·채용 공고를 임베딩하여 자연어 질의에 답변 (pgvector) |

{: .note }
AI가 합불을 결정하지 않는다. 합불 확정은 항상 사람이 한다. (ADR-0003)

---

## 기술 스택

**BACKEND** — Python · FastAPI · PostgreSQL

**FRONTEND** — React · Vite · TypeScript

**INFRA** — Docker · AWS (EC2 · S3 · SES · SQS) · GitHub Actions · Vercel

**AI** — Claude API · pgvector · LangGraph

---

## 시스템 아키텍처

| 계층 | 기술 | 배포 |
|------|------|------|
| Frontend | React + TypeScript + Vite | Vercel |
| Backend API | FastAPI (Python) | EC2 · Docker |
| Database | PostgreSQL + pgvector | Aurora Serverless v2 |
| 파일 저장 | S3 (presigned URL 업로드, SSE 암호화) | AWS S3 |
| 메일 발송 | SES + SQS (비동기 큐) | AWS SES/SQS |
| CI/CD | GitHub Actions | 자동 배포 |

---

## 팀 구성

| 역할 | 이름 | 포지션 | 담당 영역 |
|------|------|--------|----------|
| PM | 이재우 | AWS | 클라우드 인프라 구성 및 배포 |
| 개발 | 이우정 | Backend | FastAPI 서버, API 설계 및 DB 연동 |
| 개발 | 박소연 | Frontend | React UI, 칸반 보드, 지원자 관리 화면 |
| 개발 | **진수택** | **AI** | **이력서 파싱, Tool-Calling Agent, RAG 엔진** |
| 개발 | 김민아 | App | 모바일/앱 클라이언트 개발 |

---

## 개발 요구사항

| 요구 ID | 기능 | 검수 기준 |
|---------|------|----------|
| SFR-001 | 이력서 파싱 — PDF/DOCX 업로드 시 자동 추출 | 주요 필드 추출 정확도 90% 이상 |
| SFR-002 | 칸반 보드 — 드래그 단계 전환 | 상태 전환 시 DB 반영 200ms 이내 |
| SFR-003 | Tool-Calling Agent — 자연어 명령 처리 | 도구 호출 정확도 85% 이상 |
| SFR-004 | RAG 질의응답 — 이력서·공고 기반 답변 | Top-5 Recall@5 90% 이상 |
| SFR-005 | 지원자 관리 — CRUD, 검색·필터링 | 전체 CRUD 정상 동작, 페이지네이션 |
| SFR-006 | 이메일 알림 — SES 기반 자동 발송 | SQS 큐 연동, 발송 성공률 99% 이상 |

---

{: .note }
이 섹션은 팀 프로젝트 진행에 따라 업데이트됩니다.
