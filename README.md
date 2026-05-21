# gf-autopilot

Claude Code 플러그인 — **git-flow 워크플로우 자동화**.

> A Claude Code plugin that automates git-flow workflows: feature, release, and hotfix branches with consistent naming and merge orchestration.

현재 버전: `0.1.0` — 스캐폴드 단계. 실제 스킬은 `_template`을 복사해서 점진적으로 추가합니다.

---

## 무엇을 하나요? (What it does)

gf-autopilot은 git-flow 브랜칭 모델을 Claude Code 안에서 일관되게 사용하도록 돕는 스킬 모음입니다. 계획된 기능:

- `feature-start <name>` — `develop` 에서 `feature/<name>` 브랜치 생성·푸시
- `feature-finish <name>` — `feature/<name>` 를 `develop` 에 머지·삭제
- `release-start <version>` — `release/<version>` 브랜치 생성
- `release-finish <version>` — `main` 과 `develop` 에 머지, 태그 생성
- `hotfix-start <name>` — `main` 에서 `hotfix/<name>` 분기
- `hotfix-finish <name>` — `main` 과 `develop` 에 머지, 태그 생성

> 위 스킬들은 아직 구현되지 않았습니다. `skills/_template/SKILL.md` 를 참고해 하나씩 추가하세요.

---

## 설치 (Installation)

이 저장소가 GitHub에 호스팅되어 있다고 가정합니다 (`https://github.com/paraang/gf-autopilot`).

```bash
# 1) 마켓플레이스 등록 (한 번만)
/plugin marketplace add paraang/gf-autopilot

# 2) 플러그인 설치
/plugin install gf-autopilot@gf-autopilot
```

마켓플레이스 이름과 플러그인 이름이 모두 `gf-autopilot` 인 것은 의도된 동작입니다 — 이 저장소는 **단일 플러그인을 셀프 호스팅하는 마켓플레이스**이기 때문입니다.

---

## 플러그인 업데이트 (Plugin update)

대상에 따라 두 흐름이 있습니다. 본인이 어떤 역할인지에 맞춰 따라가세요.

### 1) 설치자 — 이미 설치한 플러그인을 최신 릴리스로 갱신

```bash
/plugin marketplace update gf-autopilot
/plugin update gf-autopilot@gf-autopilot
```

위 두 명령이 보통 충분합니다. 마켓플레이스 메타데이터를 새로 가져오고(`marketplace.json` 의 `source.ref` 가 새 태그로 바뀐 것을 인지), 그 태그 시점의 플러그인 본체를 로컬에 받아옵니다.

옛 캐시 잔재로 새 버전이 인식되지 않으면 **마켓플레이스 재등록**:

```bash
/plugin marketplace remove gf-autopilot
/plugin marketplace add paraang/gf-autopilot
/plugin install gf-autopilot@gf-autopilot
```

그래도 옛 버전이 잡히면 **캐시 디렉터리 수동 삭제** (Windows 예시):

```powershell
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\plugins\marketplaces\gf-autopilot"
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\plugins\cache\gf-autopilot" -ErrorAction SilentlyContinue
```

macOS / Linux 는 `~/.claude/plugins/marketplaces/gf-autopilot` 와 `~/.claude/plugins/cache/gf-autopilot` 를 `rm -rf` 합니다.

### 2) 메인테이너 — 새 버전을 릴리스해서 배포

권장: `/gf-release` 스킬 사용. 스킬이 아래를 자동 수행합니다.

1. 사용자에게 새 버전 번호 입력 받음 (필수, 자동 추론 없음)
2. 기존 태그 컨벤션과 비교해 일관성 검증
3. 프로젝트 타입별 버전 파일 동기화 (`plugin.json` 의 `version`, `marketplace.json` 의 plugin entry `version` **및 `source.ref`**)
4. `CHANGELOG.md` 갱신 — [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/) 형식, 배포자 메모 포함
5. `git flow release start` → 위 변경 commit → `git flow release finish` (main 머지 + annotated 태그 + develop 백머지 + release 브랜치 삭제)
6. 사용자 확인 후 `git push origin main develop <version>`

```text
/gf-release
```

스킬을 쓰지 않는 경우 수동 절차:

```bash
# 1. develop 위에서 release 시작
git checkout develop
git flow release start <version>

# 2. 버전 동기화
#    .claude-plugin/plugin.json       — "version" 을 <version> 으로
#    .claude-plugin/marketplace.json  — plugins[0].version 과 plugins[0].source.ref 모두 <version> 으로
git commit -am "chore(release): bump version to <version>"

# 3. CHANGELOG.md 갱신 (Keep a Changelog 1.1.0)
git commit -am "docs(changelog): add <version> entry"

# 4. release finish — main 머지 + 태그 + develop 백머지 + release 브랜치 삭제
GIT_MERGE_AUTOEDIT=no git flow release finish -m "Release <version>" <version>

# 5. push
git push origin main develop <version>
```

`marketplace.json` 의 `source.ref` 가 새 태그로 갱신되어 default branch(`develop`) 에 반영되면, 설치자들이 `/plugin marketplace update` 로 새 버전을 받게 됩니다.

> 핫픽스(이미 릴리스된 버전의 긴급 결함 수정)는 `git flow hotfix start/finish` 흐름을 사용하세요. 향후 `/gf-hotfix` 스킬로 자동화될 예정입니다.

---

## 새 스킬 추가하기 (Adding a new skill)

모든 스킬은 `skills/<skill-name>/SKILL.md` 한 파일로 정의됩니다. 시작점은 `_template` 입니다:

```bash
cp -r skills/_template skills/feature-start
# 또는 Windows PowerShell:
Copy-Item -Recurse skills\_template skills\feature-start
```

그런 다음 `skills/feature-start/SKILL.md` 를 열어:

1. frontmatter의 `name` 을 디렉터리 이름과 일치시키기
2. `description` 에 자동 트리거를 위한 정확한 사용자 문구 2~5개 넣기
3. 본문에 **Purpose / When to use / Inputs / Steps / Output** 다섯 섹션 작성

자세한 작성 가이드는 `skills/_template/SKILL.md` 에 모두 들어있습니다.

---

## 디렉터리 구조 (Repository layout)

```
gf-autopilot/
├── .claude-plugin/
│   ├── plugin.json          # 플러그인 메타데이터
│   └── marketplace.json     # 마켓플레이스 정의(셀프 호스팅)
├── skills/
│   └── _template/           # 새 스킬의 출발점 (동작하지 않음)
│       └── SKILL.md
├── README.md
├── LICENSE
└── .gitignore
```

---

## 기여 (Contributing)

1. 새 스킬은 `skills/_template` 을 복사해서 시작합니다.
2. PR 전에 로컬에서 Claude Code를 재로드하고 트리거 문구로 자동 호출되는지 확인합니다.
3. 버전은 SemVer를 따릅니다. 스킬 추가 = minor bump, 버그 수정 = patch bump.

---

## 라이선스 (License)

MIT — `LICENSE` 파일 참고.
