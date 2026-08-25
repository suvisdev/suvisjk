---
layout: default
title: "Flutter 카카오 모바일 로그인 구현"
date: 2026-08-03
categories: [feature, mobile]
---

# Flutter 카카오 모바일 로그인 구현

## 작업 요약

susu(Flutter) 앱에 카카오 모바일 로그인과 백엔드 JWT 발급을 풀스택으로 구현했다.

## 인증 흐름

```
Flutter 앱
    │
    ├─ 카카오톡 설치 → loginWithKakaoTalk()
    └─ 미설치/실패 → loginWithKakaoAccount() 폴백
         │
         ▼
    access_token 취득
         │
         ▼
    POST /auth/kakao/mobile (access_token)
         │
         ▼
    백엔드: kapi /v2/user/me 으로 토큰 검증
         │
         ├─→ 기존 사용자 → JWT 발급
         └─→ 최초 로그인 → UserMirror 자동 생성 → JWT 발급
              │
              ▼
         flutter_secure_storage에 JWT/refresh 저장
```

## 구현 내용

### 백엔드 (`apps/auth`)

| 파일 | 역할 |
|------|------|
| `kakao_mobile_verifier.py` | kapi `/v2/user/me`로 모바일 access_token 검증 |
| `mobile_refresh_store.py` | `auth:refresh:mobile:{userId}` Redis 저장소 (웹과 분리) |
| `repository.py` | `find_or_create_by_kakao()` 카카오 자동 가입 |
| `services.py` | `login_with_kakao_mobile` / `mobile_refresh` / `mobile_logout` |

### Flutter (`susu`)

- `SplashScreen` → 저장된 세션 있으면 바로 메인, 없으면 인트로 영상 → `AuthScreen`
- Android(`AndroidManifest.xml`) / iOS(`Info.plist`) 에 카카오 커스텀 URL 스킴 설정
- nginx에 `/auth/*` → `auth:9000` 프록시 추가 (기존에 누락돼 있던 것)

## 실기기 검증

EC2에서 `docker compose up -d --build auth` + `nginx restart` 후 폰에서 카카오 로그인 → JWT 수신 → 메인 화면 이동까지 E2E 성공 확인.
