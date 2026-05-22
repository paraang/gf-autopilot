---
name: gf-pr
description: Use when the user asks to "create a PR", "open a pull request", "PR 만들어줘", "feature finish 전에 PR 띄워줘", "rebase and squash 후 PR", "send the feature for review", or wants to ship a feature branch via pull request before running git flow feature finish. Updates develop, rebases the feature branch on top of it, squashes commits into one, force-with-lease pushes, ensures `.github/PULL_REQUEST_TEMPLATE.md` exists (creating it from the project's standard template if missing), then creates the pull request via GitHub CLI (`gh pr create`) and returns the PR URL to the user. If a rebase conflict appears, the skill stops and waits for the user to resolve it.
version: 0.1.0
allowed-tools: [Bash, Read, Edit, Write]
---

# gf-pr

## Purpose

`git flow feature finish` 로 곧장 develop 머지하기 전에, 변경사항을 **PR 단위로 리뷰** 받을 수 있도록 준비합니다.

- `develop` 을 원격 최신으로 끌어옴
- 현재 feature 브랜치를 그 위로 **rebase + squash (1 commit)** 처리
- 충돌 발생 시 즉시 중단하고 사용자 해결을 대기
- `.github/PULL_REQUEST_TEMPLATE.md` 가 없으면 표준 양식으로 생성
- **GitHub CLI (`gh pr create`)** 로 PR 생성 후 URL 을 사용자에게 안내

## When to use

- "PR 생성", "PR 띄우기", "리뷰 보낼게", "rebase and squash 후 PR"
- feature 작업을 마쳤지만 곧바로 develop에 머지하지 않고 리뷰 절차를 거치고 싶을 때

**Anti-pattern**

- 현재 브랜치가 `feature/*` 가 아닐 때는 사용하지 말 것. release/hotfix 브랜치는 PR 흐름이 다름.
- 작업 트리에 unstage/uncommit 변경이 있을 때는 그것부터 commit/stash 후 사용.

## Inputs

- 사전조건
  - `git flow` AVH edition (gf-init 사전 권장)
  - **GitHub CLI (`gh`) 설치 + 인증 완료** (`gh auth status` 성공)
  - 현재 브랜치가 `feature/<name>`
  - 작업 트리 clean
  - `origin` 원격 존재 (GitHub)
- 인자: 없음
- 사용자에게 받을 정보
  - squash commit 메시지 (commit 이 2개 이상이면 필수, 없으면 중단)
  - `.github/PULL_REQUEST_TEMPLATE.md` 신규 생성 시 동의

## Steps

1. **사전 점검**

   ```bash
   git flow version              # AVH Edition 확인
   gh --version                  # GitHub CLI 설치 확인
   gh auth status                # gh 인증 확인
   git status --porcelain        # 비어있어야 함
   git branch --show-current     # feature/* 여야 함
   git remote get-url origin     # GitHub URL
   ```

   조건 미충족 시 중단하고 사유 안내. `gh` 미설치 시 https://cli.github.com 설치 안내, `gh auth status` 실패 시 `gh auth login` 실행 안내 (스킬이 자동으로 인증 시도 금지).

2. **develop 최신화**

   ```bash
   git fetch origin
   git checkout develop
   git pull --ff-only origin develop
   ```

   - fast-forward 실패 시 (로컬 develop에 임의 commit 있음) 사용자에게 알리고 중단.

3. **feature 브랜치로 복귀**

   ```bash
   git checkout feature/<name>
   ```

4. **develop 위로 rebase**

   ```bash
   git rebase develop
   ```

   - **충돌 발생 시 즉시 중단**. 다음 정보를 사용자에게 출력:

     ```bash
     git status                                 # 충돌 파일 목록
     git diff --name-only --diff-filter=U
     ```

     안내:

     > "다음 파일에 충돌이 있습니다. 충돌을 해결한 뒤 `git add <files> && git rebase --continue` 를 실행하거나, 중단하려면 `git rebase --abort` 를 실행하세요. 해결이 끝나면 다시 이 스킬을 실행해 주세요."

   - **사용자 동의 없이 자동 충돌 해결 금지** (theirs/ours 선택 등).
   - rebase 완료 후 `git log --oneline develop..HEAD` 로 결과 commit 목록 확인.

5. **squash (commit 1개로 합치기)**

   ```bash
   N=$(git rev-list --count develop..HEAD)
   ```

   - `N == 1`: squash 생략. 기존 commit 메시지를 그대로 PR 제목 후보로 사용.
   - `N >= 2`:
     - 기존 commit 메시지 목록을 사용자에게 보여줌:
       ```bash
       git log --no-merges --pretty=format:"- %s" develop..HEAD
       ```
     - 사용자에게 **squash 후 한 줄 commit 메시지** 입력 요청 (이 메시지가 PR 제목 후보가 됨).
     - 사용자 입력 없으면 중단.
     - 입력 받은 뒤:
       ```bash
       git reset --soft "$(git merge-base develop HEAD)"
       git commit -m "<사용자 입력>"
       ```

6. **force-with-lease push**

   rebase 로 history 가 바뀌었으므로 일반 push 는 거부됩니다. `--force-with-lease` 는 원격이 우리 마지막으로 본 상태와 다르면 거부해 안전합니다.

   ```bash
   git push --force-with-lease origin feature/<name>
   ```

   - 거부되면 (원격에 다른 협업자 commit 있음) 즉시 중단하고 사용자에게 `git fetch` 후 상황 확인 안내. **`--force` 강행 금지.**

7. **PR 템플릿 보장**

   다음 위치 중 어느 하나라도 있으면 OK (GitHub 가 인식):
   - `.github/PULL_REQUEST_TEMPLATE.md`
   - `docs/PULL_REQUEST_TEMPLATE.md`
   - `PULL_REQUEST_TEMPLATE.md` (저장소 루트)
   - `.github/PULL_REQUEST_TEMPLATE/*.md`

   없으면 사용자에게:

   > ".github/PULL_REQUEST_TEMPLATE.md 가 없습니다. 표준 양식으로 생성해 develop 에 추가할까요?"

   동의 시 `develop` 브랜치에 아래 내용으로 생성 → commit → push:

   ```markdown
   ## #️⃣ Task ID

   task-id / feature-id

   ## 📝작업 내용

   - 작업 내용

   ## PR 생성자 메모
   ```

   실행:

   ```bash
   git checkout develop
   mkdir -p .github
   # 위 템플릿을 .github/PULL_REQUEST_TEMPLATE.md 로 작성
   git add .github/PULL_REQUEST_TEMPLATE.md
   git commit -m "chore: add PR template"
   git push origin develop
   git checkout feature/<name>
   ```

   거부 시 템플릿 없이 진행 (gh 가 빈 본문으로 PR 생성).

8. **gh pr create — PR 생성**

   기존 PR 존재 여부 먼저 확인 (force-push 만으로 갱신되므로 새로 만들지 않음):

   ```bash
   gh pr view --head feature/<name> --json url --jq .url
   ```

   - 출력이 있으면 그 URL 을 사용자에게 안내하고 9번으로. **새 PR 만들지 않음.**
   - 출력이 없으면 (=PR 없음) 새로 생성:

     ```bash
     gh pr create \
       --base develop \
       --head feature/<name> \
       --title "<squashed commit subject>" \
       --body-file .github/PULL_REQUEST_TEMPLATE.md
     ```

     - PR 템플릿이 없는 경우(=7번에서 사용자가 거부) `--body-file` 인자를 빼고 `--body ""` 로 빈 본문 생성.
     - 다른 위치(예: `docs/PULL_REQUEST_TEMPLATE.md`) 에 템플릿이 있으면 해당 경로를 `--body-file` 로 지정.

   - 생성 성공 시 stdout 마지막 줄이 PR URL — 캡쳐해 9번에서 사용.
   - 실패 시 (권한/네트워크/branch protection 등) 메시지 그대로 사용자에게 전달하고 중단. 재시도 자동화 금지.

9. **사용자에게 보고**
   - 생성/재사용된 **PR URL** (클릭 가능 형태)
   - PR 신규 생성 / 기존 갱신 여부
   - squash 후 commit subject (= PR 제목)
   - rebase 결과: ahead `develop` 대비 commit 1개
   - push 결과
   - **다음 권장 행동**:
     1. PR 페이지에서 본문을 채우고 리뷰어를 지정하세요.
     2. 자동 리뷰가 필요하면 위 PR URL 을 인자로 **`/gf-pr-review <PR URL>`** 을 실행하세요 (gf-autopilot 의 PR 리뷰 스킬).
     3. PR 머지는 반드시 **`--no-ff` (merge commit 생성)** 방식으로 진행하세요:
        ```bash
        gh pr merge <PR#> --merge --delete-branch
        ```
        GitHub UI 에서는 "Create a merge commit" 옵션. **Squash and merge / Rebase and merge 사용 금지** — feature 브랜치가 이미 1 commit 으로 squash 되어 있어 다시 squash 할 필요가 없고, merge commit 으로 git-flow 의 분기점을 보존해야 합니다.

## Output

- **PR URL** (생성됐거나 기존에 있던 PR)
- 신규 생성 / 기존 갱신 여부
- squashed commit subject (PR 제목)
- 푸시된 ref (`feature/<name>`)
- PR 템플릿 신규 생성 여부

## Notes (구현 시 주의)

- **`gh` 인증 필수**: `gh auth status` 실패 시 즉시 중단. 스킬이 토큰 자동 발급 / `gh auth login` 자동 실행 금지. 사용자가 직접 인증한 뒤 다시 실행하도록 안내.
- **기존 PR 재사용**: 같은 head 브랜치로 이미 열린 PR 이 있으면 `gh pr view --head` 로 URL 만 찾아 안내. 새 PR 생성 시도 금지 (force-push 만으로 기존 PR 본체가 새 commit 으로 갱신됨).
- **충돌 자동 해결 금지**: rebase 중 충돌은 항상 사용자에게 위임. theirs/ours 자동 선택 금지.
- **force push 정책**: 항상 `--force-with-lease` 만 사용. 일반 `--force` 금지.
- **squash 메시지는 반드시 사용자 입력**: 자동 생성/합성 금지. commit 1개일 때만 기존 메시지 재사용.
- **PR 템플릿 생성은 동의 후**: develop 브랜치에 직접 commit 하는 일회성 셋업이므로 침묵하지 말 것.
- **branch protection 충돌**: 일부 저장소는 force push 자체를 금지. 거부되면 사용자에게 보고만 하고 중단.
- **PR 머지 정책 (`--no-ff`)**: 머지는 항상 merge commit 을 생성합니다 (`gh pr merge --merge` 또는 GitHub UI 의 "Create a merge commit"). Squash and merge / Rebase and merge 금지 — 본 스킬이 이미 push 전에 squash 했으므로 추가 squash 는 무의미하고, merge commit 이 없으면 git-flow 의 history 분기점이 사라집니다. 저장소의 GitHub 설정에서 "Allow merge commits" 만 활성화하고 나머지는 비활성화하는 것을 권장.
