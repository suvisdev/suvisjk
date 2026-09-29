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

### 채팅 판단 에이전트 (v9, 2026-09-29)

```
발화 + 최근 대화
    │
    ▼
MovaChatAgent ── 판단 모델(EXAONE 3.5 2.4B LoRA, GGUF, Ollama) ──→ 다음 행동 하나
    │            <tool_call>{"name":"get_movie_details","arguments":{"title":"…"}}</tool_call> | FINAL
    ├─ 데이터 도구: search_movie · get_movie_details · now_showing   → 결과를 붙여 재판단(≤3단계)
    └─ 터미널 도구: recommend_movies · showtimes · where_to_watch     → 기존 추천·예매 트랙 실행
    │
    ▼
답변: 사실(출연진·시간표·상영작·OTT)은 코드 템플릿 · 문장은 추천 이유·리뷰 요약만 LLM
```

- 왜 바꿨나: 6칸 슬롯(intent·title·region·time·chain·followup)에 자리가 없는 질문(출연진, 직전
  카드 전체, 현재 상영작)이 전부 오답이었고, 발화에 없는 제목을 지어내는 일이 있었다.
- 학습 데이터는 템플릿 합성 1,600행(교사 비용 0원). 평가셋 66(실사용 대화 17 포함)에서
  **v9 62 · 운영 7.8B 59 · 학습 전 2.4B 55**. 코랩(fp16)보다 노트북 GGUF 채점이 높게 나오는
  일이 반복돼 내보내기 판단은 GGUF 재채점 기준.
- 가드는 학습이 아니라 코드: 제목·지역 근거, 반복 호출 차단, 호출 예산.

### 추천 엔진 — LoRA + Gemini 폴백

```
추천 요청 (에이전트 recommend_movies)
    │
    ▼
RAG 후보 (pgvector HNSW, hub_knowledge) + 태그 실매칭 → 품질 하한·시리즈당 1편
    │
    ├─ lora-server :8200 (EXAONE 3.5 2.4B LoRA, llama.cpp GGUF Q5_K_M, 노트북 GPU)
    └─ 실패·서킷 오픈 시 Gemini API 폴백 (자동, 60초 쿨다운 후 복귀)
```

- 학습은 Colab(L4), 어댑터 병합 → GGUF 양자화까지 노트북에서 완결. 운영 회귀 하네스
  28질의·멀티턴 17장면으로 매 회차 전후 비교.
- Gemini 블라인드 비교(2026-09-29): 추천 픽 27건 중 Gemini 12·EXAONE 3·동률 12 — 패인은
  "3편을 채우려 주제 밖 작품을 끼움". 다음 학습 회차의 목표다.

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
| 백엔드 | Python 3.12 · FastAPI · PostgreSQL 16 + pgvector · Redis · SQLAlchemy 2.0 Async |
| 프론트엔드 | Next.js 16 · TypeScript 5.7 · React 19 · Tailwind CSS 4 · shadcn/ui |
| 모바일 | Flutter · Dart |
| AI/ML | EXAONE 3.5 2.4B LoRA(추천·판단 에이전트) · EXAONE 3.5 7.8B(이해·잡담) · Google Gemini(폴백·리뷰 요약) · pgvector RAG |
| 데이터 | TMDB API · KOFIC/KOBIS API · 카카오 로컬(영화관) · 롯데시네마 시간표 · Google News · 위키피디아 |
| 인프라 | 노트북 k3s · Docker(DB·Redis) · llama.cpp · Ollama · Cloudflare Tunnel · AWS S3 |

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
