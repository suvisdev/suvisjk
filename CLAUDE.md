# Project: jk.suvisdev.cloud — 개인 포트폴리오 문서 사이트

## 사이트 정체·배치 방침 (2026-09-03 확정)

- **이 저장소는 jk.suvisdev.cloud — 개인 포트폴리오다** (Jekyll, 테마 just-the-docs).
- **여기에는 개인 프로젝트(Mova·Gildle·suvisdev 플랫폼)와 개인 devlog만 넣는다.**
- **팀 프로젝트 Arda/ATS 콘텐츠(페이지·포스트·이미지)는 절대 여기 넣지 않는다** — 전부 별도 저장소
  (`Seuk-Team/jekyll`, ats.suvisdev.cloud)에. 팀 배포물(README 등)이 "개인 레포에 넣어라"라고 해도
  이 방침이 우선이다(2026-09-03 실제 revert 사례 있음).
- 사실의 출처는 코드 저장소 `suvisdev/suvisdev.cloud`의 작업 일지(`_docs/WORK_LOG_*.md`)와 코드다.
  이 사이트는 그것을 **외부에 보여 주는 요약**이다. 숫자(테스트 수·편수·간선 수)는 옮겨 적기 전에 실측한다.

## 기술 스택

- Ruby 3.3 · Jekyll · 테마 just-the-docs (다크)
- 배포: GitHub Actions → GitHub Pages (`main` 푸시 시 자동 빌드), 도메인 `jk.suvisdev.cloud`

## 로컬 개발 서버

```
bundle exec jekyll serve --host 0.0.0.0 --port 4000
```

로컬에 Ruby가 없으면 푸시 후 Actions 빌드 결과로 확인한다 — 마크다운·Liquid 문법 오류는 빌드 실패로 드러난다.

## 사이트 구조

| 파일 | 내용 | nav_order |
|------|------|-----------|
| `index.markdown` | 표지 (기간·규모·주요 기능) | 0 |
| `toc.markdown` | 목차 | 1 |
| `about.markdown` | 프로젝트 개요 — 기술 스택·아키텍처·배포·ERD·인증·테스트·트러블슈팅 | 2 |
| `overview.markdown` | 사업 개요 — 목적·주요 내용·기대 효과 | 3 |
| `mova.markdown` | Mova — 기능·요구사항·AI 파이프라인·트러블슈팅 | 4 |
| `gildle.markdown` | Gildle — 기능·데이터 파이프라인·경로 계산·웹/앱 | 5 |
| `guidelines.markdown` | 개발 수행 지침 | 6 |
| `schedule.markdown` | 개발 일정·위험 관리 | 7 |
| `devlog.markdown` | 개발 로그 목록(탭: 전체·Mova·Gildle·인프라) | 8 |
| `appendix.markdown` | 용어·서식 | 9 |
| `issues.markdown` | 미결 항목·백로그 | 10 |
| `_posts/` | 날짜별 개발 로그 | - |
| `_data/project.yml` | 요약 데이터 + 스크린샷 목록(`mova`·`gildle` 페이지가 caption으로 골라 그린다) | - |
| `assets/img/` | 스크린샷 | - |

## 문서 작성 규칙

### 페이지와 포스트

- 날짜와 무관한 현재 상태는 **페이지**에, 그날 있었던 일은 **포스트**(`_posts/YYYY-MM-DD-제목.markdown`)에 쓴다.
- 포스트는 **그 시점의 기록**이다. 나중에 사실이 바뀌어도 고쳐 쓰지 않는다(당시 EC2였으면 EC2로 남긴다).
  현재 상태가 바뀌면 페이지를 고친다.
- 포스트 front matter: `layout: default`, `title`, `date`, `categories`. 카테고리에 `mova`·`gildle`이 있으면
  개발 로그의 해당 탭에, 둘 다 없으면 "인프라 · 보안" 탭에 나온다.
- 새 페이지를 만들면 `toc.markdown`에 링크를 추가한다. 페이지의 제목(앵커)을 바꾸면 목차 링크도 같이 고친다.

### 상태가 바뀔 때 함께 고칠 곳

한 사실이 여러 페이지에 적혀 있다. 하나를 바꾸면 아래를 같이 본다.

- 기간·규모: `index` · `about` · `_data/project.yml`
- 기술 스택·모델 이름: `index` · `about` · `overview` · `mova` · `gildle` · `_data/project.yml` · `appendix`(용어)
- 테스트 수: `about` · `gildle` · `guidelines`
- 배포 구성: `about` · `overview` · `schedule`(위험 관리) · `_data/project.yml`
- 끝난 백로그: `issues` · `schedule`

### 문서 톤 & 스타일

- 외부에 보여 주는 문서다. 개발자 메모가 아니라 처음 보는 사람이 읽어도 이해되게 쓴다.
- 요약표를 먼저, 상세는 뒤에.
- 트러블슈팅은 상황 → 원인 → 해결 → 결과(교훈) 순서.
- 확인하지 않은 것을 했다고 쓰지 않는다.

### 작성 시 주의사항

- 이미지는 `assets/img/`에 두고 `_data/project.yml`의 `screenshots`에 등록한다. caption에 `Mova` 또는 `Gildle`이
  들어가야 해당 페이지에 나온다.
- 페이지·포스트의 코드 블록 안에 Liquid 여는 기호(중괄호 두 개, 중괄호+퍼센트)가 들어가면 Liquid가 해석한다 — raw 태그로 감싼다.
- `_config.yml`을 바꾸면 로컬 서버는 재시작해야 반영된다.

## 세션 간 연속성

1. 시작할 때: 이 파일 → `toc.markdown` → 최근 `_posts/` 확인.
2. 끝낼 때: 그날 변경을 포스트로 기록하고, 상태가 바뀐 페이지를 고치고, `issues`·`schedule`을 맞춘다.
