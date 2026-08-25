---
layout: default
title: "Mova AI 추천 파이프라인 안정화"
categories: [feature, mova, ai]
---

# Mova AI 추천 파이프라인 안정화

- LoRA(EXAONE-2.4B AWQ) + Gemini 듀얼 백엔드 추천 구조 확립
- `RECOMMENDATION_BACKEND` env 분기로 GPU 없는 EC2에서 Gemini 폴백
- lora_server 원격 GPU(집) → Cloudflare Tunnel → EC2 호출 구조
- Gemini 레이트 리밋 완화 + 재시도 로직
