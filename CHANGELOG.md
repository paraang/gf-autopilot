# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.0] - 2026-05-21

### Added

- `gf-release` 스킬 — git-flow release start/finish 자동화 (프로젝트 타입별 버전 파일 동기화 + Keep a Changelog 1.1.0 형식 CHANGELOG 작성)
- `gf-pr` 스킬 — feature 브랜치를 develop 위로 rebase·squash·force-with-lease push 후 `gh pr create` 로 PR 생성, `.github/PULL_REQUEST_TEMPLATE.md` 보장 포함
- `gf-pr-review` 스킬 — GitHub PR URL 입력 시 깊이 있는 분석(7개 축, 4단계 심각도) 후 사용자 동의 시 `gh api` 로 inline 리뷰 게시
- README: 사전 요구사항 절 (git / git flow AVH edition / GitHub CLI)

### Changed

- `gf-init`: 초기화 종료 시 `develop` 브랜치로 자동 체크아웃
- 스킬·문서 전반의 명명 일관성 정리 (`_template` → `template`, 존재하지 않는 스킬명 참조 제거)

[Unreleased]: https://github.com/paraang/gf-autopilot/compare/0.2.0...HEAD
[0.2.0]: https://github.com/paraang/gf-autopilot/compare/0.1.1...0.2.0
