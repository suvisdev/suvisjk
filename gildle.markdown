---
layout: default
title: Gildle — 산책 경로 추천
permalink: /gildle/
nav_order: 5
---

# Gildle — 반려견 산책 경로 추천

> OSM 보행 그래프에 가로수(나무 그늘)·결빙 위험·반려견 친화도 3축 환경 점수를 매긴 뒤, A* 알고리즘으로 시즌별 최적 경로를 계산하는 산책 경로 추천 서비스.

---

## 주요 기능

| 기능 | 설명 |
|------|------|
| 보행 그래프 시각화 | osmnx로 추출한 영등포구 도보 네트워크 1,600+ 엣지를 Leaflet 지도에 인터랙티브 시각화. |
| 환경 점수 산정 | 서울 열린데이터(가로수·결빙)와 반려견 친화도를 3축으로 매핑. 엣지별 배치 점수 산정. |
| 경로 추천 | 출발/도착 클릭 → A* 알고리즘으로 환경 점수 가중 최적 경로 계산. |
| 시즌 모드 | 봄/가을(그늘 우선)·겨울(결빙 회피) 모드에 따라 가중치 자동 전환. |
| 구간별 시각화 | 경로를 선택한 레이어(나무/결빙/반려견) 점수별 색상으로 구간 표시. |
| 경로 요약 | 총 거리·도보 시간 + 나무·결빙·반려견 평균 점수 요약 패널. |
| 레이어 토글 | 나무 그늘·결빙 위험·반려견 친화도 3개 점수 레이어 전환. |

---

## 기능 요구 사항

| ID | 기능 | 설명 | 우선순위 |
|----|------|------|----------|
| G-001 | 보행 그래프 | osmnx로 OSM 도보 네트워크 추출, PostgreSQL 저장 | 필수 |
| G-002 | 환경 점수 | 가로수·결빙·반려견 3축 점수 배치 산정 (서울 열린데이터) | 필수 |
| G-003 | 경로 추천 | A* 알고리즘 기반 최적 경로 계산 | 필수 |
| G-004 | 시즌 모드 | 봄가을(그늘 우선)/겨울(결빙 회피) 가중치 전환 | 필수 |
| G-005 | 구간 시각화 | 점수별 색상 코딩 + 평균 점수 요약 패널 | 필수 |
| G-006 | 지도 UI | Leaflet 인터랙티브 지도, 클릭 출발/도착, 레이어 토글 | 필수 |
| G-007 | 지역 확장 | 영등포구 → 서울 전역 (백로그) | 선택 |

---

## 데이터 파이프라인

### OSM 보행 그래프 구축

```
OpenStreetMap (영등포구)
    │
    ▼
osmnx.graph_from_place("영등포구", network_type="walk")
    │
    ▼
NetworkX 그래프 → 엣지 리스트 추출
    │
    ▼
PostgreSQL route_edges 테이블
```

### 환경 점수 배치 산정

```
서울 열린데이터
    ├─ 가로수 현황 CSV (cp949) ──→ 위치별 나무 밀도 ──→ tree_score (0~1)
    └─ 도로결빙 취약지점 CSV    ──→ 위치별 결빙 위험 ──→ hazard_score (0~1)

반려견 기본 점수 ──→ 도로 유형·폭원 기반 ──→ dog_friendly_score (0~1)

    │
    ▼
EdgeScoreCalculator: 엣지 midpoint ↔ 데이터 위치 매칭 (반경 검색)
    │
    ▼
scored_edges.json (1,616 엣지 × 3축 점수)
```

### 경로 최적화

```
출발 노드 + 도착 노드 + 시즌 모드
    │
    ▼
A* 알고리즘
    │
    ├─ 봄/가을: cost = distance × (1 - tree_score × 0.3)
    └─ 겨울:    cost = distance × (1 + hazard_score × 0.5)
    │
    ▼
최적 경로 (노드 리스트 + 좌표)
    │
    ▼
프론트: 구간별 점수 색상 Polyline + 요약 패널
```

---

## 점수 분포 (영등포구 실측)

| 점수 | 엣지 수 | 비율 | 범위 | 평균 |
|------|---------|------|------|------|
| 나무 그늘 (tree) | 107 / 1,616 | 6.6% | 0.12 ~ 0.84 | 0.75 |
| 결빙 위험 (hazard) | 101 / 1,616 | 6.3% | 0.01 ~ 0.97 | 0.35 |
| 반려견 친화 (dog) | 1,616 / 1,616 | 100% | 0.30 ~ 0.89 | 0.34 |

---

## 기술 스택

| 구분 | 기술 |
|------|------|
| 백엔드 | Python 3.12 · FastAPI · PostgreSQL · Clean Architecture + DDD |
| 프론트엔드 | Next.js 16 · TypeScript · Leaflet · react-leaflet |
| 데이터 | OpenStreetMap · osmnx · 서울 열린데이터 (가로수·결빙) |
| 알고리즘 | A* (가중 최단 경로) · NetworkX |
| 테스트 | pytest (170+ 테스트, 레이어별 단위·통합) |

---

## 아키텍처 (Clean Architecture + DDD)

```
adapter/
├── inbound/api/v1/route_router.py     ← FastAPI Router
└── outbound/
    ├── graph/sample_walk_graph_source.py  ← OSM 그래프 로더
    └── repositories/                      ← PostgreSQL 저장소
app/
├── ports/input/calculate_route_use_case.py   ← ABC
├── ports/output/                              ← Repository ABC
└── use_cases/                                 ← Interactor
domain/
└── value_objects/
    ├── route_edge.py      ← 엣지 (from/to/점수/거리)
    ├── coordinate.py      ← 위경도
    └── season_mode.py     ← 시즌 모드 (spring_autumn/winter_safety)
```

---

## 트러블슈팅

### CSV 인코딩 통합 (cp949)

- **상황:** 서울 열린데이터 CSV가 cp949 인코딩, Python 기본 utf-8로 읽으면 깨짐
- **원인:** 공공데이터 포털이 Excel 호환 인코딩(EUC-KR 계열)으로 제공
- **해결:** `encoding="cp949"` 명시, 통합 테스트에서 실제 CSV 파일로 검증
- **결과:** 가로수·결빙 데이터 정상 파싱, 영등포구 전체 매칭 완료

### OSM 데이터 EC2 호환

- **상황:** osmnx가 EC2에 미설치 상태에서 import 시 기동 실패
- **원인:** osmnx + 의존성(GDAL 등)이 무거워 EC2 이미지에 미포함
- **해결:** osmnx lazy import로 전환, 미설치 환경에서도 기동 가능
- **결과:** EC2 배포 시 Gildle 외 서비스 영향 없음

---

## 스크린샷

{% for s in site.data.project.screenshots %}
{% if s.caption contains 'Gildle' %}
<figure>
  <img src="{{ s.path | relative_url }}" alt="{{ s.caption }}" style="max-width:100%">
  <figcaption>{{ s.caption }}</figcaption>
</figure>
{% endif %}
{% endfor %}
