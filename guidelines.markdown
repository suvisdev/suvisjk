---
layout: default
title: 주요 개발 수행 지침
permalink: /guidelines/
nav_order: 6
---

# 주요 개발 수행 지침

## 아키텍처 원칙

### Clean Architecture + Hexagonal (Ports & Adapters)

의존 방향은 바깥 → 안쪽. 도메인(앱 코어)은 FastAPI·SQLAlchemy·HTTP에 의존하지 않는다.

```
Router(Schema) → Input Port(Schema→Dto) → Interactor(Schema→Command→Port)
→ PgRepository(Command→ORM) → Dto → Router(to_schema)
```

| 레이어 | 위치 | 책임 |
|--------|------|------|
| Router | `adapter/inbound/api/v1/` | Schema 수신, DI, Dto→Schema 변환 |
| Input Port | `app/ports/input/` | ABC, Schema in / Dto out 시그니처 |
| Interactor | `app/use_cases/` | Schema→Command, Port 호출, Dto 반환 |
| Output Port | `app/ports/output/` | ABC, Command in |
| PgRepository | `adapter/outbound/pg/` | Command 처리, ORM |

### 모듈러 모놀리식 · Star Topology

단일 배포 단위이지만 앱 간 경계가 명확히 분리된다.

```
     mova   gildle   viewer   analytics   media
       \      |       /        /
        ★    ontology    ★           (Hub)
       /      |       \
   dispatch  execsuite  titanic  contents
```

- **Hub(ontology)**: 공통 이벤트·크롤링·AI 파이프라인. Spoke를 import하지 않는다.
- **Spoke**: Hub만 import 가능. Spoke 간 직접 import 금지 — Hub 이벤트 버스 경유.
- **`import-linter`** 로 커밋 시 자동 검사(계약 7개 — Hub 독립 · Spoke 간 독립 · Mova·Titanic·Gildle 도메인 순수성 등).

### 도메인 모델은 필요한 곳에만

헥사고날·클린 아키텍처는 전면 적용하지만 DDD(애그리거트·도메인 이벤트)는 전면 적용하지 않는다.
2026-09-27 감사에서 규칙 없이 만든 엔티티 14개가 전부 미사용으로 드러나 삭제했다.

| 신호 | 대응 | 실례 |
|------|------|------|
| 같은 규칙이 두 곳 이상에 복사됨 | 값 객체·도메인 서비스로 모은다 | 제목 정규화(`MovieTitle`), 동행 조건(`companion_expansion`) |
| 생성·상태 변경에 검증 조건이 붙음 | 엔티티에 둔다 | gildle `walk_entity`, `push_token_entity` |
| 외부 의존 없이 독립인 계산 규칙 | 도메인 서비스 | gildle `RouteWeightCalculator` |

### 에이전트 층

LLM이 판단하는 자리의 이름을 네 층으로 고정한다: 오케스트레이터(허브 전체 두뇌) → 에이전트(앱 두뇌)
→ 도구 → 클라이언트. 판단 루프·행동 프로토콜은 허브의 공용 부품이고, 앱은 도구 목록과 시스템
프롬프트만 준다. **판단은 모델, 사실은 코드 템플릿, 가드(인자 근거·반복·호출 예산)는 코드.**

### SOLID 원칙

| 원칙 | 적용 |
|------|------|
| SRP | Router=HTTP, Interactor=유스케이스, Repository=persistence |
| ISP | 도메인별 작은 입력 포트 분리 |
| DIP | Interactor → Port ABC; `dependencies/`에서 구현체 주입 |
| OCP | `RoseModelStrategy` ABC — 새 알고리즘 추가 시 기존 코드 수정 불필요 |
| LSP | Port ABC `@abstractmethod`로 계약 강제 |

---

## 개발 표준 및 산출물

| 항목 | 표준 |
|------|------|
| 언어 | Python 3.12 (백엔드), TypeScript 5.7 (프론트), Dart (모바일) |
| 프레임워크 | FastAPI, Next.js 16 (App Router), Flutter |
| 배포 | k3s(`./k8s/deploy.sh`), Vercel(프론트), PR → main 머지 |
| DB | PostgreSQL 16 + pgvector, Redis (노트북 Docker) |
| ORM | SQLAlchemy 2.0 Async |
| 커밋 | Conventional Commits (`feat:`, `fix:`, `docs:`, `refactor:`) |
| 린트 | ruff (Python), ESLint + Prettier (TS), import-linter |
| 타입 | mypy strict (Python), tsc --noEmit (TS) |

---

## 품질 관리 및 테스트

| 항목 | 방법 |
|------|------|
| 테스트 프레임워크 | pytest (백엔드) |
| 테스트 마커 | `gpu` (GPU 필요), `ollama` (Ollama 서버 필요) |
| 일반 실행 | `pytest -m "not gpu and not ollama"` |
| 프론트 검증 | `pnpm type-check` + `pnpm lint` |
| 백엔드 테스트 | 1,043건 (Mova 471 · Gildle 257 · 허브·인증 등) |
| 운영 회귀 하네스 | 채팅 단일턴 28질의 · 멀티턴 17장면 — 배포 전후 실제 API로 비교 |
| 모델 평가 | 같은 평가셋(86행)으로 회차 비교, 내보내기 판단은 운영과 같은 GGUF로 재채점 |
| 앱 검증 | `flutter analyze` + `flutter test` |
| 화면 검증 | UI 변경은 로컬 화면을 헤드리스 브라우저로 캡처해 확인한 뒤 배포 |
| 보안 | IDOR 전수 조사, httpOnly 쿠키 세션, 무인증 엔드포인트는 근거를 주석으로 |
| 기록 | 영역별 작업 일지 + `LESSONS.md`(실패 → 원인 → 극복 한 줄) + 인터뷰 질문 |
