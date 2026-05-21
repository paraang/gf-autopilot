---
name: gf-pr
description: Use when the user asks to "create a PR", "open a pull request", "PR 만들어줘", "feature finish 전에 PR 띄워줘", "rebase and squash 후 PR", "send the feature for review", or wants to ship a feature branch via pull request before running git flow feature finish. Updates develop, rebases the feature branch on top of it, squashes commits into one, force-with-lease pushes, ensures `.github/PULL_REQUEST_TEMPLATE.md` exists (creating it from the project's standard template if missing), and prints a GitHub compare URL that opens the PR creation page with the template prefilled. If a rebase conflict appears, the skill stops and waits for the user to resolve it. Does NOT use `gh` CLI.
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
- GitHub **compare URL** 을 만들어 사용자에게 안내 → 사용자가 브라우저로 PR 생성
- `gh` CLI 미사용

## When to use

- "PR 만들어줘", "PR 띄우기", "리뷰 보낼게", "rebase and squash 후 PR"
- feature 작업을 마쳤지만 곧바로 develop에 머지하지 않고 리뷰 절차를 거치고 싶을 때

**Anti-pattern**

- 현재 브랜치가 `feature/*` 가 아닐 때는 사용하지 말 것. release/hotfix 브랜치는 PR 흐름이 다름.
- 작업 트리에 unstage/uncommit 변경이 있을 때는 그것부터 commit/stash 후 사용.

## Inputs

- 사전조건
  - `git flow` AVH edition (gf-init 사전 권장)
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
   git status --porcelain        # 비어있어야 함
   git branch --show-current     # feature/* 여야 함
   git remote get-url origin     # GitHub URL
   ```

   조건 미충족 시 중단하고 사유 안내.

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

   거부 시 템플릿 없이 진행 (compare URL 만 출력).

8. **compare URL 생성 + 안내**

   - `git remote get-url origin` 으로 owner/repo 추출. HTTPS 와 SSH URL 양쪽 처리:
     - `https://github.com/<owner>/<repo>.git`
     - `git@github.com:<owner>/<repo>.git`
   - URL 구성:

     ```text
     https://github.com/<owner>/<repo>/compare/develop...<feature-branch>?expand=1
     ```

     `expand=1` 이 PR 생성 페이지를 바로 띄우고, PR 템플릿 파일이 있으면 본문에 자동 prefill 됩니다.

   - 옵션: squashed commit 메시지를 PR 제목으로 prefill 하려면 `&title=<URL-encoded>` 추가. 자동 추가는 하지 말고 commit subject 만 그대로 안내 텍스트에 노출.

9. **사용자에게 보고**

   - 생성된 compare URL (클릭 가능 형태)
   - squash 후 commit subject (PR 제목 후보)
   - rebase 결과: ahead `develop` 대비 commit 1개
   - push 결과
   - 다음 액션: "브라우저에서 위 URL 을 열어 PR 을 생성하세요. 본문은 `.github/PULL_REQUEST_TEMPLATE.md` 양식에 따라 자동 채워집니다."

## Output

- GitHub compare URL (PR 생성 페이지)
- squashed commit subject (PR 제목 후보)
- 푸시된 ref (`feature/<name>`)
- PR 템플릿 신규 생성 여부

## Notes (구현 시 주의)

- **`gh` CLI 미사용**: 모든 단계는 plain git + URL 생성으로 완료. PR 자체의 최종 생성(=Submit)은 사용자가 브라우저에서 수행.
- **충돌 자동 해결 금지**: rebase 중 충돌은 항상 사용자에게 위임. theirs/ours 자동 선택 금지.
- **force push 정책**: 항상 `--force-with-lease` 만 사용. 일반 `--force` 금지.
- **squash 메시지는 반드시 사용자 입력**: 자동 생성/합성 금지. commit 1개일 때만 기존 메시지 재사용.
- **PR 템플릿 생성은 동의 후**: develop 브랜치에 직접 commit 하는 일회성 셋업이므로 침묵하지 말 것.
- **이미 PR이 열려 있을 수 있음**: compare URL 은 이미 열린 PR이 있어도 동일 페이지를 보여줍니다. 사용자가 기존 PR 갱신을 의도한 거라면 force-with-lease push 만으로 충분 — 별도 PR 재생성 불필요.
- **branch protection 충돌**: 일부 저장소는 force push 자체를 금지. 거부되면 사용자에게 보고만 하고 중단.
