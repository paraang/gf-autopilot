---
name: gf-release
description: Use when the user asks to "release a new version", "git flow release", "버전 릴리즈", "릴리즈 진행", "새 버전 배포", "publish release", "cut a release", or wants to ship the current develop state as a new tagged version. Asks the user for the target version (mandatory, never inferred), validates it against existing tags' convention, detects the project type (node/maven/gradle/go/etc.) and syncs version strings into the relevant project files, writes a CHANGELOG.md entry in Keep a Changelog 1.1.0 style (with an optional 배포자 메모 section), then runs git flow release start/finish to create the merge commits and tag.
version: 0.1.0
allowed-tools: [Bash, Read, Edit, Write]
---

# gf-release

## Purpose

git-flow 의 `release start` → `release finish` 흐름을 한 번에 자동화합니다.

- 사용자에게 새 버전 번호를 **반드시** 묻습니다 (자동 추론 금지).
- 기존 태그들의 컨벤션과 비교해 일관성을 검증합니다 (다르면 사용자에게 재확인).
- 프로젝트 타입(Node/Maven/Gradle/Go/Rust/Python/Claude Code plugin 등)을 감지해 관련 메타파일의 버전을 동기화합니다.
- `CHANGELOG.md` 를 [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/) 형식으로 작성/갱신합니다.
- 사용자가 메모를 전달했으면 "배포자 메모" 섹션을 함께 기록합니다.
- 마지막에 `git flow release finish` 로 main 머지 + annotated 태그 + develop 백머지 + release 브랜치 삭제까지 수행합니다.

## When to use

- "릴리즈 진행", "release 0.2.0 만들어줘", "git flow release", "publish new version", "cut a release"
- `develop` 의 변경사항을 안정 버전으로 묶어 `main` 으로 내보낼 때

**Anti-pattern**

- 운영 중인 버전의 긴급 결함 수정은 `gf-hotfix` 가 담당합니다. 이 스킬은 그 용도가 아닙니다.
- 이미 `release/*` 브랜치가 있으면 새로 시작하지 말고, 기존 브랜치 마무리(테스트/문서) 후 finish 만 진행합니다.

## Inputs

- 사전조건
  - **`git flow` AVH edition 필수** (gf-init 사전 실행 권장)
  - 현재 브랜치는 보통 `develop`. 다르면 사용자에게 명시적으로 확인.
  - 작업 트리 clean (`git status --porcelain` 빈 출력)
- 인자: 없음. 모두 대화형 수집.
- 사용자에게 받을 정보
  1. **버전 번호** (필수) — 예: `0.2.0`, `1.0.0-rc.1`
  2. **배포자 메모** (선택) — 자유 텍스트. 없으면 CHANGELOG 의 "배포자 메모" 섹션을 통째로 생략.
- 권장 버전 컨벤션
  - SemVer (Major.Minor.Patch)
  - 접두사 없음 — `gitflow.prefix.versiontag` 가 빈 값이라는 gf-init 컨벤션과 일치 (예: `0.2.0`, not `v0.2.0`)
  - pre-release / build metadata 허용: `1.0.0-rc.1`, `1.0.0+build.42`

## Steps

1. **사전 점검 (Preflight)**

   ```bash
   git flow version                   # 출력에 "AVH Edition" 포함
   git rev-parse --is-inside-work-tree
   git status --porcelain             # 비어있어야 함
   git branch --show-current          # develop 권장
   git config gitflow.branch.develop  # gf-init 흔적 확인
   ```

   - AVH edition 미설치 시 즉시 중단하고 `gf-init` 의 설치 안내를 안내합니다.
   - 작업 트리가 dirty 면 중단하고 stash/commit 안내.
   - 현재 브랜치가 `develop` 이 아니면 사용자에게 확인 (release 시작은 보통 develop 기준).
   - 이미 `release/*` 브랜치가 존재하면 새로 시작하지 말고 기존 브랜치 finish 옵션만 안내.

