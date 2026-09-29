---
layout: default
title: 프로젝트 개요
permalink: /about/
nav_order: 2
---

# 프로젝트 개요

## 프로젝트 정보

| 항목 | 내용 |
|------|------|
| 프로젝트명 | **suvisdev** — AI 영화 추천(Mova) · 반려견 산책 경로(Gildle) 통합 플랫폼 |
| 개발 기간 | 2026.07 ~ 2026.09 (약 10주) |
| 개발 인원 | 1명 (개인 프로젝트) |
| 역할 | 풀스택 개발 · ML 파이프라인 · 인프라 · 데이터 수집 |
| 레포지토리 | [github.com/suvisdev/suvisdev.cloud](https://github.com/suvisdev/suvisdev.cloud) |
| 데모 | [suvisdev.cloud](https://suvisdev.cloud) |

---

## 프로젝트 배경

**영화 추천** — 수백 편의 신작 속에서 취향에 맞는 영화를 고르는 데 시간이 든다. 기존 협업 필터링은 콜드 스타트 문제가 있고, 리뷰는 사람이 직접 써야 한다. Mova는 LoRA 파인튜닝 모델로 개인화 추천을 제공하고, 크롤링 데이터를 Gemini가 분석해 AI 리뷰와 별점을 자동 생성한다.

**산책 경로** — 반려견과 산책할 때 그늘·결빙·반려견 출입 가능 여부를 고려한 경로를 찾기 어렵다. Gildle은 OSM 보행 그래프에 환경 점수를 매겨 계절·상황별 최적 산책 경로를 계산한다.

---

## 서브 프로젝트

### [Mova — AI 영화 추천](/mova/)

LoRA 파인튜닝 EXAONE-2.4B와 Gemini API 듀얼 백엔드로 개인화 영화 추천, AI 리뷰 자동 생성, 실시간 박스오피스 랭킹, AI 챗봇, 개봉 예정작 알림을 제공하는 영화 플랫폼.

| 핵심 기능 | 구현 |
|-----------|------|
| AI 추천 | LoRA 파인튜닝(EXAONE-2.4B AWQ) + Gemini 폴백 듀얼 백엔드, 취향 벡터 코사인 재정렬 |
| AI 리뷰 | KOBIS·Google News·위키피디아 크롤링 → Gemini 평론가 톤 리뷰 + 긍부정 별점 자동 산정 |
| 랭킹 | KOFIC API 일별/주간/월간 박스오피스 집계, 포스터·관객수·매출 시각화 |
| 챗봇 | Gemini 기반 영화 추천·정보 맥락 대화, 사용자 시청 이력 반영 |
| 대량 수집 | TMDB + KOFIC 합산 배치 파이프라인, 영화당 upsert → credits 백필 → RAG 인제스트 |

### [Gildle — 반려견 산책 경로](/gildle/)

OSM 보행 그래프에 가로수(나무 그늘)·결빙 위험·반려견 친화도 3축 환경 점수를 매긴 뒤, A* 알고리즘으로 시즌별 최적 경로를 계산하는 산책 경로 추천 서비스.

| 핵심 기능 | 구현 |
|-----------|------|
| 보행 그래프 | osmnx로 영등포구 도보 네트워크 추출 (1,178 노드 / 1,616 엣지) |
| 환경 점수 | 서울 열린데이터(가로수·결빙 CSV, cp949) + 반려견 친화도 3축 배치 산정 |
| 경로 추천 | A* 알고리즘, 시즌 모드(봄가을: 그늘 우선 / 겨울: 결빙 회피) 가중치 |
| 시각화 | Leaflet 인터랙티브 지도, 구간별 점수 색상 코딩 + 경로 요약 패널 |

---

## 기술 스택

| 구분 | 기술 |
|------|------|
| **백엔드** | Python 3.12 · FastAPI · PostgreSQL 16 + pgvector · Redis · SQLAlchemy 2.0 Async · Alembic |
| **프론트엔드** | Next.js 16 · TypeScript 5.7 · React 19 · Tailwind CSS 4 · shadcn/ui · 네이버 지도 JS v3 |
| **모바일** | Flutter · Dart · 카카오 로그인 SDK · 네이버 지도 SDK |
| **AI/ML** | EXAONE 3.5 2.4B LoRA(추천 GGUF Q5_K_M · 판단 에이전트 v9) · EXAONE 3.5 7.8B(이해·홈 채팅) · Google Gemini(폴백·리뷰 요약) · pgvector HNSW RAG |
| **데이터** | TMDB API · KOFIC/KOBIS API · 카카오 로컬 API · 브이월드 건물 · OpenStreetMap / osmnx · SRTM 고도 |
| **인프라** | 노트북 온프레미스 k3s(backend·auth·cloudflared 파드) · Docker(db·redis) · llama.cpp lora-server · Ollama · Cloudflare Tunnel · Vercel · AWS S3 |

---

## 아키텍처

### 시스템 구성도

```
┌──────────────────────────────────────────────────────────┐
│                      클라이언트                            │
│                                                          │
│   Next.js 16 (Vercel)            Flutter (Android/iOS)   │
│        │                              │                  │
│        └─────────── HTTPS ────────────┘                  │
│                       │                                  │
├───────────────────────┼──────────────────────────────────┤
│      Cloudflare Tunnel (api. / auth.suvisdev.cloud)       │
│                       │                                  │
│   ─────────── 노트북 온프레미스 (k3s) ───────────          │
│           ┌───────────┼───────────┐                      │
│           ▼           ▼           ▼                      │
│      backend 파드  auth 파드  Traefik 인그레스             │
│           │                                              │
├───────────┼──────────────────────────────────────────────┤
│      FastAPI  (모듈러 모놀리식 · Star Topology)            │
│                                                          │
│         mova   gildle   viewer   analytics   media        │
│            \      |       /         /       (Spoke)       │
│             ★   ontology   ★                (Hub)         │
│            /      |       \                              │
│      dispatch  execsuite  titanic  contents               │
│                                                          │
│      ─────────── core.* (공유 인프라) ──────────           │
│      security · matrix(DB·S3) · lol(OllamaClient·LoRA)    │
├──────────────────────────────────────────────────────────┤
│  PostgreSQL 16 + pgvector · Redis (도커)   S3 (AWS)        │
│  lora-server :8200 (llama.cpp GGUF, GPU)  Ollama :11434   │
└──────────────────────────────────────────────────────────┘
```

프로덕션은 **노트북 한 대의 온프레미스**다(2026-09-07 k3s 컷오버). 백엔드(CPU)와 GPU 추론
(lora-server·Ollama)이 같은 기계 안에서 통신하고, 외부 요청은 Cloudflare Tunnel이 노트북까지
가져온다. 클라우드 분리안은 검토 뒤 기각했다 — 백엔드가 유휴에도 2GB RAM이라 무료 인스턴스에
들어가지 않고, 백엔드↔GPU 지연이 늘어난다(2026-09-29).

### 모듈러 모놀리식 · Star Topology

단일 배포 단위(FastAPI)에서 앱 간 경계를 명확히 분리하는 구조다.

- **Hub (ontology)** — 공통 크롤링·AI 파이프라인·이벤트 버스. Spoke를 import하지 않는다.
- **Spoke (mova·gildle·viewer 등)** — 각 도메인 로직. Hub만 import 가능, Spoke 간 직접 import 금지.
- **`import-linter`** — 커밋 시 의존 방향을 자동 검사한다.

| 의존 방향 | 허용 |
|-----------|------|
| Spoke → Hub (ontology) | ✅ |
| Spoke → Spoke (직접) | ❌ 금지 |
| Hub → Spoke | ❌ 금지 |
| Spoke · Hub → core.* | ✅ |

### 두뇌 층 — 오케스트레이터 · 에이전트 · 도구 · 클라이언트 (2026-09-29)

LLM이 판단하는 자리에 이름을 네 층으로 고정했다. "오케스트레이터"는 전체 두뇌 하나에만 쓴다.

```
오케스트레이터 (Orchestrator)   ← 전체 두뇌 · 허브(ontology)에 하나 (예정)
 └ 에이전트 (Agent)             ← 앱 두뇌 · 앱마다 하나 — MovaChatAgent(v9) · GildleWalkAgent(예정)
    └ 도구 (Tool)               ← 판단 없이 정해진 일 — search_movie · showtimes · plan_walk …
       └ 클라이언트 (Client)    ← 바깥 세계 연결 — OllamaClient · Kakao · KOFIC · DB 리포지토리
```

- 판단 루프(`AgentLoop`)·행동 프로토콜·`JudgePort`는 허브의 공용 부품이다. 앱은 도구 목록과
  시스템 프롬프트만 준다. 루프가 코드로 막는 것: 인자 근거(발화·대화·결과에 없는 제목·지역 차단),
  같은 호출 반복, 호출 예산(3단계).
- 허브는 스포크를 import하지 않으므로, 앱은 기동 때 허브 레지스트리에 자기 에이전트를 등록하고
  오케스트레이터는 그 목록만 본다(예정).
- 구 `SuvisdevOrchestrator`는 실체가 Ollama HTTP 클라이언트라 `OllamaClient`로 개명했다.

### Clean Architecture + Hexagonal (Ports & Adapters)

의존 방향은 바깥 → 안쪽. 도메인 코어는 FastAPI·SQLAlchemy·HTTP에 의존하지 않는다.

```
adapter/inbound/api/     ← Router (Schema 수신, DI)
    │
    ▼
app/ports/input/         ← Input Port (ABC)
    │
    ▼
app/use_cases/           ← Interactor (비즈니스 로직)
    │
    ▼
app/ports/output/        ← Output Port (ABC)
    │
    ▼
adapter/outbound/pg/     ← PgRepository (ORM)
```

이 구조 덕분에 Gildle의 CSV → PostgreSQL 전환 시 **도메인/애플리케이션 레이어 변경 0건**으로 Adapter만 교체할 수 있었다.

---

## 배포 환경

### 이중 환경 운영

| 환경 | 구성 | 특이사항 |
|------|------|----------|
| **EC2 (프로덕션)** | Docker Compose (backend·auth·db·redis·nginx·cloudflared) | GPU 없음, 디스크 30GB |
| **집 GPU (LoRA)** | WSL2 + RTX, systemd lora-server (:8200) | Cloudflare Tunnel로 EC2에 노출 |
| **Vercel** | Next.js 프론트엔드 자동 배포 | main 브랜치 푸시 시 자동 빌드 |

### 추천 엔진 폴백

```
GPU 가용 (기본)                     GPU 미가용
─────────────                      ──────────
RECOMMENDATION_BACKEND=lora         RECOMMENDATION_BACKEND=gemini
lora_server:8200                    Gemini API
    │                                   │
    └─ Cloudflare Tunnel ──→ EC2        └──→ EC2
```

- `RECOMMENDATION_BACKEND` env 하나로 전환, docker compose 재기동 왕복 약 8초
- EC2 배포 후 항상 `docker exec <backend> printenv | grep RECOMMENDATION_BACKEND`로 확인

---

## ERD · 데이터 모델

Mova의 영화 도메인 테이블과 Gildle의 보행 그래프 테이블이 users 테이블을 공유하되, ORM 메타데이터는 앱별로 분리한다.

```
┌─ Mova ────────────────────────────────────────────────┐
│  movies ──┬── characters ── actors                    │
│           ├── movie_directors                         │
│           ├── tags                                    │
│           ├── reviews (AI 자동 + 사용자 작성)           │
│           ├── rankings (KOFIC 일별/주간/월간)           │
│           ├── picks (추천 이력)                         │
│           └── watchlist · user_actions                │
│                                                       │
│  market_chat_conversations ── market_chat_entries      │
│  user_taste_vectors (리뷰 기반 취향 벡터, 768d)         │
│  hub_knowledge (RAG 벡터, Gemini embedding)            │
├───────────────────────────────────────────────────────┤
│  users ──┬── user_identity (Google·Kakao·Naver OAuth) │
│          └── groups                         [공유]     │
├─ Gildle ──────────────────────────────────────────────┤
│  route_nodes (osm_id) ── route_edges                  │
│      tree_score · hazard_score · dog_friendly_score   │
├─ 관리 ────────────────────────────────────────────────┤
│  visitor_activity · alembic_version                   │
└───────────────────────────────────────────────────────┘
```

- 마이그레이션: Alembic (36개 테이블, head `20260821_0001`)
- Mova: movies 200+편 적재, actors 389 / characters 371 / movie_directors 40
- Gildle: route_nodes 1,178 / route_edges 1,616 (3축 점수)

---

## 인증 · 보안

### 인증 체계

| 항목 | 구현 |
|------|------|
| 웹 OAuth | Google · Kakao · Naver OAuth 2.0, RS256 JWT |
| 모바일 OAuth | 카카오 모바일 로그인 (kapi 토큰 검증), HS256 JWT |
| 세션 관리 | Redis (`auth:refresh:web:{jti}` / `auth:refresh:mobile:{userId}` 네임스페이스 분리) |
| 권한 | admin / user 역할, `require_admin` · `require_user` 공통 가드 |
| 검증 폴백 | RS256 → HS256 이중 검증 (웹·모바일 토큰 호환) |

### 3계층 토큰 전달

```
클라이언트 (Bearer Token)
    │
    ▼
route.ts 프록시 (Authorization 헤더 전달)
    │
    ▼
FastAPI (require_user / require_admin 가드)
    │
    └→ JWT 디코딩 → user_id 추출 → 소유권 검증
```

### 보안 대응

| 위협 | 대응 |
|------|------|
| IDOR | 전수 조사 5건 식별 → 전부 JWT 토큰 기반으로 전환, 취약점 0건 |
| S3 미인증 접근 | 버킷 비공개 + presigned URL (1시간 만료)로만 접근 |
| API 키 노출 | 환경변수 분리 (`.env` 단일 파일), `.env.auth` 격리 (RS256 개인키) |
| Spoke 간 의존 | `import-linter` 자동 검사로 아키텍처 규칙 강제 |

---

## 테스트

| 영역 | 테스트 수 | 방법 |
|------|----------|------|
| Mova | 200+ | pytest, 유스케이스·리포지토리·API 레이어별 |
| Gildle | 170+ | pytest, Clean Architecture 레이어별 단위·통합 |
| 프론트엔드 | - | `pnpm type-check` (tsc --noEmit) + `pnpm lint` |
| 전체 백엔드 | 545+ | `pytest -m "not gpu and not ollama"` |

```bash
# 일반 실행 (GPU·Ollama 불필요)
pytest -m "not gpu and not ollama"

# Gildle 단독
pytest apps/gildle/tests

# 프론트 타입 검사
pnpm type-check
```

---

## 트러블슈팅

### Docker .env 미설정 → DB 전면 장애

- **증상**: 백엔드 전 요청 502
- **원인**: `docker compose up -d`에서 `--env-file suvisdev/.env`를 빠뜨려 `${POSTGRES_USER}` 등이 빈 문자열로 치환, DB 컨테이너가 빈 자격증명으로 재생성
- **해결**: CLAUDE.md에 필수 규칙으로 등록, 이후 동일 장애 0건
- **교훈**: Docker Compose 환경변수 외부 주입 시 `.env-file` 누락은 무증상 실패를 유발 — CI 체크리스트에 포함

### S3 Presigned URL 403

- **증상**: EC2에서 발급한 presigned URL로 이미지 불러오기 실패
- **원인**: `boto3.client("s3")` 기본 글로벌 엔드포인트 → 리전 엔드포인트로 307 리다이렉트 시 SigV4 서명의 Host 불일치
- **해결**: `endpoint_url=f"https://s3.{region}.amazonaws.com"` 명시
- **교훈**: SigV4 서명은 Host 헤더를 포함하므로 리다이렉트 시 무효화됨

### PendingRollbackError 도미노

- **증상**: 대량 수집 중 1건 실패 후 이후 418건 전부 연쇄 실패
- **원인**: SQLAlchemy AsyncSession이 flush 실패 시 pending-rollback 상태를 유지, `session.rollback()` 호출 없이는 같은 세션의 이후 모든 쿼리 실패
- **해결**: 각 except 블록에 `await session.rollback()` 추가, 실패를 영화 한 편으로 격리
- **교훈**: 배치 처리에서 세션 공유 시 실패 격리 패턴 필수

### EC2 30GB 디스크 반복 부족

- **증상**: backend·auth 컨테이너 빌드 중 디스크 풀
- **원인**: 동일 Dockerfile(torch+CUDA)인데 이미지가 따로 태깅돼 중복 레이어 미공유
- **해결**: `docker builder prune -a` + 순차 빌드로 임시 대응, 이미지 통합은 백로그
