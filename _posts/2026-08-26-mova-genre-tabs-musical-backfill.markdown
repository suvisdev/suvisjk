---
layout: default
title: "Mova 장르 탭 확장과 뮤지컬 태그 백필"
date: 2026-08-26
categories: [bugfix, mova]
---

# Mova 장르 탭 확장과 뮤지컬 태그 백필

## 요약

| 영역 | 내용 |
|------|------|
| 안정성 | 8/25 폴백 어댑터가 DI 미연결이던 문제 수정 — Gemini 자동 폴백 실제 작동 |
| 데이터 | TMDB 키워드 기반 "뮤지컬" 장르 태그 65편 백필 |
| UI | `/mova/movies` 장르 탭 16개로 확장, 로딩 영상 크롭 수정 |

## Gemini 자동 폴백 DI 연결

프로덕션 `/mova/chat` 502의 원인을 규명했다. 노트북 lora-server가 내려가 터널이 530을 반환 → EC2의 LoRA 호출 연속 실패 → 서킷 오픈까지는 설계대로였으나, 8/25에 추가한 `FallbackRecommendationAdapter`가 **DI에 연결돼 있지 않아** Gemini 자동 폴백이 작동하지 않고 에러가 그대로 노출됐다.

`get_recommendation_port()`가 lora 모드에서 `FallbackRecommendationAdapter(primary=LoRA, fallback=Gemini)`를 조립하도록 연결했다. 서킷 쿨다운(60초) 후 LoRA 재시도 → 성공 시 자동 복귀하는 왕복 자동 전환이 완성됐다.

## 뮤지컬 장르 태그 백필

DB 실측 결과 "뮤지컬" 장르가 **0편**이었다 — TMDB 장르 체계에 뮤지컬이 없어 레미제라블은 "역사, 드라마", 라라랜드는 "코미디, 로맨스, 드라마"로만 수집돼 있었다. "음악"(92편)은 콘서트 실황·다큐 계열로 뮤지컬과 별개다.

- TMDB 키워드(musical, broadway musical, musical theater) discover 458편과 카탈로그 3,419편을 제목 정규화 + 연도 ±1로 매칭 → **65편** 태깅 (레미제라블·라라랜드·위대한 쇼맨·위키드·겨울왕국 등).
- `scripts/tag_musical_genre.py` 신규 — 멱등 실행, `--dry-run` 지원. `TmdbAdapter.fetch_discover`에 `with_keywords` 파라미터 추가.

## 장르 탭 확장 + 로딩 영상 수정

- `/mova/movies` 장르 탭을 DB 실측에 맞게 갱신: 모험·판타지·가족·미스터리·음악을 추가하고 뮤지컬은 백필과 함께 유지 (총 16탭). 편수가 적은 역사·전쟁·서부·TV영화는 제외.
- 채팅 로딩 클래퍼보드 영상의 하단 문구 잘림 수정. 영상 파일은 원본과 동일했고, 원인은 CSS 크롭(`aspect-[7/5]` + `object-left`)이었다 — `aspect-video`로 교체하고 크롭을 제거했다.

## 검증

mova 채팅 테스트 12건 통과, `pnpm type-check` 통과. 배포 후 EC2 컨테이너 코드·알렘빅 head·Vercel 번들·백필 데이터(65편)까지 실측 확인.
