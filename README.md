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

업데이트:

```bash
/plugin marketplace update gf-autopilot
/plugin update gf-autopilot@gf-autopilot
```

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
