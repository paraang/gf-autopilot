---
name: gf-init
description: Use when the user asks to "initialize git flow", "git flow init", "gitflow 초기화", "프로젝트 git flow 설정", "set up git flow on this project", or starts a new repository that needs git-flow conventions. REQUIRES git-flow AVH edition (latest) — if missing or a non-AVH edition, the skill MUST stop and tell the user to install it. No plain-git fallback. Runs git init if needed, ensures an initial commit on main, and configures git-flow with the recommended branch names and prefixes (main/develop, feature/, bugfix/, release/, hotfix/, support/, empty version tag). If git-flow is already configured, compares the current config to the recommendation and asks the user before overwriting.
version: 0.1.0
allowed-tools: [Bash, Read]
---

# gf-init

## Purpose

새 프로젝트 또는 기존 저장소에 git-flow 브랜칭 모델을 일관되게 적용합니다.

- 저장소가 없으면 `git init -b main` 으로 초기화
- 빈 저장소면 첫 커밋(empty commit)을 만들어 git-flow가 동작하도록 준비
- gf-autopilot 권장 설정으로 git-flow 구성
- 이미 설정되어 있으면 현재 값과 권장 값을 비교해 사용자에게 보고하고, **동의 없이는 덮어쓰지 않음**

## When to use

- "git flow init", "gitflow 초기화", "프로젝트 git flow 설정해줘", "set up git flow" 같은 요청
- 새 프로젝트를 만들고 git-flow를 곧바로 쓰려는 상황
- 기존 git-flow 설정이 권장 규약과 일치하는지 확인이 필요한 경우

**Anti-pattern**

- 이미 다른 컨벤션(trunk-based, GitHub Flow 등)으로 운영 중인 저장소에 사용자 확인 없이 적용하지 마세요.
- 이 스킬은 브랜치 생성/머지 등 실제 git-flow 워크플로우를 실행하지 않습니다. 그건 별도 스킬 (`/gf-pr`, `/gf-release`) 이나 `git flow feature|release|hotfix` 명령의 영역입니다.

## Inputs

- 사전조건
  - `git` 설치 필수
  - **`git flow` (git-flow AVH edition 최신 버전) 설치 필수**. 없거나 비-AVH edition이면 스킬은 즉시 중단하고 설치를 요청합니다. **`git config` 만으로 우회하지 않습니다** — gf-autopilot의 후속 스킬 (`/gf-release`, `/gf-pr` 등) 과 `git flow` CLI 직접 호출이 모두 AVH edition 에 의존하기 때문입니다.
- 인자: 없음
- 권장 설정 값

  | 키                            | 값                  |
  | ----------------------------- | ------------------- |
  | `gitflow.branch.master`       | `main`              |
  | `gitflow.branch.develop`      | `develop`           |
  | `gitflow.prefix.feature`      | `feature/`          |
  | `gitflow.prefix.bugfix`       | `bugfix/`           |
  | `gitflow.prefix.release`      | `release/`          |
  | `gitflow.prefix.hotfix`       | `hotfix/`           |
  | `gitflow.prefix.support`      | `support/`          |
  | `gitflow.prefix.versiontag`   | `` (빈 값)          |
  | `gitflow.path.hooks`          | `.git/hooks` (기본) |

  > `gitflow.path.hooks` 는 git-flow의 기본값과 동일하므로 명시적으로 설정하지 않습니다. 사용자가 별도 위치를 원할 때만 설정합니다.

## Steps

