---
layout: home
title: Home
nav_order: 0
permalink: /
---

# suvisdev

**AI 영화 추천(Mova) · 반려견 산책 경로(Gildle) 통합 플랫폼**
{: .fs-6 .fw-300 }

LoRA 파인튜닝 추천 + OSM 보행 그래프 경로 최적화를 하나의 모듈러 모놀리식 아키텍처에 통합한 풀스택 프로젝트.
{: .fs-5 .fw-300 }

[데모 사이트](https://suvisdev.cloud){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[GitHub](https://github.com/suvisdev/suvisdev.cloud){: .btn .fs-5 .mb-4 .mb-md-0 }

---

| 항목 | 내용 |
|------|------|
| 개발 기간 | 2026.07 ~ 2026.09 (약 10주) |
| 개발 인원 | 1명 (개인 프로젝트) |
| 역할 | 풀스택 개발 · ML 파이프라인 · 인프라 · 데이터 수집 |

---

## 주요 기능

| 기능 | 설명 |
|------|------|
| AI 맞춤 추천 | LoRA(EXAONE-2.4B AWQ) + Gemini 듀얼 백엔드 개인화 추천 |
| AI 리뷰 생성 | KOBIS·뉴스·위키 크롤링 → Gemini 리뷰 + 별점 자동 산정 |
| 박스오피스 랭킹 | KOFIC API 일별/주간/월간 집계 · 시각화 |
| 영화 AI 챗봇 | 시청 이력 반영 맥락 대화 |
| 보행 그래프 경로 | OSM 233,964 edges + 3축 환경 점수 최적 경로 |
| 소셜 로그인 | Google·Kakao·Naver OAuth 2.0, JWT, RBAC |
