# gf-autopilot

Claude Code 플러그인 — **git-flow 워크플로우 자동화**.

> A Claude Code plugin that automates git-flow workflows: feature, release, and hotfix branches with consistent naming and merge orchestration.

---

## What it does

gf-autopilot은 git-flow 브랜칭 모델을 Claude Code 안에서 일관되게 사용하도록 돕는 스킬 모음.

---

## Prerequisites

- [git](https://git-scm.com/)
- [git flow (AVH edition)](https://github.com/petervanderdoes/gitflow-avh)
- [GitHub CLI (`gh`)](https://cli.github.com/)

---

## Installation

이 저장소가 GitHub에 호스팅되어 있다고 가정합니다 (`https://github.com/paraang/gf-autopilot`).

```bash
# 1) 마켓플레이스 등록 (한 번만)
/plugin marketplace add paraang/gf-autopilot

# 2) 플러그인 설치
/plugin install gf-autopilot@gf-autopilot
```

업데이트:

```bash
/plugin marketplace update gf-autopilot
/plugin update gf-autopilot@gf-autopilot
```

## Getting Started

```
$ gf-init
```

## The sprint

```
                                 ↗ pr 리뷰 (w/ claude code) ↘
feature start → commit → pr 생성                               merge
                                 ↘ pr 리뷰 (w/ co-workers)  ↗


feature start → commit → feature finish
```

| Skill           | 역할                                                                                                                                                                                                                           |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `/gf-init`      | `gf-autopilot`을 사용하기 위한 선행조건 검사. git-flow 컨벤션 적용(main/develop + 표준 접두사) + `.gitignore` 의 `pr-review-*` 패턴 + `.github/PULL_REQUEST_TEMPLATE.md` 표준 양식 자동 등록 + 종료 시 `develop` 으로 체크아웃 |
| `/gf-pr`        | 현재 feature 브랜치를 `develop` 위로 rebase·squash·force-with-lease push 후 `gh pr create` 로 PR 생성. PR 본문은 PR 템플릿 양식 그대로 prefill, 머지는 `--no-ff` (merge commit) 권장                                           |
| `/gf-pr-review` | GitHub PR URL 입력 → 변경 파일·호출자·테스트까지 읽고 7개 축·4단계 심각도로 분석 → `event=COMMENT` 로 자동 게시 (사용자 확인 게이트 없음). `APPROVE`/`REQUEST_CHANGES` 는 자동 발행하지 않음                                   |
| `/gf-release`   | 사용자에게 새 버전 번호를 받아 메타파일(`plugin.json`, `marketplace.json` 의 `source.ref` 등) bump + `CHANGELOG.md` 작성 (Keep a Changelog 1.1.0) + `git flow release start/finish` + 태그 + push 까지 일괄                    |
| `template`      | 새 스킬 작성용 참조 양식 (frontmatter 스펙·본문 5섹션 가이드·완성 예시). 동작하지 않으며 `skills/template/` 를 복사해 새 스킬을 시작합니다                                                                                     |

---

## License

MIT — `LICENSE` 파일 참고.
