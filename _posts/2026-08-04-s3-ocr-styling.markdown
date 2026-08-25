---
layout: default
title: "S3 사진 OCR + 프론트 스타일 통일"
date: 2026-08-04
categories: [feature, frontend]
---

# S3 사진 OCR + 프론트 스타일 통일

## S3 사진 OCR 페이지

S3에 저장된 사진을 Gemini 멀티모달로 OCR 처리해 텍스트를 추출하는 `/lesson/photos` 페이지를 구현했다.

### 백엔드

| 구현 | 내용 |
|------|------|
| `Tank.list_objects(prefix)` | S3 `list_objects_v2` 페이지네이션 |
| `GET /api/media/photos/ocr` | `require_admin` 가드, 최신순 정렬, 최대 12장 |
| OCR 엔진 | Gemini 멀티모달 이미지→텍스트 |
| 에러 처리 | 개별 OCR 실패는 전체를 막지 않고 "(텍스트 추출 실패)"로 대체 |

### 트러블슈팅: Presigned URL 403

- **증상**: EC2에서 발급한 presigned URL로 이미지를 불러오면 403
- **원인**: `boto3.client("s3", region_name=...)` 만 지정하면 글로벌 엔드포인트(`s3.amazonaws.com`) 기준으로 URL이 생성됨. 리전 엔드포인트로 307 리다이렉트되면서 SigV4 서명의 `Host` 헤더가 불일치
- **해결**: `endpoint_url=f"https://s3.{self.region}.amazonaws.com"` 명시

## 프론트 스타일 통일

`suvis/` 프론트엔드의 Tailwind CSS 임의값 문법을 shadcn/ui 토큰 체계로 통일했다.

- `app/globals.css`의 `@theme inline`에 `--color-mova-*` 10개 토큰 등록
- `app/mova/**` · `components/mova/**` 27개 tsx 파일에서 354곳의 `text-[var(--mova-text)]` → `text-mova-text` 기계적 치환
- 미사용 `styles/globals.css` 삭제
- `pnpm type-check` · `pnpm build` 통과, 시각적 회귀 없음
