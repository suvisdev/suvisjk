---
layout: default
title: "Mova GPU 추천 경로 복구 — 데스크톱 재구축과 LoRA 재학습"
date: 2026-09-01
categories: [infra, mova]
---

# Mova GPU 추천 경로 복구 — 데스크톱 재구축과 LoRA 재학습

8월 6일부터 끊겨 있던 자체 GPU 추천 경로(EC2 → Cloudflare Tunnel → 집 GPU의
EXAONE LoRA 서버)를 데스크톱에서 재구축하고, 잃어버린 LoRA 어댑터까지 재학습해
당일 안에 프로덕션 E2E까지 복구했다.

## 출발점 — 배포 실측에서 발견한 것들

전날(8/31) 배포된 리뷰 감정분석·투표 기능의 실측 확인부터 시작했다.
마이그레이션 3건 적용, 신규 엔드포인트 라이브(감정 요약 200, 투표 미인증 401)
까지는 정상. 그런데 로그에서 감정분석 스케줄러가 리뷰마다
`'frozenset' object has no attribute 'discard'`로 실패하는 걸 발견했다.

추적 결과 **transformers 4.47.1 업스트림 버그**였다 —
`get_available_devices()`가 FrozenSet을 반환하는데, CPU 전용 머신 분기에서
`.discard("cpu")`를 호출한다. GPU 머신은 이 분기를 타지 않으므로 기능 자체는
무사하고, EC2(GPU 없음)에서는 어차피 실패가 의도된 경로라 조치 불필요.
다만 이 과정에서 **프로덕션 리뷰 166건 중 감정 데이터가 0건**임을 발견했다 —
읽기 경로만 라이브였고 분석 파이프라인은 한 번도 돈 적이 없었다.

## GPU 서버 재구축 — 노트북에서 데스크톱으로

구 노트북의 lora-server는 터널째 죽어 있었고(Cloudflare 530), 이 데스크톱엔
RTX 3050 8GB만 있고 아무것도 없었다. 재구축하면서 겪은 궁합 문제:

| 시도 | 결과 |
|------|------|
| gptqmodel 7.3.6 (AWQ 서빙) | transformers 5.16이 딸려옴 → EXAONE 구식 remote code와 비호환 (`get_input_embeddings` NotImplementedError) |
| transformers 4.47.1 고정 | gptqmodel이 `transformers.masking_utils` 임포트로 사망 (>=5.14 강제) |
| gptqmodel 6.x 다운그레이드 | 빌드 자체가 실패 |
| **fp16 hf 백엔드 (채택)** | serve.py에 이미 있던 미사용 분기 — 비양자화라 품질 손해도 없음 |

serve.py의 gptqmodel 임포트를 AWQ 분기 안 지연 임포트로 옮기고, hf 분기에
`trust_remote_code=True`와 `torch_dtype`을 정정하니 VRAM 5.5GB로 기동.
모델 가중치(9GB)는 용량이 넉넉한 D 드라이브에 배치했다. systemd 유저 서비스
2개(lora-server, cloudflared)와 linger로 재부팅 생존까지 확보하고, 신규 터널
`lora-desktop`으로 `lora.suvisdev.cloud` CNAME을 덮어썼다.

베이스 모델만으로 EC2 왕복은 성공했지만, 실호출에서 `'Actor:eal'` 같은
깨진 제목이 나왔다 — 파인튜닝 어댑터가 없으면 출력 형식이 불안정하다.

## LoRA 재학습 — 교사 증류를 프로덕션 카탈로그로

어댑터가 구 노트북에만 있어 재학습이 필요했다. 8/25와 같은 Gemini 교사 증류
방식이되, 이번엔 **EC2 프로덕션 DB(카탈로그 3,978편)에서 데이터셋을 생성**해
그라운딩 성공률을 높였다:

- 교사 데이터셋 65건(그라운딩 55 + 잡담용 no-pick 10), 스킵 49건은 주제형
  질의("좀비 영화", "요리 소재 영화")에서 태그 검색 후보와 교사 추천이
  어긋나는 한계 — TMDB keyword 태그 백필이 되면 개선될 영역
- EXAONE-3.5-2.4B fp16 + LoRA(r=16) 3에폭, **loss 0.88 → 0.65 → 0.51**
- gradient checkpointing을 plain 백엔드에도 공통 적용 — 실측 VRAM 7.9/8GB로
  아슬하게 수용 (없었으면 OOM)
- transformers 4.47의 `apply_chat_template` 반환형 차이(순수 텐서 vs
  BatchEncoding)는 `return_dict=True` 명시로 통일

## 결과

새 어댑터 반영 후 프로덕션 채팅 E2E:

> "긴장감 넘치는 범죄 스릴러 추천해줘"
> → **추격자(2008) · 범죄와의 전쟁(2012) · 범죄도시(2017)**

EC2 로그에 `POST https://lora.suvisdev.cloud/generate 200` — 깨진 제목 없이
장르에 정확히 맞는 그라운딩된 추천이 돌아왔다. LoRA가 죽어도 Gemini 자동
폴백(8/26 배선)이 받치고 있음을 이번 복구 과정에서 프로덕션 실호출로 실증한
것도 수확이다.

## 남은 것

- 프로덕션 리뷰 감정분석 백필 — Echo 어댑터 재학습 또는 이관 필요 (이 머신엔
  감정분석용 어댑터가 없다)
- 교사 데이터셋 확대 — 주제형 질의 스킵 49건은 TMDB keyword 태그 백필과 연계
- 구 `lora-notebook` 터널 정리
