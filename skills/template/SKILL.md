---
name: template
description: Reference template and authoring guide for gf-autopilot skills. NOT a working skill — do not invoke. Copy this directory to start a new skill. Open this file only when explicitly asked about "how to write a skill", "skill template", or "template".
version: 0.1.0
---

# template — gf-autopilot 스킬 작성 가이드

이 디렉터리는 **동작하는 스킬이 아닙니다**. 새 스킬을 만들 때 복사해서 시작하는 참고용 템플릿이자, gf-autopilot 플러그인 안에서 스킬을 작성하는 방법을 설명하는 튜토리얼입니다.

> This directory is **not a working skill**. It is a copy-to-start template and an authoring tutorial for skills inside the gf-autopilot plugin.

---

## 1. 스킬이란? (What is a skill?)

Claude Code 스킬은 `skills/<skill-name>/SKILL.md` 한 파일로 정의되는 모델 호출 가능 능력(model-invoked capability)입니다. 사용자가 직접 슬래시 커맨드처럼 호출할 수도 있고(user-invoked), Claude가 대화 맥락에 따라 자동으로 호출할 수도 있습니다(model-invoked). 차이는 frontmatter 한 줄로 결정됩니다.

---

## 2. Frontmatter 전체 스펙 (Full frontmatter spec)

스킬의 첫 블록은 YAML frontmatter여야 합니다. 지원되는 필드:

| 필드            | 필수 | 설명                                                                                       |
| --------------- | ---- | ------------------------------------------------------------------------------------------ |
| `name`          | 필수 | 스킬 식별자. 디렉터리 이름과 정확히 일치해야 합니다. (e.g. `feature-start`)                |
| `description`   | 필수 | Claude가 이 스킬을 언제 사용할지 판단하는 트리거 문장. **가장 중요한 필드.** 아래 §3 참고. |
| `version`       | 선택 | SemVer 권장 (`0.1.0`). 변경 이력 추적용.                                                   |
| `argument-hint` | 선택 | `<branch-name> [base]` 처럼 슬래시 커맨드 인자 힌트. `/help`에 표시됩니다.                 |
| `allowed-tools` | 선택 | 사전 허용 도구 목록. 권한 프롬프트를 줄입니다. 예: `[Bash, Read, Grep]`                    |
| `model`         | 선택 | 이 스킬 실행 시 사용할 모델 오버라이드. `haiku` / `sonnet` / `opus`.                       |
| `license`       | 선택 | 라이선스 표기/참조.                                                                        |

> 참고: `user-invocable` 같은 별도 필드는 존재하지 않습니다. 사용자가 슬래시 커맨드로 부르는 것과 Claude가 자동으로 호출하는 것 모두 같은 SKILL.md로 동작합니다. 호출 방식은 `description` 의 톤(트리거 문구 포함 여부)과 `argument-hint` 의 유무로 자연스럽게 갈립니다.

### Frontmatter 예시 — 모델 자동 호출용

```yaml
---
name: feature-start
description: Use when the user asks to "start a new feature", "create a feature branch", "begin work on feature/<x>", or any git-flow feature kickoff. Creates a feature/<name> branch from develop and pushes it.
version: 0.1.0
allowed-tools: [Bash, Read]
---
```

### Frontmatter 예시 — 사용자 슬래시 커맨드용

```yaml
---
name: feature-finish
description: Finish a git-flow feature branch — merge into develop, delete the branch, push.
argument-hint: <feature-name>
allowed-tools: [Bash, Read]
---
```

---

## 3. 자동 트리거링을 위한 description 작성법

`description` 필드는 Claude가 이 스킬을 자동으로 끌어쓸지 결정하는 거의 유일한 신호입니다. 다음 패턴을 따르세요.

**좋은 description 형식:**

```
Use when the user asks to "<phrase 1>", "<phrase 2>", mentions "<keyword>", or discusses <topic-area>. <One-sentence summary of what it does.>
```

**포함해야 할 것:**

- 사용자가 실제로 말할 법한 **정확한 트리거 문구** 2~5개 (한국어/영어 둘 다)
- 관련 키워드
- 스킬이 다루는 토픽 영역
- 한 줄 요약

**피해야 할 것:**

- 너무 일반적인 단어("git", "branch" 만 있으면 다른 스킬과 충돌)
- 트리거 의도가 없는 템플릿/유틸은 description에 "NOT a working skill", "reference only" 같은 안티-트리거 문구를 명시하세요. (이 `template` 이 그 예시입니다.)

