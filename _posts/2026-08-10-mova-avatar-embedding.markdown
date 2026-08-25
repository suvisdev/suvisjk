---
layout: default
title: "아바타 업로드 + HNSW 벡터 인덱스"
categories: [feature, mova]
---

# 아바타 업로드 + HNSW 벡터 인덱스

- S3 아바타 업로드 구현 (presigned URL, 프론트 크롭 UI)
- `EMBEDDING_BACKEND` 전제 오류 발견 및 수정
- hub_knowledge에 HNSW 벡터 인덱스 도입 — 검색 성능 향상
- nginx stale DNS 502 + mova 로그인 장애 대응 (auth 게이트웨이)
