---
layout: default
title: "Sprint 2 — Mova 코어 기능 구현"
date: 2026-08-02
categories: [sprint, mova]
---

# Sprint 2 — Mova 코어 기능 구현

## 작업 요약

Mova 영화 플랫폼의 핵심 기능을 구현했다.

| 기능 | 상태 | 비고 |
|------|------|------|
| TMDB 영화 데이터 import | ✅ 완료 | 200+ 편 초기 적재 |
| 박스오피스 랭킹 | ✅ 완료 | KOFIC API 일별/주간/월간 |
| 영화 상세·검색·컬렉션·찜 | ✅ 완료 | - |
| Gemini 영화 챗봇 | ✅ 완료 | - |
| 개봉 예정작 | ✅ 완료 | TMDB API 월별 그룹핑 |

## 대량 수집 파이프라인 구축

`bulk_import_movies.py`를 통해 TMDB + KOFIC 합산 수만 편을 점진 적재하는 배치 파이프라인을 구축했다.

```
TMDB discover/popular API
    │
    ├─→ upsert_movie (slug 기준 멱등)
    ├─→ credits 백필 (배우·감독)
    └─→ hub_knowledge 인제스트 (RAG 벡터)
         │
KOFIC searchMovieList API
    │
    ├─→ upsert_movie
    └─→ hub_knowledge 인제스트
```

- `--source tmdb_popular|tmdb_discover|kofic` 소스 선택
- `--start-page N`으로 재시작 지원
- 영화당 실패는 해당 건만 스킵 (세션 격리)

## 트러블슈팅: PendingRollbackError 도미노

대량 수집 첫 실행에서 `characters.character_name VARCHAR(50)` 초과로 1건이 실패한 후, 이후 418건이 연쇄 실패하는 문제를 발견했다.

- **원인**: SQLAlchemy AsyncSession은 flush 실패 시 세션을 pending-rollback 상태로 남김. 명시적 `session.rollback()` 없이는 같은 세션의 이후 모든 쿼리가 즉시 실패.
- **해결**: 각 `except` 블록에 `await session.rollback()` 추가, 실패를 영화 한 편으로 격리.
- **검증**: 회귀 테스트 3건으로 도미노 재현 방지 확인.

## 로컬 DB 환경 구축

`docker compose --env-file suvisdev/.env`로 로컬 개발 DB를 구축하고 전체 파이프라인을 실행했다.

- `alembic upgrade head`로 36개 테이블 생성
- credits 백필: actors 389 / characters 371 / movie_directors 40
- hub_knowledge 인제스트: 39편 RAG 벡터 적재
