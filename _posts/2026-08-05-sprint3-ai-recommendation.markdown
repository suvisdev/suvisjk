---
layout: default
title: "Sprint 3 — AI 추천 + 리뷰 파이프라인"
date: 2026-08-05
categories: [sprint, mova, ai]
---

# Sprint 3 — AI 추천 + 리뷰 파이프라인

## LoRA 파인튜닝 추천 엔진

EXAONE-2.4B AWQ 모델을 LoRA로 파인튜닝해 영화 추천 엔진을 구축했다.

| 항목 | 내용 |
|------|------|
| 모델 | EXAONE-2.4B AWQ (LG AI Research 한국어 특화) |
| 파인튜닝 | LoRA (Low-Rank Adaptation) |
| 서빙 | systemd 서비스 `:8200`, Cloudflare Tunnel로 EC2에 노출 |
| 폴백 | `RECOMMENDATION_BACKEND` env로 lora ↔ gemini 전환 (왕복 8초) |

### 듀얼 백엔드 아키텍처

```
사용자 취향 데이터
    │
    ├─ GPU 가용 → LoRA 서버 (EXAONE-2.4B AWQ, :8200)
    │                 │
    │                 └→ Cloudflare Tunnel → EC2 API
    │
    └─ GPU 미가용 → Gemini API (폴백)
                      │
                      └→ EC2 API
```

- EC2에는 GPU가 없으므로 기본적으로 Cloudflare Tunnel을 통해 집 GPU 서버의 LoRA 서버를 호출
- GPU 서버 다운 시 `RECOMMENDATION_BACKEND=gemini`로 전환 후 docker compose 재기동

## AI 리뷰 생성 파이프라인

KOBIS 박스오피스 데이터를 기반으로 크롤링 → AI 리뷰를 자동 생성하는 파이프라인을 구축했다.

```
KOBIS 일별 박스오피스 → 영화 제목 리스트
    │
    ├→ Google News 크롤링 (관련 기사)
    ├→ 한국어 위키피디아 크롤링 (줄거리·평가)
    └→ KOBIS 흥행 지표 (관객수·매출)
         │
         ▼
    Gemini → 평론가 톤 리뷰 (200~300자)
         │
         ▼
    Gemini → 긍부정 종합 별점 산정 (1.0~5.0)
         │
         ▼
    reviews 테이블 저장 (ai_reviewer 계정)
```

## 취향 벡터 재정렬

사용자 리뷰 기반 taste vector와 영화 embedding의 코사인 유사도로 추천 결과를 재정렬하는 기능을 구현했다.

- `UserTasteVectorRepositoryPort.get_taste_vector()` — 사용자 taste vector 조회
- `MoviesRepositoryPort.list_embeddings_by_ids()` — 영화 embedding 배치 조회
- ChatInteractor에서 LLM 추천 결과를 코사인 유사도로 재정렬
- 비로그인 / taste vector 없음 / embedding 없음 시 스킵 (graceful degradation)
