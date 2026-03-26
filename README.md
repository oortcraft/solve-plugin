# solve

**MD-centric problem-solving agent framework for Claude Code**

AI로 실행하는 건 쉽다. 하지만 **뭘 실행할지 모르고 시작하면 방향을 잃는다.**

solve는 문제 분석부터 실행까지의 전 과정을 단계별 전담 에이전트가 가이드하는 Claude Code 플러그인입니다. 모든 상태는 사람이 읽을 수 있는 마크다운 파일에 저장되어, 세션이 끊겨도 자연스럽게 이어갈 수 있습니다.

---

## Install

`~/.claude/settings.json`에 추가:

```json
{
  "extraKnownMarketplaces": {
    "solve": {
      "source": {
        "source": "git",
        "url": "https://github.com/oortcraft/solve-plugin.git"
      }
    }
  },
  "enabledPlugins": {
    "solve@solve": true
  }
}
```

Claude Code를 재시작하면 `/solve` 커맨드가 활성화됩니다.

---

## Quick Start

```bash
# 1. 새 프로젝트 시작
/solve init my-project

# 2. 문제를 설명하면 Problem Analyst가 정리해줌
"유튜브 영상 제작을 자동화하고 싶은데, 어디서부터 시작해야 할지 모르겠어"

# 3. 세션이 끊겨도 다시 이어가기
/solve

# 4. 현재 상태 확인
/solve status
```

---

## How It Works

### 4-Phase Workflow

```
Phase 1: PROBLEM ──── "뭐가 문제야?"
    │   Agent: Problem Analyst (sonnet)
    │   Files: WHY.md + THINK.md
    │
    ▼
Phase 2: DIRECTION ── "어떤 방향으로?"
    │   Agent: Direction Setter (sonnet)
    │   Files: DIRECTION.md
    │
    ▼
Phase 3: PLANNING ─── "구체적으로 뭘 할 거야?"
    │   Agent: Solve Planner (opus)
    │   Files: PLAN.md
    │
    ▼
Phase 4: EXECUTION ── "만들자!"
        Agent: Solve Executor (opus)
        Files: code + PROGRESS.md
```

각 단계는 조건이 충족되어야 다음으로 넘어갑니다. 시스템이 사용자를 적절한 절차로 가이드하며, 헤매면 이전 단계로 돌아가도록 안내합니다.

### .solve/ Directory

프로젝트 폴더 안에 `.solve/` 디렉토리가 생성됩니다:

```
my-project/
├── .solve/
│   ├── state.json      # 현재 단계 (기계용)
│   ├── WHY.md          # 문제 배경, 동기
│   ├── THINK.md        # 브레인스토밍, 아이디어
│   ├── DIRECTION.md    # 비전, 원칙, 범위
│   ├── PLAN.md         # 실행 계획, 태스크
│   ├── PROGRESS.md     # 진행 상황, 세션 로그
│   └── REFERENCE.md    # 참고 자료
└── src/                # 실제 코드
```

모든 MD 파일은 사람이 직접 열어서 읽고 편집할 수 있습니다.

---

## Commands

| Command | Description |
|---------|-------------|
| `/solve init <name>` | 새 프로젝트 초기화 (.solve/ 생성) |
| `/solve` | 기존 프로젝트 이어가기 (자동으로 현재 단계 에이전트 실행) |
| `/solve status` | 현재 단계, 진행 상황 요약 |
| `/solve next` | 다음 단계로 이동 (조건 미충족 시 경고 후 강제 가능) |
| `/solve back` | 이전 단계로 돌아가기 (이유 기록) |

---

## Session Persistence

solve의 핵심 가치는 **세션 간 연속성**입니다.

```
세션 1: /solve init → WHY.md 작성 중... → 세션 종료

세션 2: /solve → state.json 읽음 → "지난번에 문제 분석까지 했습니다. 이어서 할까요?"

세션 3: /solve → "방향이 설정됐습니다. 계획을 세울까요?"
```

state.json이 손상되면 PROGRESS.md에서 복구를 시도하고, 그것도 안 되면 사용자에게 직접 물어봅니다.

---

## Agents

| Agent | Model | Phase | Role |
|-------|-------|-------|------|
| **Problem Analyst** | sonnet | 1. PROBLEM | 막연한 생각을 구조화된 문제 정의로 변환. 한 번에 하나의 질문만. |
| **Direction Setter** | sonnet | 2. DIRECTION | 비전, 원칙, 핵심 결정, 범위를 확립. |
| **Solve Planner** | opus | 3. PLANNING | 방향을 구체적 태스크로 변환. 모든 태스크에 수락 기준 필수. |
| **Solve Executor** | opus | 4. EXECUTION | PLAN.md 순서대로 구현. 완료 시 체크오프. |

각 에이전트는 자기 소유 파일만 수정합니다. PROGRESS.md와 REFERENCE.md는 모든 에이전트가 추가 가능합니다.

---

## Phase Transitions

단계 전환은 **조건 기반**입니다. 에이전트가 전환을 제안하고 사용자가 확인합니다.

| Transition | Conditions |
|------------|------------|
| Problem → Direction | WHY.md에 Problem Statement + Background, THINK.md에 2+ 구조화된 생각 |
| Direction → Planning | Vision, 2+ 원칙, 1+ 결정, In/Out Scope 정의 |
| Planning → Execution | 2+ 태스크 각각 Acceptance 기준 있음, 사용자 리뷰 완료 |
| Execution → Complete | 모든 태스크 체크 완료 |

조건이 안 됐는데 넘어가고 싶으면 `/solve next`로 강제 이동 가능 (로그에 기록됨).

---

## Coaching & Regression

사용자가 헤매면 시스템이 이전 단계로 안내합니다:

- Phase 2에서 **"사실 문제가 다른 것 같아"** → Phase 1로 돌아가기 제안
- Phase 3에서 **태스크가 원칙과 모순** → Phase 2로 돌아가기 제안
- Phase 4에서 **계획에 없는 큰 작업 발견** → Phase 3으로 돌아가기 제안

---

## Why Not Just Use OMC?

| | OMC | solve |
|---|---|---|
| 상태 파일 | 기계 중심 (`.omc/state/`) | 사람 중심 (`.solve/*.md`) |
| 시작점 | 명확한 요구사항 | 막연한 생각 |
| 핵심 | 실행 자동화 | 사고 과정 가이드 |
| 관계 | - | 보완적 (solve → OMC 연계 가능) |

---

## License

MIT
