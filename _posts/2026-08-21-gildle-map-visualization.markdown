---
layout: default
title: "Gildle 지도 시각화 + 경로 추천 UI"
date: 2026-08-21
categories: [feature, gildle]
---

# Gildle 지도 시각화 + 경로 추천 UI

## Leaflet 보행 그래프 시각화

scored_edges.json의 1,616개 엣지를 Leaflet 지도 위에 인터랙티브하게 시각화하는 페이지를 구현했다.

### 구현 내용

| 항목 | 내용 |
|------|------|
| 지도 라이브러리 | react-leaflet 5.0 + Leaflet 1.9 |
| 타일 | CartoDB dark |
| 레이어 | 나무 그늘 / 결빙 위험 / 반려견 친화도 3개 탭 |
| 시각화 | CircleMarker at midpoint, 점수별 색상 4단계 |
| 상호작용 | 도로명 + 3축 점수 Tooltip, 자동 경계 맞춤 |

### 구간별 색상 코딩

경로 추천 결과를 선택한 레이어의 점수에 따라 구간별로 색상 코딩한다.

```
점수 높음 ████ 진한색 (나무: 초록, 결빙: 빨강, 반려견: 파랑)
점수 중간 ████ 중간색
점수 낮음 ████ 연한색
점수 0    ████ 회색
미산정    ████ 노란색 (yellow fallback)
```

## 경로 추천 UI

출발/도착 클릭 → A* 알고리즘으로 최적 경로를 계산하고, 구간별 점수에 따른 색상 Polyline과 요약 패널을 표시한다.

### 경로 요약 패널

```
┌─────────────────────────────────────────┐
│  총 거리: 1.2km  │  도보 시간: 약 15분   │
│  🌳 나무 0.72  │  ❄️ 결빙 0.15  │  🐕 반려견 0.68  │
└─────────────────────────────────────────┘
```

### RouteSegment 타입

```typescript
type RouteSegment = {
  from: [number, number]
  to: [number, number]
  edge: ScoredEdge | null
}
```

- `edgeLookup` Map으로 O(1) 엣지→점수 매칭
- 경로를 black outline Polyline (weight 8) + 구간별 색상 Polyline (weight 5)로 렌더링
- 시즌 모드(봄가을/겨울)에 따라 A* 가중치 자동 전환

## 백엔드 API

| 엔드포인트 | 역할 |
|-----------|------|
| `GET /api/gildle/graph-edges` | 1,616개 scored edges JSON 반환 |
| `POST /api/gildle/navigate` | A* 최적 경로 계산 (path + coordinates) |
