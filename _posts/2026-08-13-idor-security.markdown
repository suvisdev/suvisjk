---
layout: default
title: "IDOR 취약점 5건 전수 수정"
date: 2026-08-13
categories: [security]
---

# IDOR 취약점 5건 전수 수정

## 발견

API 엔드포인트 전수 조사에서 user_id를 경로·바디 파라미터로 받아 다른 사용자의 리소스에 접근할 수 있는 IDOR(Insecure Direct Object Reference) 취약점 5건을 발견했다.

## 취약점 목록

| # | 엔드포인트 | 취약 유형 |
|---|-----------|-----------|
| 1 | mypage | 경로에 user_id 직접 노출 |
| 2 | profile | 바디 파라미터로 user_id 수신 |
| 3 | watchlist | 다른 사용자의 찜 목록 접근 가능 |
| 4 | picks | 추천 이력 타인 열람 가능 |
| 5 | chat | 채팅 기록 타인 접근 가능 |

## 수정 방법

모든 엔드포인트에서 user_id를 클라이언트 입력 대신 JWT 토큰에서 추출하도록 전환했다.

```python
# Before (취약)
@router.get("/mypage/{user_id}")
async def get_mypage(user_id: int):
    ...

# After (수정)
@router.get("/mypage")
async def get_mypage(user=Depends(require_user)):
    user_id = user.user_id  # JWT 토큰에서 추출
    ...
```

## 3계층 토큰 전달

프론트엔드 → API 프록시 → 백엔드 전 구간에서 JWT 토큰이 전달되도록 보장했다.

```
클라이언트 (Bearer Token)
    │
    ▼
route.ts 프록시 (Authorization 헤더 전달)
    │
    ▼
백엔드 (require_user / require_admin 가드)
    │
    └→ JWT 디코딩 → user_id 추출 → 소유권 검증
```

## 결과

- IDOR 취약점 **0건** (전수 조사 완료)
- 모든 리소스 접근에 토큰 기반 소유권 검증 적용
