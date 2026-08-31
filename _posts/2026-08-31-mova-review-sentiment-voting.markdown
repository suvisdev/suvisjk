---
layout: default
title: "Mova 리뷰 감정분석 통합·유용성 투표"
date: 2026-08-31
categories: [feature, mova]
---

# Mova 리뷰 감정분석 통합 · 유용성 투표

## Echo 감정분석 → 리뷰 시스템 깊은 통합

EXAONE-3.5-2.4B LoRA 파인튜닝(NSMC 긍정/부정)으로 학습한 Echo 감정분석
모델을 리뷰 전체 파이프라인에 통합했다.

| 기능 | 설명 |
|------|------|
| 에디터 리뷰 자동 별점 | rating=NULL인 AI 에디터 리뷰에 감정 신뢰도 기반 0.5~5.0 자동 부여 |
| 취향 벡터 개선 | effective_rating = 별점 × (1 + 0.3 × 감정 alignment) — 감정과 별점이 일치하면 강화, 불일치면 감쇄 |
| 감정 요약 | 영화 상세 페이지에 긍정/부정 비율 바 + 한 줄 요약 텍스트 |
| 신뢰도 태그 | 에디터 리뷰에 뉴스 소스 수 기반 "AI 에디터 ·높음/보통/낮음" 배지 |
| 자동 스케줄러 | 24시간 주기 배치 — GPU 없는 환경에서는 자동 종료 |

### 별점 변환 공식

```
긍정: 2.5 + score × 2.5  →  score=0.9일 때 ★4.5
부정: 2.5 - score × 2.0  →  score=0.9일 때 ★1.0
```

0.5단위 반올림, 0.5~5.0 클램핑. 리뷰 작성 시 BackgroundTask로 자동 분석되고,
미처리 분은 24시간 배치 스케줄러가 안전망으로 잡는다.

## 리뷰 유용성 투표 (도움돼요)

리뷰 카드의 ThumbsUp 버튼을 실제 투표 기능으로 구현했다.

- `review_votes` 테이블: user_id + review_id UNIQUE(토글 방식)
- `POST /mova/reviews/{id}/vote`: 로그인 필수, 이미 투표했으면 취소
- 리뷰 목록 조회 시 vote_count를 서브쿼리로 함께 반환
- 프론트에서 클릭 즉시 카운트 반영

### 아키텍처

기존 헥사고날 레이어를 그대로 따랐다:

```
Router → UseCase(ABC) → Interactor → Repository(ABC) → PgRepository
         ↓ toggle_vote       ↓               ↓
    ReviewVoteResultDto   toggle_vote    INSERT/DELETE + COUNT
```

## DB 마이그레이션

| 리비전 | 내용 |
|--------|------|
| `20260831_0001` | reviews에 sentiment_label, sentiment_score 컬럼 추가 |
| `20260831_0002` | reviews에 news_source_count 컬럼 추가 |
| `20260831_0003` | review_votes 테이블 생성 |
