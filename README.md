# siolab-claude

Claude / Claude Code 스터디 기록.

[siolab](https://github.com/ShinsRo/siotlab)의 `ai/claude` 에 서브모듈로 마운트되어 있다. 독립 repo로 둔 이유는 병렬 에이전트를 험하게 돌려보는 과정에서 siolab 본체와 설정·권한·worktree가 섞이지 않게 하기 위함이다.

## 주제

1. [병렬 작업](parallel/README.md) — 서브에이전트, 에이전트 뷰, 에이전트 팀, 동적 워크플로우

## 작업 규칙

이 repo는 서브모듈이므로 몇 가지 규율이 필요하다.

**cwd는 항상 이 repo 안에서.** `claude`를 siolab 루트에서 띄우면 저장소 루트가 siolab이 되어 이 repo의 `.claude/` 설정이 로드되지 않는다. 또한 siolab 루트에서 `claude --worktree`나 `/batch`를 쓰면 그 worktree에는 서브모듈이 체크아웃되지 않아(`git worktree add`는 서브모듈을 채우지 않음) 이 디렉터리가 빈 상태로 보인다. 복구는 해당 worktree에서 `git submodule update --init`.

**커밋은 2단계.** 이 repo에서 커밋·푸시한 뒤, siolab 쪽에서 포인터 커밋을 따로 해야 한다.

```bash
# 1) 여기서
git add -A && git commit -m "..." && git push

# 2) siolab 루트에서
git add ai/claude && git commit -m "chore: siolab-claude 포인터 갱신"
```

**detached HEAD 주의.** `git submodule update` 이후 이 repo는 detached 상태가 된다. 작업 전에 `git switch main`. 잊고 커밋하면 브랜치에 붙지 않은 커밋이 된다.

**worktree 기준 브랜치.** `worktree.baseRef` 기본값은 `"fresh"`(origin의 기본 브랜치)이므로, 푸시하지 않은 커밋은 새 worktree에 들어오지 않는다. 진행 중인 작업 위에서 worktree를 만들려면 `.claude/settings.json`에 `worktree.baseRef: "head"`.

## 클론

```bash
git clone --recurse-submodules git@github.com:ShinsRo/siotlab.git
# 이미 클론했다면
git submodule update --init ai/claude
```