2. **사용자에게 버전 번호 요청 (필수)**

   - AskUserQuestion 또는 명확한 평문 질문으로:
     > "릴리즈할 버전 번호를 알려주세요. (SemVer 권장, 예: 0.2.0)"
   - **자동 추론 금지**. 답이 없거나 비어있으면 중단.

3. **버전 컨벤션 검증**

   ```bash
   git tag -l --sort=-v:refname | head -10
   ```

   - 직전 태그 N개를 보고 컨벤션 패턴을 추출:
     - SemVer 형식 일관성
     - 접두사(`v`, 없음 등) 일관성
     - pre-release 표기 형식 일관성
   - 사용자가 입력한 버전이 그 패턴과 다르면:
     - **차이 표**를 사용자에게 보여주고
     - "기존 컨벤션은 `<X>` 형식인데 새 버전 `<Y>` 가 다릅니다. 그대로 진행할까요?" 재확인
     - 동의 없이 강제 진행 금지.
   - 단조 증가 검증: 입력 버전이 직전 최신보다 낮으면 경고하고 재확인.
   - 동일 태그가 이미 존재하면 즉시 중단 (force 삭제 금지).

4. **사용자에게 배포자 메모 요청 (선택)**

   - "이번 릴리즈에 추가할 배포자 메모가 있다면 알려주세요. 없으면 그냥 엔터/생략하셔도 됩니다."
   - 입력이 있으면 CHANGELOG.md 의 "### 배포자 메모" 섹션에 줄 단위로 삽입.
   - 입력이 없으면 해당 섹션을 통째로 생략 (빈 헤더만 남기지 않음).

5. **프로젝트 타입 감지 + 버전 동기화 대상 식별**

   저장소 루트에서 아래 마커 파일을 글로빙해 감지합니다.

   | 마커 파일                              | 프로젝트 타입            | 버전 위치                                                   |
   | -------------------------------------- | ------------------------ | ----------------------------------------------------------- |
   | `package.json`                         | Node.js                  | `version` 필드                                              |
   | `package-lock.json` / `yarn.lock`      | Node.js (lock)           | lockfile 동기화 명령 별도 안내                              |
   | `pom.xml`                              | Maven                    | `<project>/<version>` (의존성 `<version>` 과 혼동 주의)     |
   | `build.gradle` / `build.gradle.kts`    | Gradle                   | `version = '...'` 또는 `version = "..."`                    |
   | `gradle.properties`                    | Gradle (alt)             | `version=...` 라인                                          |
   | `go.mod`                               | Go                       | (보통 git 태그가 진실 — 파일 수정 없음. 사용자에게 안내)    |
   | `Cargo.toml`                           | Rust                     | `[package]` 섹션의 `version`                                |
   | `pyproject.toml`                       | Python (Poetry / PEP621) | `[tool.poetry] version` 또는 `[project] version`            |
   | `setup.py`                             | Python (legacy)          | `version=...`                                               |
   | `composer.json`                        | PHP                      | `version` (선택적, 보통은 태그 기반)                        |
   | `.claude-plugin/plugin.json`           | Claude Code plugin       | `version` 필드                                              |
   | `.claude-plugin/marketplace.json`      | Claude Code marketplace  | plugin entry 의 `version` **및** `source.ref` (둘 다 bump)  |
   | `CHANGELOG.md`                         | (어떤 프로젝트든)        | 8번 단계에서 별도 갱신                                      |

   - 발견된 파일 목록을 사용자에게 표로 보여주고 "다음 파일들의 버전을 함께 갱신할까요?" 확인.
   - 일부만 원하면 사용자가 선택. 동의 없이 임의 추가 금지.
   - **Claude Code 플러그인 프로젝트**: `marketplace.json` 의 `source.ref` 도 새 버전 태그로 같이 bump (그래야 새 install 이 새 태그를 받음).

6. **release 브랜치 시작**

   ```bash
   git flow release start <version>
   ```

   - 실패 시 (예: 이미 release/\* 있음) 사용자 안내 후 중단.

