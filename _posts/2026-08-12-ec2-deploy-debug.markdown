---
layout: default
title: "EC2 배포 디버깅 + 운영 안정화"
date: 2026-08-12
categories: [deploy, infra]
---

# EC2 배포 디버깅 + 운영 안정화

## Docker .env 미설정 장애

`docker compose up -d --build backend`를 `--env-file suvisdev/.env` 없이 실행하여 DB 전면 장애가 발생한 사고를 수습했다.

| 항목 | 내용 |
|------|------|
| 증상 | 백엔드 전 요청 502 |
| 원인 | `${POSTGRES_USER}` 등이 빈 문자열로 치환, DB 컨테이너가 빈 자격증명으로 재생성 |
| 데이터 | `db_data` named volume 보존으로 유실 없음 |
| 해결 | `docker compose --env-file suvisdev/.env up -d --build backend db` |
| 재발 방지 | CLAUDE.md에 `--env-file` 필수 규칙 등록 |

## AWS S3 연동 정상화

EC2에 AWS 자격증명이 누락돼 있던 문제를 발견하고 수정했다.

- `Tank.list_objects` 호출 시 `NoCredentialsError` 발생
- IAM Role 미부착 + `.env`에 키 미설정 상태
- 1차 시도: `AWS_SECRET_ACCESS_KEY`가 14자로 잘려 `InvalidAccessKeyId` 재실패
- 재발급 후 정상화, S3 객체 다운로드 + Gemini OCR 종단 검증 완료

## 계정 불일치 발견

susu(카카오 로그인, `user_id=4`)와 웹 관리자(구글 로그인, 다른 `user_id`)가 별개 계정으로 생성돼 있는 문제를 발견했다.

- `/lesson/photos`에서 관리자 본인 사진만 조회하도록 되어 있었으나, 사진은 카카오 계정으로 업로드
- 관리자는 전체 사용자 사진을 조회하도록 스코프 변경
- 계정 연결 기능은 별도 백로그

## Mova 동적 세그먼트 404 해결

Vercel에서 mova 동적 세그먼트 라우트(`[user_id]`, `[slug]`, `[movieId]`)가 404를 반환하는 문제를 해결했다.

- **원인**: `next.config.mjs`의 `async rewrites()` 캐치올(`/api/:path*`)이 동적 라우트보다 먼저 매칭
- **해결**: 캐치올에 의존하던 3개 라우트(`titanic/smith/chat`, `soccer/chat`, `langchain/chat`)에 전용 route.ts를 신설한 뒤 캐치올 삭제
- **검증**: 배포 후 `x-matched-path` 헤더로 동적 라우트 6개 정상 매칭 확인
