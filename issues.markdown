---
layout: default
title: 미결 항목
permalink: /issues/
nav_order: 11
---

# 미결 항목

## 보류된 기능

| ID | 항목 | 상태 | 비고 |
|----|------|------|------|
| B-001 | Gildle 지역 확장 (영등포구 → 서울 전역) | 백로그 | OSM 데이터 용량·처리 시간 검토 필요 |
| B-002 | EC2 Docker 이미지 통합 (backend + auth) | 백로그 | 30GB 디스크 제약 근본 해결 |
| B-003 | Flutter 모바일 앱 완성 | 진행중 | 카카오 OAuth 연동 완료, UI 작업 중 |
| B-004 | mail/contacts 공개 페이지 401 이슈 | 미결 | `require_admin` 가드와 공개 접근 충돌 |

## 알려진 이슈

| ID | 항목 | 영향 | 우회 |
|----|------|------|------|
| I-001 | 위키피디아 429 레이트 리밋 | 크롤링 시 일부 영화 데이터 누락 | 부분 수집 허용, 재시도 시 완료 |
| I-002 | nvidia-smi free 수치 불안정 (WSL2) | VRAM 잔량 오판 | 실제 모델 로드로 확인 |
| I-003 | LoRA 서버 + 학습 동시 불가 | VRAM 경합 | systemctl stop/start 워크플로 |
