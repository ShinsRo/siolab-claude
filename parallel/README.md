# Claude Code 병렬 작업

Claude Code로 작업을 병렬화하는 방법들을 조사하고 익히기 위한 프로젝트.

- 기준 문서: [에이전트를 병렬로 실행하기](https://code.claude.com/docs/ko/agents)
- 정리 방식: 접근 방식 하나당 폴더 하나. 개념 정리 + 실제 실행 후 관찰 기록.
- 문서 본문은 Learning 모드로 직접 작성한다. (스켈레톤을 미리 깔아두지 않음)

## 접근 방식 지도

기준 문서가 갈라놓은 축을 그대로 따른다. "누가 조율하는가 / 작업자끼리 대화가 필요한가 / 같은 파일을 건드리는가" 세 질문으로 갈린다.

| # | 접근 방식 | 조율 주체 | 결과 보고 대상 | 상태 |
|---|---|---|---|---|
| 01 | [서브에이전트](https://code.claude.com/docs/ko/sub-agents) | Claude (한 세션 안에서 위임) | 생성한 대화 | 안정 |
| 02 | [에이전트 뷰](https://code.claude.com/docs/ko/agent-view) | 사람 (백그라운드 세션 디스패치) | 사용자 | 연구 프리뷰 |
| 03 | [에이전트 팀](https://code.claude.com/docs/ko/agent-teams) | Claude 리더 (팀원 할당·감독) | 팀원 간 직접 메시징 | 실험적, 기본 비활성화 |
| 04 | [동적 워크플로우](https://code.claude.com/docs/ko/workflows) | 스크립트 (다수 서브에이전트 + 교차 검증) | 워크플로우 실행 | 안정 |

보조 도구 — 병렬 실행 방식 자체는 아니지만 위 네 가지를 받쳐주는 것들:

- [Worktrees](https://code.claude.com/docs/ko/worktrees) — 세션마다 별도 git 체크아웃. 같은 파일 충돌 방지.
- [크로스 세션 메시징](https://code.claude.com/docs/ko/cross-session-messaging) — 세션 간 상태·결과 전달.
- [`/batch`](https://code.claude.com/docs/ko/commands) — 큰 변경을 worktree 격리 서브에이전트 5~30개로 쪼개 각각 PR. 서브에이전트 + worktree의 패키지된 사용.

범위에서 제외(다른 문제를 푸는 기능이므로 혼동만 정리하고 넘어감): 백그라운드 bash(에이전트 생성 안 함), [루틴](https://code.claude.com/docs/ko/routines)(클라우드 스케줄 실행).

## 폴더 구조

```
parallel/
├── README.md
├── TODO.md
├── 01-sub-agents/          # 서브에이전트, 포크(/subtask)
│   ├── README.md
│   └── .claude/agents/     # 이 주제에서 쓰는 서브에이전트 정의 실물
├── 02-agent-view/          # claude agents, 백그라운드 세션
├── 03-agent-teams/         # 실험적
├── 04-workflows/           # 동적 워크플로우
└── 90-tooling/             # worktrees, 크로스 세션 메시징, /batch
```

각 폴더는 `README.md`(개념·설정·한계)를 기본으로 하고, 필요에 따라 다음을 둔다.

- 설정 파일 실물: `.claude/agents/*.md`, `settings.json` 조각 등
- 실행·관찰 기록: `observations.md` (동작, 토큰 비용 체감, 실패 케이스)
- 스크립트나 노트북이 필요하면 같은 폴더에

### 설정 파일을 주제 폴더 안에 둘 수 있는 이유

프로젝트 서브에이전트는 **cwd에서 저장소 루트까지 거슬러 올라가며 만나는 모든 `.claude/agents/`** 를 스캔한다. 따라서 `01-sub-agents/`에서 `claude`를 띄우면 `01-sub-agents/.claude/agents/`의 정의가 그대로 로드된다. 저장소 루트에 모아둘 필요가 없다.

- 같은 `name`이 여러 층에 있으면 cwd에 가까운 정의가 이긴다 (v2.1.178+)
- 반대로 저장소 루트를 넘어가지는 않는다 → siolab 본체(`../../`)의 설정은 보이지 않는다. [독립 repo로 분리한 이유](../README.md)와 맞물린다.
- `.claude/agents/`는 재귀 스캔되지만 식별자는 `name` frontmatter뿐이므로, 트리 전체에서 `name`을 겹치지 않게 유지한다

## 실행 상태 확인 치트시트

| 대상 | 명령 |
|---|---|
| 백그라운드 세션 전체 | `claude agents` |
| 현재 세션의 백그라운드 작업(완료된 서브에이전트 포함) | `/tasks` |
| 동적 워크플로우 실행 | `/workflows` |
| 서브에이전트 정의 위치 | `.claude/agents/*.md` (v2.1.198부터 `/agents`는 패널을 열지 않고 경로만 안내) |

## 주의

세션·서브에이전트를 동시에 여럿 돌리면 토큰 사용량이 곱으로 늘어난다. 실습 시 [비용](https://code.claude.com/docs/ko/costs) 기준을 함께 기록한다.
