# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.1] - 2026-05-22

### Changed

- `gf-pr`: PR 머지는 `--no-ff` (merge commit 생성) 방식만 허용하도록 정책 명시. Squash/Rebase merge 금지 — 본 스킬이 push 전에 이미 squash 하므로 추가 squash 는 무의미하고, merge commit 으로 git-flow 분기점을 보존해야 함
- `gf-init`: 초기화 시 `.gitignore` 에 `pr-review-*.md`, `pr-review-*.json` 패턴을 자동 등록하는 단계 추가 (`/gf-pr-review` 리포트 파일이 실수로 commit 되지 않도록)
- `gf-pr-review`: 사용자 확인 게이트 제거 — 항상 `event=COMMENT` 로 자동 게시. `APPROVE`/`REQUEST_CHANGES` 는 자동 발행하지 않고 추천 등급만 보고에 표시

## [0.2.0] - 2026-05-21

### Added

- `gf-release` 스킬 — git-flow release start/finish 자동화 (프로젝트 타입별 버전 파일 동기화 + Keep a Changelog 1.1.0 형식 CHANGELOG 작성)
- `gf-pr` 스킬 — feature 브랜치를 develop 위로 rebase·squash·force-with-lease push 후 `gh pr create` 로 PR 생성, `.github/PULL_REQUEST_TEMPLATE.md` 보장 포함
- `gf-pr-review` 스킬 — GitHub PR URL 입력 시 깊이 있는 분석(7개 축, 4단계 심각도) 후 사용자 동의 시 `gh api` 로 inline 리뷰 게시
- README: 사전 요구사항 절 (git / git flow AVH edition / GitHub CLI)

### Changed

- `gf-init`: 초기화 종료 시 `develop` 브랜치로 자동 체크아웃
- 스킬·문서 전반의 명명 일관성 정리 (`_template` → `template`, 존재하지 않는 스킬명 참조 제거)

[Unreleased]: https://github.com/paraang/gf-autopilot/compare/0.2.1...HEAD
[0.2.1]: https://github.com/paraang/gf-autopilot/compare/0.2.0...0.2.1
[0.2.0]: https://github.com/paraang/gf-autopilot/compare/0.1.1...0.2.0
