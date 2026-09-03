---
layout: default
title: 부록
permalink: /appendix/
nav_order: 10
---

# 부록

## 용어 정의

| 용어 | 설명 |
|------|------|
| Mova | AI 영화 추천 서비스. Movie + Nova의 합성어 |
| Gildle | 반려견 산책 경로 추천 서비스. Guild + Idle의 합성어 |
| LoRA | Low-Rank Adaptation. 대형 언어모델을 적은 파라미터로 파인튜닝하는 기법 |
| EXAONE | LG AI Research의 한국어 특화 대형 언어모델 |
| AWQ | Activation-aware Weight Quantization. 모델 경량화 기법 |
| OSM | OpenStreetMap. 오픈소스 지도 데이터 |
| osmnx | OSM 데이터를 NetworkX 그래프로 변환하는 Python 라이브러리 |
| A* | 휴리스틱 기반 최단 경로 탐색 알고리즘 |
| Star Topology | Hub-and-Spoke 의존 구조. Hub만 공통 의존, Spoke 간 직접 의존 금지 |
| Clean Architecture | 의존 방향이 바깥→안쪽인 계층형 아키텍처 |
| Hexagonal | Ports & Adapters 패턴. 입출력을 추상화해 도메인 코어를 보호 |
| KOFIC | 영화진흥위원회. 한국 박스오피스 데이터 API 제공 |
| KOBIS | 영화관입장권통합전산망. KOFIC의 박스오피스 시스템 |
| TMDB | The Movie Database. 영화 메타데이터·포스터 API |
| DDD | Domain-Driven Design. Gildle에 적용한 도메인 중심 설계 |
| IDOR | Insecure Direct Object Reference. 인가 우회 취약점 |

---

## 관련 서식

| 서식 | 설명 |
|------|------|
| Conventional Commits | `feat:`, `fix:`, `docs:`, `refactor:` 접두사 커밋 메시지 |
| CLAUDE.md | 코딩 에이전트 인수인계 문서. 스택별로 작성 |
| WORK_LOG.md | 일별 작업 일지 |
| ADR | Architecture Decision Record. 주요 설계 결정 기록 |