7. **버전 파일들 수정 + commit**

   - 5번에서 합의된 파일들의 버전 문자열을 새 값으로 정확히 1곳씩 교체.
     - **JSON / TOML / YAML**: 가능하면 파서 기반 수정. 텍스트 치환 시 첫 매칭만 신중히.
     - **`pom.xml`**: 의존성의 `<version>` 과 혼동하지 않도록 XPath `/project/version` 만 변경.
     - **Gradle**: 최상위 `version = '...'` 만. subproject 별도 처리 필요 시 사용자 확인.
   - **반드시 변경 diff 를 사용자에게 먼저 보여준 뒤 commit**:
     ```bash
     git diff --stat
     git diff
     ```
   - commit:
     ```bash
     git add <files>
     git commit -m "chore(release): bump version to <version>"
     ```

8. **CHANGELOG.md 작성/갱신**

   위치: 저장소 루트 `CHANGELOG.md`. 없으면 헤더와 함께 새로 생성.

   - **형식**: [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/)
   - **새 entry 는 `## [Unreleased]` 다음, 즉 최상단 버전 섹션으로 삽입**.
   - 기존에 `## [Unreleased]` 가 있다면 그 안의 카테고리 항목들을 새 버전 섹션의 적절한 카테고리로 **이동**시키고, `[Unreleased]` 는 빈 헤더만 남깁니다.
   - 변경 내역 보조 수집: 직전 태그 이후 commit 메시지 분석
     ```bash
     git log --no-merges --pretty=format:"%s" <prev>..HEAD
     ```
     - Conventional Commits prefix 가 있으면 그대로 카테고리 추정: `feat:`→Added, `fix:`→Fixed, `perf:`/`refactor:`→Changed, `docs:`→Changed(문서), `chore:`→스킵, `BREAKING CHANGE:`→Removed/Changed + major bump 경고.
     - **prefix 없는 비-Conventional 메시지도 자동 추정**해서 가장 가까운 카테고리에 배치. 사용자에게 분류를 묻지 말 것. 추정 불가능한 항목은 `Changed` 로 기본 배치.
   - **작성 톤 (중요)**: CHANGELOG 는 사용자 관점의 release note 이지 commit log 사본이 아닙니다.
     - **기능적 변경사항(사용자 / API / 동작 / 설정의 가시적 변화)에 집중** 합니다.
     - 주석 추가, 변수 rename, 포매팅, 사소한 리팩토링, 내부 헬퍼 정리 등 **사용자/API 가시성이 없는 변경은 한두 줄로 묶어 요약**하거나, 의미가 약하면 **생략** 합니다.
     - 같은 종류의 자잘한 변경 여러 건은 한 줄로 합치세요 (예: "내부 주석 및 변수명 정리").
     - 줄당 1줄, 동사로 시작, 끝마침 없음.
   - 사용자가 4번에서 메모를 전달했으면 "### 배포자 메모" 섹션을 최상단에 추가. 없으면 섹션 생략.
   - 첫 릴리스이고 `CHANGELOG.md` 가 없으면 헤더도 함께 생성 (아래 양식 예시 참고).
   - GitHub 원격이 있을 때만 하단 비교 링크 자동 갱신:
     ```
     [Unreleased]: https://github.com/<owner>/<repo>/compare/<version>...HEAD
     [<version>]: https://github.com/<owner>/<repo>/compare/<prev>...<version>
     ```
   - commit:
     ```bash
     git add CHANGELOG.md
     git commit -m "docs(changelog): add <version> entry"
     ```

9. **release finish**

   ```bash
   GIT_MERGE_AUTOEDIT=no git flow release finish -m "Release <version>" <version>
   ```

   자동 수행:

   - `release/<version>` → `main` 머지 (no-ff)
   - annotated 태그 `<version>` 생성 (메시지: "Release \<version>")
   - 태그 → `develop` 백머지
   - `release/<version>` 로컬 브랜치 삭제

   실패 시 release 브랜치를 **남겨두고** 사용자에게 수동 복구 안내. 강제 삭제/리셋 금지.

