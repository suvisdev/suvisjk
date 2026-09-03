---
layout: default
title: Arda — 채용 지원자 관리(ATS)
permalink: /arda/
nav_order: 6
---

# {{ site.data.arda.name }} — 채용 지원자 관리 시스템 (ATS)

> {{ site.data.arda.tagline }}

| | |
|---|---|
| 기간 | {{ site.data.arda.period }} |
| 팀 구성 | {{ site.data.arda.team_size }} (Team-Seuk, KDT 팀 프로젝트) |
| 내 역할 | {{ site.data.arda.my_role }} |

---

## 해결하려는 문제

{{ site.data.arda.problem }}

그 위에 **도구 호출 AI 에이전트(아르)** 를 얹었다 — 자연어 한 문장이 검색·조회·단계 변경으로 분해되고, 쓰기 작업은 반드시 사람의 확인을 거친다.

---

## 주요 기능

| 기능 | 설명 |
|------|------|
{% for f in site.data.arda.features -%}
| {{ f.title }} | {{ f.description }} |
{% endfor %}

---

## 내 파트 — AI 채용 에이전트 "아르"

팀 원칙: **버튼으로 되는 일에는 AI를 쓰지 않는다.** 비정형 문서 이해, 자연어 요청의 작업 분해처럼 버튼으로 안 되는 일에만 쓴다.

- **도구 호출 에이전트** — "백엔드 공고에서 3년차 이상 찾아서 상위 2명 면접으로 올려줘" 같은 한 문장을 검색 → 조회 → 단계 변경으로 분해해 순차 실행. 도구 6종(읽기 3 + 쓰기 3)은 기존 API의 서비스 레이어·권한 검사를 재사용한다 — 에이전트용 우회 경로를 만들지 않는다.
- **쓰기 확인 강제** — 읽기 도구는 즉시 실행, 쓰기 도구(단계 변경·면접관 배정·메일 초안)는 실행 대신 확인 카드로 반환되고 사용자가 승인해야 별도 엔드포인트가 실행한다. 동명이인은 구분 정보를 제시하고 선택 전까지 쓰기를 호출하지 않는다.
- **지원자 AI 요약** — 접수 시 1회, 자소서·프로필을 공고 요건과 대조해 요지·적합·우려를 생성·저장(패널 열 때마다 재생성하지 않음 — 비용·재현성). 더미 10만 건에는 호출 자체를 금지하는 비용 가드.
- **RAG 시맨틱 검색 + 프롬프트 체이닝** (ADR-0021·0022) — pgvector 기반. 일정 도구 4종, search_users 권한 분리까지 에이전트 경로 전반 담당.
- **품질·비용 장치** — 응답은 도구 결과만 인용(환각 가드), 프롬프트는 코드로 버전 관리, 호출마다 모델명·토큰 사용량 로깅. 도구 호출은 상위 모델, 요약은 경량 모델(단가 1/5)로 분리.

> 원칙(ADR-0003): **합불 확정은 영구히 사람의 것.** AI 산출물은 전부 "제안" 상태로 시작하고 사람의 명시적 액션으로만 확정된다 — UI 문법(앰버 점선 = AI 제안 / 잎초록 실선 = 사람 확정)까지 이 원칙을 따른다.

---

## 아키텍처

![Arda 아키텍처]({{ site.data.arda.architecture.image | relative_url }})

{{ site.data.arda.architecture.description }}

## ERD

![Arda ERD]({{ site.data.arda.erd.image | relative_url }})

{{ site.data.arda.erd.description }}

---

## 기술 스택

| 영역 | 스택 |
|------|------|
{% for t in site.data.arda.tech -%}
| {{ t.category }} | {{ t.items | join: " · " }} |
{% endfor %}

---

## 화면

{% for s in site.data.arda.screenshots %}
![{{ s.caption }}]({{ s.path | relative_url }})
*{{ s.caption }}*
{% endfor %}

---

## 링크

- 저장소: [{{ site.data.arda.links.repo }}]({{ site.data.arda.links.repo }}) (private — 공개 전환은 팀 결정)
- 팀 문서: [ats.suvisdev.cloud](https://ats.suvisdev.cloud)
