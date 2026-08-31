---
layout: default
title: "mypy 재활성화 — 415건에서 0건까지"
date: 2026-08-31
categories: [refactor, backend, infra]
---

# mypy 재활성화 — 415건에서 0건까지

## 요약

| 영역 | 내용 |
|------|------|
| 문제 | mypy strict 모드 1,327건 에러로 pre-commit 훅 비활성화 (2026-08-26) |
| 실측 | exclude 보정 후 실제 대상 415건 (1,079 파일) |
| 수정 | 5단계 구조적 패턴 수정, 118파일 |
| 결과 | 0 errors, pytest 728 passed, pre-commit 훅 복구 |

## 왜 꺼져 있었나

8월 26일에 전체 검증 파이프라인을 복구하면서 mypy를 처음 돌렸더니 1,327건이 쏟아졌다. 커밋할 때마다 모든 백엔드 변경이 막히는 상태라 훅을 임시 비활성화하고 백로그에 올렸다.

실제로 돌려보니 `tests/`, `scripts/`, `alembic/` exclude가 이미 적용돼 415건이었고, 에러의 80%가 다섯 가지 구조적 패턴에 집중돼 있었다.

## 5단계 수정

### Phase 1 — 설정 보정

```toml
[tool.mypy]
explicit_package_bases = true   # 모듈 이름 충돌 해소
exclude = ["tests/", "test/", "_docs/", "scripts/", "alembic/"]
```

`explicit_package_bases`가 없으면 `suvisdev.apps.mova`와 `mova` 두 이름으로 같은 파일을 발견해 첫 파일에서 멈춘다. `test/`(s 없음)와 `_docs/`(임시 스크립트)도 exclude에 추가.

### Phase 2A — `from_orm(orm: object)` → `orm: Any` (~81건)

Clean Architecture 경계의 역직렬화 메서드 12개. ORM 클래스를 직접 import하면 레이어 위반이라 `object`로 선언돼 있었는데, `.id`, `.slug` 같은 속성 접근이 전부 `[attr-defined]`로 잡혔다.

```python
# Before
@classmethod
def from_orm(cls, orm: object) -> MovieDetailDto: ...

# After — Any는 이 자리에서 정당하다 (역직렬화 경계)
@classmethod
def from_orm(cls, orm: Any) -> MovieDetailDto: ...
```

### Phase 2B — `to_schema() -> object` → 구체 반환 타입 (~33건)

순환 import 회피를 위해 `-> object`로 선언된 `to_schema()` 메서드들. `TYPE_CHECKING` 블록으로 타입만 노출하고 런타임 lazy import는 유지.

```python
from __future__ import annotations
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from mova.adapter.inbound.api.schemas.xxx_schema import XxxSchema

def to_schema(self) -> XxxSchema:  # object → 구체 타입
    from mova.adapter.inbound.api.schemas.xxx_schema import XxxSchema
    return XxxSchema(...)
```

### Phase 2C — Titanic 포트 ABC async 통일 (~29건)

8개 출력 포트 ABC가 `def method()`인데 구현체는 `async def method()`. I/O 메서드라 포트를 `async def`로 맞추는 게 맞다.

### Phase 3-4 — 개별 타입 수정 (~272건)

나머지는 파일별 개별 수정:
- SQLAlchemy `ColumnElement[bool]` vs `Exists` 타입 불일치 → `# type: ignore[assignment]`
- Optional 접근 (`dict | None`에 `.get()`) → None 가드
- `no-any-return` → `cast()` 또는 변수 타입 명시
- 외부 라이브러리 (ultralytics, timm, boto3) → `# type: ignore[error-code]`

## pre-commit 훅: mirrors-mypy → 로컬 훅

처음엔 `mirrors-mypy`를 그대로 활성화했는데, 이 훅은 **변경된 파일만 개별 전달**해서 mypy가 base class를 해석하지 못하고 `Class cannot subclass "X" (has type "Any")` 같은 false positive를 133건 쏟아냈다.

import-linter와 동일하게 로컬 훅으로 전환해서 프로젝트 전체를 한 번에 실행하도록 변경.

```yaml
- repo: local
  hooks:
    - id: mypy
      name: "mypy — strict type check"
      entry: bash -c 'cd suvisdev && python -m mypy --config-file pyproject.toml --explicit-package-bases apps/'
      language: system
      pass_filenames: false
      files: ^suvisdev/
```

## 에러 감소 추이

| 단계 | 해소 | 누적 잔여 |
|------|------|----------|
| 시작 | — | 415 |
| Phase 2A (from_orm) | ~81 | ~334 |
| Phase 2B (to_schema) | ~33 | ~301 |
| Phase 2C (titanic async) | ~29 | ~272 |
| Phase 3-4 (개별) | ~272 | 0 |

## 배운 것

1. **구조적 패턴부터** — 415건 중 143건(34%)이 `from_orm`·`to_schema`·async 불일치 세 패턴. 개별 에러를 하나씩 잡기 전에 패턴을 먼저 식별하면 효율이 3배 이상 차이난다.

2. **`TYPE_CHECKING` + lazy import** — Clean Architecture에서 순환 import와 타입 안전성을 동시에 잡는 표준 패턴. `from __future__ import annotations`와 조합하면 런타임 비용 0.

3. **pre-commit mypy는 per-file로 돌리면 안 된다** — ABC·포트 같은 cross-file 의존이 있는 프로젝트에서 파일 단위 실행은 false positive 폭탄. `pass_filenames: false`로 전체 실행이 맞다.
