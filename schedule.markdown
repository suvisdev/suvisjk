---
layout: default
title: 개발 일정
permalink: /schedule/
nav_order: 7
---

# 개발 일정 및 추진 체계

## 단계별 개발 일정

### Sprint 1 — 기반 구축

| 예정 | 진행중 | 완료 |
|------|--------|------|
| | | ✅ FastAPI + Clean Architecture 기반 세팅 |
| | | ✅ PostgreSQL (Neon) + SQLAlchemy 2.0 Async |
| | | ✅ Docker Compose 로컬 환경 |
| | | ✅ Next.js 16 프론트엔드 초기 구조 |
| | | ✅ JWT 인증 + OAuth 2.0 (Google·Kakao·Naver) |

### Sprint 2 — Mova 코어

| 예정 | 진행중 | 완료 |
|------|--------|------|
| | | ✅ TMDB 영화 데이터 import (200+ 편) |
| | | ✅ 박스오피스 랭킹 (KOFIC API) |
| | | ✅ 영화 상세·검색·컬렉션·찜 |
| | | ✅ Gemini 영화 챗봇 |
| | | ✅ 개봉 예정작 (TMDB) |

### Sprint 3 — AI 추천 + 리뷰

| 예정 | 진행중 | 완료 |
|------|--------|------|
| | | ✅ EXAONE-2.4B AWQ LoRA 파인튜닝 |
| | | ✅ LoRA 서버 (systemd + Cloudflare Tunnel) |
| | | ✅ Gemini 폴백 듀얼 백엔드 |
| | | ✅ 크롤링 파이프라인 (KOBIS·뉴스·위키) |
| | | ✅ AI 리뷰 자동 생성 (Gemini) |

### Sprint 4 — Gildle + 보안

| 예정 | 진행중 | 완료 |
|------|--------|------|
| | | ✅ OSM 보행 그래프 추출 (osmnx) |
| | | ✅ 환경 점수 배치 산정 (가로수·결빙) |
| | | ✅ A* 경로 추천 + 시즌 모드 |
| | | ✅ Leaflet 지도 시각화 |
| | | ✅ IDOR 취약점 5건 전수 수정 |

### Sprint 5 — 고도화

| 예정 | 진행중 | 완료 |
|------|--------|------|
| 지역 확장 (영등포구 → 서울 전역) | | ✅ 구간별 점수 색상 시각화 |
| Flutter 모바일 앱 완성 | | ✅ 경로 요약 패널 (평균 점수) |
| EC2 Docker 이미지 통합 | | ✅ AI 리뷰 별점 긍부정 기반 산정 |

---

## 위험 관리 방안

| 위험 요소 | 영향 | 대응 |
|-----------|------|------|
| GPU 서버 다운 | 추천 서비스 중단 | `RECOMMENDATION_BACKEND` env로 lora↔gemini 8초 폴백 |
| EC2 디스크 부족 | 빌드 실패 | `docker builder prune -a`, 순차 빌드, 이미지 통합 백로그 |
| Docker .env 누락 | DB 전면 장애 | `--env-file suvisdev/.env` 필수 규칙 등록 |
| 외부 API 제한 | 크롤링 실패 | 위키피디아 429 대응 재시도 + 부분 수집 허용 |
| VRAM 경합 | LoRA 서버 + 학습 동시 불가 | lora-server stop → 학습 → start 워크플로 |
