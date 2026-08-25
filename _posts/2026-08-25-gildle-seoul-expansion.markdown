---
layout: default
title: "Gildle 서울 전역 확장 + OSM 나무/공원 통합"
categories: [feature, gildle]
---

# Gildle 서울 전역 확장 + OSM 나무/공원 통합

- 보행 그래프 영등포구(1,616) → 서울 전체(233,964 edges) 확장
- Overpass API 서울 나무 6,851 · 공원 3,053 수집
- 격자 매칭으로 tree_score 18배 증가 (686→12,219)
- dog_friendly > 0.3 엣지 76,109개
- 줌 12~14 격자 샘플링 (233k→4.7k~42k) 서버 사이드 간소화
- 지도 UX: 점→선, 장소 검색(Nominatim), 뷰포트 동적 로딩
