---
layout: default
title: 주요 개발 수행 지침
permalink: /guidelines/
nav_order: 7
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
     mova   gildle   viewer
       \      |       /
        ★ ontology ★     (Hub)
       /      |       \
   titanic  inception  ...
```

- **Hub(ontology)**: 공통 이벤트·크롤링·AI 파이프라인. Spoke를 import하지 않는다.
- **Spoke**: Hub만 import 가능. Spoke 간 직접 import 금지 — Hub 이벤트 버스 경유.
- **`import-linter`** 로 커밋 시 자동 검사.

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
| DB | PostgreSQL (Neon), Redis |
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
| Gildle 테스트 | 170+ 테스트, Clean Architecture 레이어별 단위·통합 |
| 보안 | IDOR 전수 조사, OWASP Top 10 대응 |