---

## 4. SKILL.md 본문 표준 섹션 (Standard body outline)

frontmatter 다음 본문은 일반 마크다운입니다. 다음 5개 섹션을 권장합니다:

```markdown
# <Skill Name>

## Purpose

이 스킬이 해결하는 문제 한 문단. "왜" 존재하는지.

## When to use

- 트리거 조건 bullet
- 트리거하면 안 되는 경우 (anti-patterns)

## Inputs

- 인자 / 환경 / 사전조건 (예: 현재 브랜치가 develop이어야 함)

## Steps

1. 첫 번째 액션 (보통 상태 확인)
2. 두 번째 액션
3. ...
4. 검증
5. 보고

## Output

사용자에게 무엇을 돌려주는지 — 새 브랜치 이름, 머지된 커밋 SHA, 다음에 할 일 등.
```

---

## 5. 완성 예시 — `feature-start` 스킬

아래는 별도 파일이 아니라 **본 템플릿 안에 삽입된 예시**입니다. 실제로 만들 때는 이 내용을 `skills/feature-start/SKILL.md` 에 복사해 넣으세요.

```markdown
---
name: feature-start
description: Use when the user asks to "start a new feature", "create a feature branch", "feature 시작", "기능 브랜치 만들어", or any git-flow feature kickoff. Creates feature/<name> from develop, switches to it, and pushes upstream.
version: 0.1.0
argument-hint: <feature-name>
allowed-tools: [Bash, Read]
---

# feature-start

## Purpose

git-flow 컨벤션에 따라 `develop` 브랜치에서 `feature/<name>` 브랜치를 일관되게 시작합니다. 브랜치 이름 규칙, 베이스 브랜치 확인, 원격 푸시까지 한 번에 처리해 휴먼 에러를 줄입니다.

## When to use

- 사용자가 "새 기능 시작", "feature/foo 만들어", "start a feature for X" 라고 말할 때
- 현재 작업 트리가 깨끗하고(`git status` clean), `develop` 브랜치가 존재할 때

**Anti-pattern**: 릴리스/핫픽스 브랜치 생성 요청에는 트리거하지 마세요 (별도 스킬).

## Inputs

- `$ARGUMENTS` — 새 기능 이름 (kebab-case 권장, e.g. `user-login`)
- 사전조건: `git status` 가 clean, `develop` 브랜치 존재, `origin` 리모트 존재

## Steps

1. `git status --porcelain` 실행 — 비어있지 않으면 사용자에게 stash/commit 안내 후 중단.
2. `git fetch origin` 으로 최신화.
3. `git checkout develop && git pull --ff-only origin develop`.
4. `git checkout -b feature/$ARGUMENTS`.
5. `git push -u origin feature/$ARGUMENTS`.
6. 새 브랜치 이름과 다음 단계 안내를 사용자에게 보고.

## Output

- 생성된 브랜치 이름 (`feature/user-login`)
- 추적 원격 (`origin/feature/user-login`)
- 다음 액션 제안: "이제 코드를 작성하고 끝나면 /feature-finish user-login 을 실행하세요."
```

---

## 6. 새 스킬을 시작하는 방법 (Copy this template)

PowerShell:

```powershell
Copy-Item -Recurse skills\template skills\<new-skill-name>
# 그 다음 skills\<new-skill-name>\SKILL.md를 열어
# frontmatter의 name, description, version, body를 교체하세요.
```

bash:

```bash
cp -r skills/template skills/<new-skill-name>
# edit skills/<new-skill-name>/SKILL.md
```

체크리스트:

- [ ] `name` 이 디렉터리 이름과 일치
- [ ] `description` 에 트리거 문구 2개 이상 포함 (한/영)
- [ ] 본문에 Purpose / When to use / Inputs / Steps / Output 다섯 섹션
- [ ] 안티-트리거 사례 (이 스킬을 호출하면 안 되는 경우) 명시
- [ ] `allowed-tools` 로 필요한 도구만 허용
- [ ] 로컬에서 Claude Code 재로드 후 트리거 문구로 자동 호출되는지 확인

---

## 7. 참고 (Reference)

- Anthropic Claude Code 공식 플러그인 가이드: https://code.claude.com/docs/en/plugins
- 본 프로젝트 저장소: https://github.com/paraang/gf-autopilot
