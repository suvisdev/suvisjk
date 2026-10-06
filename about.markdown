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
| 개발 기간 | 2026.07.08 ~ 2026.09.30 (약 12주) — v1 마감 2026-09-30 |
| 개발 인원 | 1명 (개인 프로젝트) |
| 역할 | 풀스택 개발 · ML 파이프라인 · 인프라 · 데이터 수집 |
| 레포지토리 | [github.com/suvisdev/suvisdev.cloud](https://github.com/suvisdev/suvisdev.cloud) |
| 데모 | [suvisdev.cloud](https://suvisdev.cloud) |

---

## 프로젝트 배경

**영화 추천** — 수천 편 속에서 취향에 맞는 영화를 고르는 데 시간이 든다. 기존 협업 필터링은 콜드 스타트 문제가 있고, LLM 챗봇은 없는 극장과 상영 시간을 지어낸다. Mova는 직접 파인튜닝한 소형 모델이 도구를 골라 부르고, 사실은 데이터에서 가져와 답한다.

**산책 경로** — 반려견과 산책할 때 그늘·경사·결빙·들를 곳을 고려한 경로를 찾기 어렵다. Gildle은 서울 전역 보행 그래프에 환경 점수와 건물 그림자를 입혀 계절·시간대별 경로 후보를 이유와 함께 보여 준다.

---

## 서브 프로젝트

### [Mova — AI 영화 추천](/mova/)

파인튜닝한 EXAONE 3.5 2.4B가 채팅의 판단(도구 선택)과 추천을 맡고, Gemini가 폴백과 리뷰 요약을 맡는 영화 플랫폼. 카탈로그 3,919편.

| 핵심 기능 | 구현 |
|-----------|------|
| 채팅 에이전트 | 판단 모델 v9가 도구 10개 중 다음 행동을 고름. 인자 근거·반복 호출·호출 예산은 코드 가드 |
| AI 추천 | EXAONE 3.5 2.4B LoRA(GGUF Q5_K_M) + Gemini 자동 폴백, 취향 벡터 코사인 재정렬 |
| 봤어요 · 별점 | 채팅 한 문장으로 기록, 후보 되묻기 뒤의 선택을 하던 일로 이어 받음 |
| 에디터 리뷰 | 뉴스 요약 → 리뷰 + 감정분석 별점 + 출처 링크 |
| 랭킹 · 시간표 | KOFIC 박스오피스, 카카오 로컬·롯데시네마 상영 시간표 |
| 대량 수집 | TMDB + KOFIC 배치 파이프라인, 영화당 upsert → credits 백필 → RAG 인제스트 |

### [Gildle — 반려견 산책 경로](/gildle/)

서울 전역 보행 그래프에 나무·결빙·경사 점수와 건물 그림자를 입혀, 계절·시간대별 경로 후보를 계산하는 산책 경로 추천 서비스. 웹과 안드로이드 앱(길들)이 같은 기능을 제공한다.

| 핵심 기능 | 구현 |
|-----------|------|
| 보행 그래프 | 서울 전역 노드 164,740 · 간선 약 23만 (OSM / osmnx) |
| 건물 그림자 | 건물 약 80만 동, 월별 12벌 × 13시간대 사전 계산 |
| 경로 후보 | 빠른·그늘·푸른·편한·언덕길 + 고를 이유, 계절 모드 3종 |
| 문장 추천 | EXAONE 7.8B가 조건을 읽고 규칙이 검증, A*가 계산 |
| 산책 기록 | 위치 추적·저장·통계, 웹·앱 공통 |
| 지도 | 네이버 지도(웹 JS v3 · Flutter SDK) |

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

프로덕션은 **집 서버와 GPU 노트북 두 대의 온프레미스**다(2026-10-05~06). DB·Redis·추천 모델 서버(lora-server)는
상시 켜 둔 집 서버 한 곳에 두고, 두 기기 모두 k3s로 같은 백엔드를 띄운다. 노트북이 집 네트워크에 있으면 노트북이 서빙하고,
꺼지거나 밖으로 나가면 약 30초 안에 집 서버가 넘겨받는다(판단기 두 개가 Cloudflare Tunnel 연결을 바꾼다). 노트북이 밖에
있어도 Tailscale로 GPU(Ollama)만 집 서버에 빌려준다. 클라우드 분리안은 검토 뒤 기각했다 — 백엔드가 유휴에도 2GB RAM이라
무료 인스턴스에 들어가지 않고, 백엔드↔GPU 지연이 늘어난다(2026-09-29).

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

### 운영 구성 (2026-09 현재)

| 환경 | 구성 | 특이사항 |
|------|------|----------|
| **집 서버 (상시)** | k3s 파드(backend·auth·cloudflared) + Docker(PostgreSQL·Redis, 유일) + lora-server(:8200) + Ollama(CPU 예비) | GTX 1650 SUPER 4GB. 이미지를 빌드하지 않는다(메모리 부족으로 DB가 죽은 적이 있다) |
| **노트북 (우선 서빙·GPU)** | k3s 파드 + Ollama(GPU) + 배포 러너 | RTX 4060 8GB. 집이면 우선 서빙, 밖이면 GPU만 Tailscale로 제공 |
| **Colab** | LoRA 학습 → 병합 → GGUF 양자화 | 노트북 GPU는 운영 전용이라 학습은 코랩에서 끝낸다 |
| **Vercel** | Next.js 프론트엔드 | main 머지 시 자동 배포 |
| **EC2** | 중지 보관 | 2026-09-03 노트북으로 이전. 30GB 디스크·GPU 부재가 이전 사유 |

배포는 GitHub Actions가 한다 — main에 백엔드가 머지되면 노트북의 셀프호스티드 러너가 이미지를 빌드하고, 바뀐 레이어만
집 서버의 로컬 레지스트리로 보내 두 기기를 같은 버전으로 맞춘다(노트북이 집 밖이면 노트북은 빌드만 하고 집 서버에 반영).
각 기기 안에서는 `./k8s/deploy.sh --external-db`가 `.env`를 Secret으로 변환해 주입하고 롤아웃한다. `.env`가 없으면 스크립트가 즉시 실패하므로 compose 시절의 "빈 자격증명으로 DB 재생성" 사고는 구조적으로 재발하지 않는다.

### 추천 엔진 폴백

```
추천 요청
    │
    ▼
lora-server :8200 (같은 노트북 GPU, llama.cpp GGUF)
    │
    ├─ 성공 → 응답
    └─ 실패 → Gemini API로 자동 전환
              연속 2회 실패하면 60초 동안 바로 Gemini로 우회(서킷 브레이커) → 이후 자동 복귀
```

- 초기엔 환경 변수(`RECOMMENDATION_BACKEND`)를 손으로 바꾸고 재기동했다(왕복 8초). 2026-08-26부터는 코드가 자동으로 전환한다.
- 환경 변수는 수동 고정이 필요할 때만 쓴다.

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
│  chat_conversations ── chat_messages                  │
│  user_taste_vectors (리뷰 기반 취향 벡터, 768d)         │
│  hub_knowledge (RAG 벡터, bge-m3 1024d, HNSW)          │
├───────────────────────────────────────────────────────┤
│  users ──┬── user_identity (Google·Kakao·Naver OAuth) │
│          └── groups                         [공유]     │
├─ Gildle ──────────────────────────────────────────────┤
│  walks (산책 기록·경로 좌표) · push_tokens             │
│  route_nodes ── route_edges (초기 시범 데이터)          │
│  * 서울 전역 그래프·그늘 표는 JSON 파일로 메모리 적재   │
├─ 관리 ────────────────────────────────────────────────┤
│  visitor_activity · alembic_version                   │
└───────────────────────────────────────────────────────┘
```

- 마이그레이션: Alembic (리비전 42개, 테이블 42개, head `20260930_0001`)
- Mova: movies 3,919편 · actors 21,551명 · tags 13,277개 · reviews 455건 · hub_knowledge 3,756건
- Gildle: 보행 그래프 노드 164,740 · 간선 약 23만(`scored_edges.json`), 그늘 표 월별 12벌

---

## 인증 · 보안

### 인증 체계

| 항목 | 구현 |
|------|------|
| 인증 게이트웨이 | 별도 파드(`auth.suvisdev.cloud`) — 이메일 로그인·가입, 토큰 발급·갱신·폐기, RS256 JWT |
| 웹 OAuth | Google · Kakao · Naver OAuth 2.0 |
| 모바일 | 카카오 모바일 로그인(kapi 토큰 검증) + 이메일 가입, 회원 탈퇴 |
| 웹 세션 | **httpOnly 쿠키**(access 7일 · refresh 14일). 토큰을 브라우저 저장소에 두지 않는다(2026-09-30 전환) |
| 세션 저장 | Redis (`auth:refresh:web:{jti}` / `auth:refresh:mobile:{userId}` 네임스페이스 분리) |
| 권한 | admin / user 역할, `require_admin` · `require_user` 공통 가드 |

### 쿠키 세션 (BFF)

```
브라우저 ── 쿠키 자동 전송 ──→ Next.js 프록시(same-origin)
                                  │  쿠키의 access를 Bearer로 바꿔 전달
                                  │  401이면 refresh로 한 번 갱신 후 재시도
                                  ▼
                              FastAPI (require_user / require_admin)
                                  └→ JWT 디코딩 → user_id → 소유권 검증
```

- 로그인은 Next.js 프록시가 게이트웨이를 대신 호출하고, 받은 토큰을 자기 도메인의 httpOnly 쿠키로 심는다. 게이트웨이는 다른 도메인이라 직접 심을 수 없다.
- 브라우저 자바스크립트는 토큰을 읽을 수 없다 — 스크립트 공격(XSS)으로 토큰이 빠져나가지 않는다.
- 화면의 "로그인됨" 표시는 브라우저 저장소의 표시용 정보다. 페이지를 열 때 쿠키가 유효한지 확인해 어긋나면 표시를 지운다.

### 보안 대응

| 위협 | 대응 |
|------|------|
| IDOR | 전수 조사 5건 식별 → 전부 JWT 토큰 기반으로 전환, 취약점 0건 |
| S3 미인증 접근 | 버킷 비공개 + presigned URL (1시간 만료)로만 접근 |
| 토큰 탈취(XSS) | 토큰을 localStorage에서 httpOnly 쿠키로 이전 |
| API 키 노출 | 환경변수 분리 (`.env` 단일 파일 → k8s Secret), RS256 개인키 격리 |
| Spoke 간 의존 | `import-linter` 자동 검사로 아키텍처 규칙 강제 |

---

## 테스트

| 영역 | 테스트 수 | 방법 |
|------|----------|------|
| Mova | 471 | pytest, 유스케이스·리포지토리·API 레이어별 |
| Gildle | 257 | pytest, Clean Architecture 레이어별 단위·통합 |
| Ontology(허브) · 인증 등 | 300+ | 에이전트 루프, RAG, 인증 게이트웨이 |
| 전체 백엔드 | **1,043 passed** | `pytest -m "not gpu and not ollama"` |
| 운영 회귀 하네스 | 단일턴 28 · 멀티턴 17 | 배포 전후 실제 채팅 API에 같은 질의를 보내 비교 |
| 프론트엔드 | - | `pnpm type-check` (tsc --noEmit) + `pnpm lint` |
| 길들 앱 | - | `flutter analyze` + `flutter test` |

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
- **해결**: CLAUDE.md에 필수 규칙으로 등록, 이후 k3s로 옮기면서 배포 스크립트가 `.env`를 Secret으로 변환(없으면 즉시 실패) — 구조적으로 재발 불가
- **교훈**: 환경변수 외부 주입에서 파일 누락은 무증상 실패를 부른다. 규칙으로 막는 것보다 스크립트가 실패하게 만드는 편이 확실하다

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
- **해결**: `docker builder prune -a` + 순차 빌드로 임시 대응 → 2026-09-03 프로덕션을 노트북으로 옮기면서 문제 자체가 사라짐

### 쿠키 세션 전환 뒤의 회귀 3건 (2026-09-30)

- **증상**: 화면은 로그인 상태인데 "인증이 필요합니다" / 마이페이지가 로그인 화면으로 되돌아감 / 본문 없는 응답(204)이 500
- **원인**: ① 전환 전에 로그인한 브라우저엔 표시만 있고 쿠키가 없음 ② 일괄 치환이 변수명 `request`만 잡아 `req`를 쓰는 프록시 9개를 놓침 ③ 프록시가 204 응답에 빈 본문을 실어 응답 생성이 실패
- **해결**: 페이지를 열 때 쿠키 유효성 확인, 패턴이 아닌 전수 검색으로 잔여 0 확인, 204·205·304는 본문 없이 전달
- **교훈**: 모든 요청이 지나가는 코드를 바꿨으면 200·401뿐 아니라 204·기존 로그인 사용자까지 확인한다
