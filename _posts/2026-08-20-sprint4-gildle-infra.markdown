---
layout: default
title: "Sprint 4 — Gildle 인프라 + 보행 그래프"
date: 2026-08-20
categories: [sprint, gildle]
---

# Sprint 4 — Gildle 인프라 + 보행 그래프

## OSM 보행 그래프 구축

osmnx 2.x로 영등포구(여의도) 보행 네트워크를 추출하고, Clean Architecture 기반 Hexagonal 구조를 구축했다.

| 항목 | 수치 |
|------|------|
| 중심점 | 여의도 (37.528, 126.933) |
| 반경 | 1.5km |
| 노드 | 1,178개 |
| 엣지 | 3,260개 (중복 제거 후 1,616 undirected) |

### 데이터 파이프라인

```
OpenStreetMap (영등포구)
    │
    ▼
osmnx.graph_from_point(center, dist, network_type="walk")
    │
    ▼
NetworkX 그래프 → 엣지 리스트 + GraphML 캐시 (1.3MB)
    │
    ▼
EdgeScoreCalculator: 3축 환경 점수 산정
    │
    ├─ 가로수 CSV (cp949) → tree_score (0~1)
    ├─ 결빙 CSV (cp949) → hazard_score (0~1)
    └─ 도로 유형/폭원 → dog_friendly_score (0~1)
    │
    ▼
scored_edges.json (1,616 엣지 × 3축, 456KB)
```

## 환경 점수 산정 (EdgeScoreCalculator)

| 점수 | 엣지 수 | 비율 | 산정 방식 |
|------|---------|------|-----------|
| tree_score | 107/1,616 | 6.6% | 도로명 매칭 우선 → 좌표 근접 50m 폴백, 수종 비율 + 밀도 정규화 |
| hazard_score | 101/1,616 | 6.3% | 반경 내 거리 기반 선형 감쇠, 복수 겹침 시 max |
| dog_friendly | 1,616/1,616 | 100% | tree_score × 0.7 + 0.3 휴리스틱 |

## CSV → PostgreSQL 전환

Clean Architecture의 Port(ABC)를 유지한 채 Adapter만 교체하여 CSV → PostgreSQL로 전환했다.

- **도메인/애플리케이션 레이어 변경 0건** — Port 추상화의 실제 효과 입증
- `GILDLE_DB_MODE=csv|postgres` env로 CSV/Pg 전환 가능
- 마이그레이션: `route_edges` score 3컬럼 + `route_nodes` osm_id 추가
- 테스트 145 → 170건 (Pg 리포지토리 17건 + import 8건)

## 트러블슈팅

### CSV 인코딩 (cp949)

서울 열린데이터 CSV가 cp949 인코딩이라 Python 기본 utf-8로 읽으면 깨지는 문제. `encoding="cp949"` 명시로 해결, 통합 테스트에서 실제 CSV 파일로 검증.

### osmnx EC2 호환

osmnx + GDAL 의존성이 무거워 EC2에 미포함. lazy import로 전환하여 미설치 환경에서도 기동 가능하도록 했다.
