---
layout: default
title: "Mova 멀티턴 주제 전환 버그 수정"
date: 2026-08-19
categories: [bugfix, mova]
---

# Mova 멀티턴 주제 전환 버그 수정

## 증상

1턴에서 "액션 영화 추천해줘" → 2턴에서 "여행 영화 추천해줘"로 주제를 전환하면 "새 후보 없음"이 반환되는 문제.

## 원인 분석

`IntentExtractionService.extract()`에서 이전 턴 컨텍스트가 현재 턴의 검색 필터와 RAG 쿼리에 오염되는 경로가 2개 있었다.

### 1차 오염: search_filters

```python
# 이전 턴("액션")이 composed_text에 포함
composed_text = _prepend_recent_user_context(text, history)

# composed_text가 결정론적 경로에 흘러가며 이전 장르가 잔존
filters = build_search_filters(composed_text)  # ← 오염
```

### 2차 오염: refined_query (RAG)

1차 수정 후에도 Gemini가 이전 턴 컨텍스트를 포함한 refined_query("송강호 공포 영화")를 반환하면, 이 값이 RAG 시맨틱 검색 쿼리로 직행하여 후보 자체가 오염됐다.

## 수정

- `composed_text`는 **Gemini EXTRACT_PROMPT에만** 사용
- `search_filters`, `keywords`는 **현재 턴(text)에서만** 유도
- `refined_query`도 Gemini 응답 대신 **결정론적 결과**에서 추출

## 추가 수정: 한국 영화 풀 필터 강화

초성 게임에서 `original_language='ko'`이지만 한국 배우가 없는 TMDB 오분류 영화가 혼입되는 문제를 발견. `_has_korean_actor()` EXISTS 서브쿼리를 추가하여 한국어 이름(`[가-힣]`) 배우가 한 명도 없는 영화를 제외했다.