1. **사전 점검 (Preflight) — 게이트, 통과 못 하면 즉시 중단**

   ```bash
   git --version
   git flow version            # 출력에 "AVH Edition" 문자열이 반드시 포함되어야 함
   git rev-parse --is-inside-work-tree 2>/dev/null
   ```

   - `git` 이 없으면 즉시 중단하고 설치 안내.
   - **`git flow` 명령이 없거나, 출력에 `AVH Edition` 이 보이지 않으면 스킬을 즉시 중단**하고 아래 안내를 출력합니다. `git config` 우회는 하지 않습니다.

     **git-flow AVH edition 설치 안내 (최신 버전 필수)**

     | OS / 환경                  | 설치 명령                                                                                                |
     | -------------------------- | -------------------------------------------------------------------------------------------------------- |
     | macOS (Homebrew)           | `brew install git-flow-avh`                                                                              |
     | Windows (Scoop)            | `scoop install gitflow`                                                                                  |
     | Windows (Chocolatey)       | `choco install gitflow-avh`                                                                              |
     | Windows (Git for Windows)  | Git for Windows 2.5.3 이상에 AVH edition이 기본 포함 — 최신 Git for Windows 설치/업데이트                |
     | Debian/Ubuntu              | `sudo apt-get install git-flow` (배포판 패키지가 비-AVH이면 아래 공식 스크립트 사용)                     |
     | Fedora/RHEL                | `sudo dnf install gitflow`                                                                               |
     | Unix 공식 설치 스크립트    | `curl -fsSL https://raw.githubusercontent.com/petervanderdoes/gitflow-avh/develop/contrib/gitflow-installer.sh \| bash` |

     공식 저장소: https://github.com/petervanderdoes/gitflow-avh

     설치 후 새 셸을 열어 `git flow version` 출력에 `AVH Edition` 이 보이는지 확인한 뒤 이 스킬을 다시 실행하도록 안내합니다.

2. **git 저장소 초기화 (필요 시)**

   - `is-inside-work-tree` 가 false / 에러면:
     ```bash
     git init -b main
     ```
   - 커밋이 0개이면 (`git rev-parse HEAD` 실패) 초기 빈 커밋 생성:
     ```bash
     git commit --allow-empty -m "chore: initial commit"
     ```
   - 현재 브랜치가 `master` 인 경우 사용자에게 묻고 동의 시 `git branch -m master main` 으로 rename.

3. **git-flow 설정 상태 점검**

   ```bash
   git config --local --get-regexp '^gitflow\.' || true
   ```

   - 결과가 비어있으면 → **분기 A** (4번)
   - 결과가 있으면 → **분기 B** (5번)

4. **분기 A — 미설정: 권장 설정 적용**

   ```bash
   git config gitflow.branch.master main
   git config gitflow.branch.develop develop
   git config gitflow.prefix.feature feature/
   git config gitflow.prefix.bugfix bugfix/
   git config gitflow.prefix.release release/
   git config gitflow.prefix.hotfix hotfix/
   git config gitflow.prefix.support support/
   git config gitflow.prefix.versiontag ""
   ```

   - `develop` 브랜치가 없으면 생성:
     ```bash
     git show-ref --verify --quiet refs/heads/develop || git branch develop main
     ```
   - 적용 결과 표를 사용자에게 출력.

5. **분기 B — 기존 설정 존재: 권장사항과 비교**

   - 8개 키(versiontag 포함) 각각에 대해 `git config --local --get <key>` 로 현재값 수집.
   - 권장값과 비교하여 **차이 표**를 작성:

     | 키 | 현재 값 | 권장 값 | 일치 |
     |---|---|---|---|
     | ... | ... | ... | ✅ / ❌ |

   - 모두 ✅ 면 "이미 권장 설정과 일치합니다." 보고 후 6번으로 건너뜀.
   - ❌ 가 하나라도 있으면 **사용자에게 명시적 선택을 요구**:
     - (A) **권장 설정으로 덮어쓰기** — 4번의 명령을 그대로 실행. 기존 브랜치는 그대로 두고 config만 갱신.
     - (B) **현재 설정 유지** — 변경 없이 종료. 차이 표만 기록.
     - (C) **부분 덮어쓰기** — 어떤 키만 덮어쓸지 사용자가 지정.
   - **중요: 사용자 확인 없이 자동 덮어쓰기 금지.** 항상 AskUserQuestion 등으로 명시적 동의를 받으세요.
   - 덮어쓰기 전 안전을 위해 `.git/config` 를 `.git/config.bak-<timestamp>` 로 복사 백업.

6. **검증 (Verification)**

   ```bash
   git config --local --get-regexp '^gitflow\.'
   git branch --list main develop
   git log --oneline -1 main
   git log --oneline -1 develop
   ```

   - 권장 키 8개가 모두 보이고, `main` / `develop` 브랜치가 존재하면 OK.

