---
layout: default
title: 부록
permalink: /appendix/
nav_order: 9
---

# 부록

## 용어 정의

| 용어 | 설명 |
|------|------|
| Mova | AI 영화 추천 서비스. Movie + Nova의 합성어 |
| Gildle | 반려견 산책 경로 추천 서비스. Guild + Idle의 합성어 |
| LoRA | Low-Rank Adaptation. 대형 언어모델을 적은 파라미터로 파인튜닝하는 기법 |
| EXAONE | LG AI Research의 한국어 특화 대형 언어모델 |
| AWQ | Activation-aware Weight Quantization. 초기에 쓴 양자화 방식 |
| GGUF | llama.cpp의 모델 파일 형식. 현재 서빙 형식(Q5_K_M 양자화) |
| 에이전트 | 도구 목록 중 다음 행동을 모델이 고르는 구조. 앱마다 하나 |
| RAG | 검색 증강 생성. pgvector HNSW로 후보를 찾아 모델에 준다 |
| BFF | Backend For Frontend. 프론트 서버가 쿠키를 읽어 백엔드 호출을 대행하는 구조 |
| k3s | 경량 쿠버네티스. 노트북 프로덕션의 앱 파드 실행 환경 |
| OSM | OpenStreetMap. 오픈소스 지도 데이터 |
| osmnx | OSM 데이터를 NetworkX 그래프로 변환하는 Python 라이브러리 |
| A* | 휴리스틱 기반 최단 경로 탐색 알고리즘 |
| Star Topology | Hub-and-Spoke 의존 구조. Hub만 공통 의존, Spoke 간 직접 의존 금지 |
| Clean Architecture | 의존 방향이 바깥→안쪽인 계층형 아키텍처 |
| Hexagonal | Ports & Adapters 패턴. 입출력을 추상화해 도메인 코어를 보호 |
| KOFIC | 영화진흥위원회. 한국 박스오피스 데이터 API 제공 |
| KOBIS | 영화관입장권통합전산망. KOFIC의 박스오피스 시스템 |
| TMDB | The Movie Database. 영화 메타데이터·포스터 API |
| DDD | Domain-Driven Design. 규칙이 중복되거나 검증이 필요한 곳에만 선택 적용 |
| IDOR | Insecure Direct Object Reference. 인가 우회 취약점 |

---

## 관련 서식

| 서식 | 설명 |
|------|------|
| Conventional Commits | `feat:`, `fix:`, `docs:`, `refactor:` 접두사 커밋 메시지 |
| CLAUDE.md | 코딩 에이전트 인수인계 문서. 스택별로 작성 |
| WORK_LOG_*.md | 영역별(Mova·Gildle·그 외) 일별 작업 일지 |
| LESSONS.md | 실패 → 원인 → 극복을 한 줄씩 모은 압축 기록 |
| INTERVIEW_QUESTIONS.md | 그날 작업을 스스로 설명하기 위한 문답 |
| ADR | Architecture Decision Record. 주요 설계 결정 기록 |
