---
layout: default
title: Mova — AI 영화 추천
permalink: /mova/
nav_order: 4
---

# Mova — AI 영화 추천 플랫폼

> LoRA 파인튜닝 EXAONE-2.4B와 Gemini API 듀얼 백엔드로 개인화 영화 추천, AI 리뷰 자동 생성, 실시간 박스오피스 랭킹, AI 챗봇, 개봉 예정작 알림을 제공하는 영화 플랫폼.

---

## 주요 기능

| 기능 | 설명 |
|------|------|
| AI 맞춤 추천 | LoRA 파인튜닝 EXAONE-2.4B와 Gemini API 듀얼 백엔드. 사용자 취향 학습 기반 개인화 추천, GPU 미가용 시 Gemini 자동 폴백(전환 8초). |
| AI 리뷰 생성 | KOBIS 박스오피스·Google 뉴스·위키피디아 자동 크롤링 → Gemini가 평론가 톤 리뷰 작성. 수집 자료의 긍부정 반응을 종합해 별점 산정. |
| 박스오피스 랭킹 | KOFIC API로 일별·주간·월간 한국 박스오피스 집계. 포스터·관객수·매출 시각화. |
| 개봉 예정작 | TMDB API로 한국 개봉 예정 영화 자동 갱신, 월별 그룹핑. |
| 영화 AI 챗봇 | Gemini 기반 영화 추천·정보 대화. 사용자 시청 이력과 취향을 반영한 맥락 대화. |
| 컬렉션·찜 | 개인 영화 컬렉션 생성·관리, 찜 목록 토글. |
| 마이페이지 | 시청 기록, 리뷰 목록, 프로필 관리. |

---

## 기능 요구 사항

| ID | 기능 | 설명 | 우선순위 |
|----|------|------|----------|
| M-001 | AI 추천 | LoRA/Gemini 듀얼 백엔드 개인화 추천, 폴백 전환 | 필수 |
| M-002 | AI 리뷰 | KOBIS·뉴스·위키 크롤링 → Gemini 리뷰 생성 + 별점 산정 | 필수 |
| M-003 | 랭킹 | KOFIC 박스오피스 일별/주간/월간 집계·시각화 | 필수 |
| M-004 | 챗봇 | Gemini 영화 추천·정보 맥락 대화 | 필수 |
| M-005 | 개봉 예정작 | TMDB API 자동 갱신, 월별 그룹핑 | 선택 |
| M-006 | 컬렉션·찜 | 개인 영화 목록 생성·관리, 찜 토글 | 필수 |
| M-007 | 마이페이지 | 시청 기록, 리뷰 목록, 프로필 관리 | 필수 |
| M-008 | 소셜 로그인 | Google·Kakao·Naver OAuth 2.0 통합 인증 | 필수 |

---

## AI 파이프라인

### 추천 엔진 — 듀얼 백엔드

```
사용자 취향 데이터
    │
    ├─ GPU 가용 ──→ LoRA 서버 (EXAONE-2.4B AWQ, :8200)
    │                    │
    │                    └─→ Cloudflare Tunnel ──→ EC2 API
    │
    └─ GPU 미가용 ──→ Gemini API (폴백)
                         │
                         └─→ EC2 API
```

- `RECOMMENDATION_BACKEND` 환경변수로 `lora` ↔ `gemini` 전환
- 전환 왕복 약 8초 (docker compose 재기동)
- LoRA 서버: systemd 서비스, WSL2 GPU (RTX)

### AI 리뷰 생성 파이프라인

```
KOBIS 일별 박스오피스 ──→ 영화 제목 리스트
    │
    ├─→ Google News 크롤링 (관련 기사 수집)
    ├─→ 한국어 위키피디아 크롤링 (줄거리·평가·인포박스)
    └─→ KOBIS 흥행 지표 (관객수·매출)
         │
         ▼
    JSONL 통합 자료
         │
         ▼
    Gemini ──→ 평론가 톤 리뷰 (200~300자)
         │
         ▼
    Gemini ──→ 긍부정 종합 별점 산정 (1.0~5.0, 0.5 단위)
         │
         ▼
    reviews 테이블 저장 (ai_reviewer 계정)
```

---

## 기술 스택

| 구분 | 기술 |
|------|------|
| 백엔드 | Python 3.12 · FastAPI · PostgreSQL (Neon) · Redis · SQLAlchemy 2.0 Async |
| 프론트엔드 | Next.js 16 · TypeScript 5.7 · React 19 · Tailwind CSS 4 · shadcn/ui |
| 모바일 | Flutter · Dart |
| AI/ML | EXAONE-2.4B AWQ (LoRA) · Google Gemini API |
| 데이터 | TMDB API · KOFIC/KOBIS API · Google News · 위키피디아 |
| 인프라 | Docker Compose · AWS EC2 · AWS S3 · Cloudflare Tunnel |

---

## 트러블슈팅

### LoRA 서버 ↔ Gemini 듀얼 백엔드 폴백

- **상황:** GPU 노트북(LoRA 서버)이 꺼지거나 Cloudflare Tunnel이 끊기면 추천 API 전면 장애
- **원인:** EXAONE-2.4B AWQ 모델이 로컬 GPU에서만 서빙 가능, EC2에는 GPU 없음
- **해결:** `RECOMMENDATION_BACKEND` 환경변수로 lora/gemini 전환, docker compose 재기동 한 번으로 폴백(왕복 8초)
- **결과:** GPU 미가용 시에도 추천 서비스 무중단 제공

### IDOR 취약점 5건 전수 수정

- **상황:** user_id를 경로·바디 파라미터로 받아 다른 사용자의 리소스에 접근 가능
- **원인:** 초기 개발 시 user_id를 클라이언트 입력으로 신뢰, 토큰 기반 신원 확인 누락
- **해결:** 전수 조사로 5건(mypage·profile·watchlist·picks·chat) 식별 후 전부 JWT 토큰 기반으로 전환
- **결과:** 모든 리소스 접근에 소유권 검증 적용, IDOR 취약점 0건

---

## 스크린샷

{% for s in site.data.project.screenshots %}
{% if s.caption contains 'Mova' %}
<figure>
  <img src="{{ s.path | relative_url }}" alt="{{ s.caption }}" style="max-width:100%">
  <figcaption>{{ s.caption }}</figcaption>
</figure>
{% endif %}
{% endfor %}