7. **`develop` 브랜치로 체크아웃 (next-release development)**

   git-flow 에서 다음 릴리스를 향한 모든 작업(feature/bugfix)은 `develop` 위에서 분기합니다. 초기화 직후 사용자가 곧바로 작업을 시작할 수 있도록 마지막에 `develop` 으로 체크아웃합니다.

   ```bash
   git checkout develop
   ```

   - 이미 `develop` 위에 있으면 no-op.
   - 작업 트리에 사용자가 만든 staged/unstaged 변경이 있어 체크아웃이 실패하면, 강제 전환하지 말고 사용자에게 stash 또는 commit 후 직접 체크아웃하도록 안내합니다.
   - 분기 B에서 사용자가 "현재 설정 유지" 를 선택했더라도 이 단계는 동일하게 수행합니다 (브랜치 위치는 설정과 무관).

8. **`.gitignore` 에 `pr-review-*` 패턴 추가**

   `/gf-pr-review` 스킬이 저장소 루트에 생성하는 `pr-review-<N>.md` / `pr-review-<N>.json` 리포트 파일이 실수로 commit 되지 않도록 사전에 `.gitignore` 에 등록합니다.

   추가할 라인 (정확히 이 두 줄만, 헤더 주석 포함):

   ```gitignore

   # gf-pr-review report files
   pr-review-*.md
   pr-review-*.json
   ```

   - `.gitignore` 가 없으면 위 내용으로 새 파일 생성.
   - 이미 있으면 동일 패턴이 존재하지 않을 때만 append (중복 라인 만들지 말 것). 예:
     ```bash
     grep -qxF 'pr-review-*.md' .gitignore || printf '\n# gf-pr-review report files\npr-review-*.md\npr-review-*.json\n' >> .gitignore
     ```
   - 변경이 발생하면 사용자에게 알리고 다음 단계 안내와 함께 commit 여부를 사용자 결정에 맡깁니다. 자동 commit 금지.

9. **다음 단계 안내**

   - "현재 브랜치는 `develop` 입니다. 이제 `git flow feature start <name>` 으로 새 기능을 시작할 수 있습니다."
   - 원격 저장소가 설정되어 있고(`git remote get-url origin` 성공) 두 브랜치가 푸시되지 않았다면, 사용자에게 `git push -u origin main develop` 실행 여부를 묻습니다(강제 실행 금지).
   - 8번에서 `.gitignore` 가 수정되었으면 같이 commit 할지 사용자에게 묻습니다 (예: `chore: ignore pr-review-* reports`).

## Output

- 적용된 권장 설정 표 (분기 A) 또는 차이 표 + 사용자 선택 결과 (분기 B)
- 변경된 항목 / 변경되지 않은 항목 요약
- 현재 브랜치 상태 (`main`, `develop` 존재 여부)
- **최종 체크아웃된 브랜치** (`develop` 이어야 정상)
- 백업 파일 경로 (분기 B에서 덮어쓰기 한 경우)
- 다음 액션 제안

## Notes (구현 시 주의)

- **git-flow AVH edition 강제**: 사전 점검(Step 1)에서 AVH edition이 확인되지 않으면 어떤 우회도 하지 말고 즉시 중단합니다. `git config` 만으로 brach prefix를 박아두면 사용자는 다음 스킬에서 `git flow feature start ...` 호출이 실패해 혼란을 겪습니다. gf-autopilot은 AVH CLI 존재를 단일 진실 공급원으로 가정합니다.
- **로컬 설정만 변경**: 모든 `git config` 명령은 `--local` 범위로 적용합니다 (`--global` 금지). 사용자의 다른 저장소에 영향을 주면 안 됩니다.
- **백업**: 분기 B에서 덮어쓰기 전 반드시 `.git/config` 백업.
- **`git flow init -d` 를 쓰지 않는 이유**: 기본값이 `master` 이고 인터랙티브 입력을 가정하기 때문. 명시적 `git config` 명령이 더 안전하고 재현 가능.
- **OS 호환성**: 위 명령은 Bash / PowerShell 모두에서 동일하게 동작합니다. `versiontag` 의 빈 문자열은 `""` 그대로 사용해도 양쪽 셸에서 작동합니다.
- **dry-run**: 사용자가 "dry-run" 또는 "미리보기" 를 요청하면 4번/5번의 `git config` 명령을 실행하지 말고 **실행될 명령 목록만 출력**합니다.