10. **push (사용자 확인 후)**

    ```bash
    git remote get-url origin
    git push origin main develop <version>
    ```

    - 푸시할 ref 목록을 먼저 사용자에게 제시하고 동의 후 실행.
    - 동의 없이 강제 푸시 금지.
    - 원격이 없으면 push 단계를 건너뛰고 사용자에게 원격 추가 후 재시도 안내.

11. **검증 + 보고**

    ```bash
    git tag -l --sort=-v:refname | head -5
    git log --oneline --decorate --all -10
    git rev-parse <version>
    ```

    사용자에게 다음을 요약 보고:

    - 새 태그 이름 + 가리키는 commit SHA
    - 동기화된 파일 목록 + 각 파일의 새 버전 값
    - CHANGELOG.md 에 새로 추가된 섹션 발췌
    - `main` / `develop` 의 최신 commit SHA
    - 푸시 결과 (성공 ref / 스킵 사유)

## Output

- 새 태그 이름과 commit SHA
- 동기화된 버전 파일 목록 (diff 발췌)
- CHANGELOG.md 신규 entry 텍스트
- `main`, `develop` 최신 commit SHA
- push 결과
- 다음 액션 제안 (예: "이제 `develop` 에서 다음 작업을 이어갈 수 있습니다.")

## Notes (구현 시 주의)

- **버전은 반드시 사용자 입력**: 자동 추론(minor +1 등) 절대 금지. 직전 태그를 *제안값* 으로 보여주는 것은 OK, 그러나 입력 없이 진행 금지.
- **컨벤션 검증은 거부가 아니라 확인**: 다르더라도 사용자가 의도하면 진행. 단 침묵 금지.
- **버전 파일 수정 전 diff 공개**: 어떤 파일의 어떤 라인이 바뀌는지 사용자가 보고 OK 한 뒤 commit.
- **이미 존재하는 태그면 중단**: force-delete 금지.
- **release 브랜치 충돌 시**: 새로 시작하지 말고 기존 finish 만 진행하도록 안내.
- **finish 실패 시 정리**: release 브랜치 유지 + 사용자에게 수동 복구 안내. 자동 리셋/리베이스 금지.
- **CHANGELOG 카테고리 자동 분류는 자율 수행**: Conventional 이든 아니든 알아서 추정·분류해서 작성합니다. **사용자에게 카테고리 분류를 묻지 않습니다.** (대신 작성된 CHANGELOG diff 를 사용자에게 보여줘 사후 수정 기회는 제공합니다.)
- **CHANGELOG 작성 톤**: 기능적 변경 중심. 주석/포매팅/사소한 내부 정리 같은 사용자/API 가시성 없는 변경은 묶어서 한 줄로 요약하거나 생략합니다.
- **lock 파일 (`package-lock.json`, `yarn.lock`, `Cargo.lock`, `poetry.lock` 등)**: 본 스킬은 직접 수정하지 않습니다. 동기화 명령(`npm install --package-lock-only` 등)을 사용자에게 제안만 합니다.
- **`go.mod`**: 일반적으로 git 태그가 단일 진실. 파일 수정 없음. 사용자에게 명시.
- **`marketplace.json` 의 `source.ref` bump**: Claude Code 플러그인 프로젝트에서만 적용. 다른 프로젝트엔 해당 없음.

## CHANGELOG.md 양식 예시 (참고)

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.0] - 2026-05-21

### 배포자 메모

- gf-init 후 develop 으로 자동 체크아웃 추가
- 마켓플레이스 source 를 HTTPS url 형식으로 통일

### Added

- gf-init: 초기화 종료 시 `develop` 체크아웃

### Changed

- marketplace.json source 를 `url` + https URL 로 표준화

### Fixed

- SSH 키 없는 사용자가 install 시 발생하던 인증 오류

[Unreleased]: https://github.com/paraang/gf-autopilot/compare/0.2.0...HEAD
[0.2.0]: https://github.com/paraang/gf-autopilot/compare/0.1.1...0.2.0
```
