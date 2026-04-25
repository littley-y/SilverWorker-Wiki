# Post-Setup Review — Graphify + GitHub Pages 연동 (Claude)

> **검토 대상**: `General/history/2026-04-25-graphify-setup.md` + 실제 파일 시스템 상태
> **검토 범위**: graphify-out/, .graphifyignore, .gitignore, pages.yml, post-commit hook, AGENTS.md
> **Reviewer**: Claude
> **Date**: 2026-04-25
> **Verdict**: 🟡 **Conditional OK — spec_02 진입 전 4건 정리 권장**

---

## 0. 종합

graphify 빌드 자체는 정상 동작 (247 nodes / 242 edges / 27 communities / 100% EXTRACTED / $0). AGENTS.md의 master orchestration 내용도 보존된 채 graphify 섹션만 append됨 — overwrite 사고는 없었음.

다만 `.gitignore` 정책 변경 1건이 silently 발생했고, GitHub Pages 자동 배포 path 매핑에 결함이 있어 **현재 상태로는 graphify 갱신이 사이트에 반영되지 않습니다**. multi-worktree 환경에서 중복 노드 추출 가능성도 검증 필요.

---

## 1. ✅ 정상 처리

| 항목 | 상태 |
|---|---|
| `graphify-out/graph.json` (209 KB) | ✅ 존재 |
| `graphify-out/graph.html` (194 KB) | ✅ 존재 |
| `graphify-out/GRAPH_REPORT.md` | ✅ 존재, 가독성 OK |
| Claude PreToolUse hook | ✅ `.claude/settings.json` |
| OpenCode plugin | ✅ `.opencode/plugins/graphify.js` |
| Gemini BeforeTool hook | ✅ `.gemini/settings.json` |
| Git post-commit hook | ✅ `SilverWorkerNow/.git/hooks/post-commit` |
| AGENTS.md 무결성 | ✅ master orchestration + graphify 섹션 append (73줄) |
| `.graphifyignore` | ✅ build/dart_tool/env/secrets 제외 적절 |

추출 결과 god nodes 1·7위가 `flutter/material.dart`, `firebase_core`로 외부 패키지인 점은 코드가 spec_01 skeleton 단계라 정상. spec_02~10 진행하며 자체 추상화(`UserModel`, `JobModel` 등)로 이동하는지 모니터링 가치 있음.

---

## 2. ⚠️ Major

### M-1. `.gitignore`의 `AGENTS.md` 항목 — **위험한 잘못된 정책**

```
# Graphify self-reference (don't extract own instructions)
AGENTS.md
CLAUDE.md
GEMINI.md
.graphifyignore
```

**문제**:
- 주석의 의도(graphify가 자기 instruction을 추출하지 않게)는 **`.graphifyignore`의 책임**이지 `.gitignore`의 책임이 아님. 두 파일의 역할이 혼동됨.
- AGENTS.md는 본 프로젝트의 **Master Orchestration Document** ("Common Constitution"). git에서 제외되면 안 됨.
- 현재는 AGENTS.md가 SilverWorker 루트(non-git)에만 있어 즉시 사고는 아님. 그러나 **다음 `graphify * install` 실행 시 git working tree(SilverWorkerNow_dev 등) 안에 AGENTS.md가 생성되면 silently lost**. multi-agent 워크플로우 재현성 파괴.

**수정**:
```
# .gitignore 에서 다음 라인 삭제
AGENTS.md
CLAUDE.md
GEMINI.md

# .graphifyignore 에 추가 (graphify 자기 추출 방지)
AGENTS.md
CLAUDE.md
GEMINI.md
```

### M-2. `android/app/google-services.json`이 `.gitignore`에 추가됨 — 무선언 정책 변경

```
# Environment & Secrets
.env
android/app/google-services.json
```

**문제**:
- 이전 v2 리뷰 사이클에서 "commit 유지 + GCP API key restriction(M-3) 적용"으로 결정한 항목이, graphify 작업 도중 **명시적 의사결정 기록 없이** ignore로 변경됨
- 이미 commit된 상태일 경우 `git rm --cached android/app/google-services.json` 필요 (안 하면 ignore 무효)
- M-3(API key application restriction)이 여전히 미적용 상태 → git에서도 빠지면 외부 협업자가 빌드 불가, CI 빌드 환경에서 secrets-as-env 패턴 도입 필요
- v2 리뷰의 M-4 정책이 **방향만 바뀐 채 결정 근거 미기록**

**수정 옵션 (사용자 결정 필요)**:
- **(A) commit 유지로 복귀** + M-3 GCP application restriction 즉시 적용
- **(B) ignore 유지** + (1) `git rm --cached`로 git에서 제거 (2) CI에 secret 주입 메커니즘 도입 (3) `history/`에 정책 변경 결정 기록

### M-3. `pages.yml`의 path 매핑 결함 — **자동 배포 동작 안 함**

`SilverWorkerNow/.github/workflows/pages.yml`:
```yaml
on:
  push:
    branches: [ master ]
    paths:
      - 'docs/**'
      - 'graphify-out/**'
```

**문제**:
- workflow는 `SilverWorkerNow/`(git working tree) 기준으로 path 평가
- 하지만 실제 `graphify-out/`은 **`/home/dudxo13/Projects/SilverWorker/graphify-out/`** (parent 디렉토리, git tree 밖)에 위치 → trigger 발화 안 됨
- `docs/index.html`은 1회 `cp`된 정적 스냅샷. graphify update가 갱신해도 docs/는 자동 동기화되지 않음 → **deploy되어도 옛 graph**

**수정 옵션**:
- **(A)** `graphify-out/`을 `SilverWorkerNow/graphify-out/`으로 이동 + `graphify update SilverWorkerNow` 재빌드
- **(B)** workflow에 빌드 단계 추가:
  ```yaml
  - name: Install graphify
    run: pip install --user graphifyy
  - name: Rebuild graph
    run: graphify update . --no-llm  # AST-only
  - name: Copy to docs
    run: cp graphify-out/graph.html docs/index.html
  ```
- **(C)** GitHub Pages 자동 배포 포기, 수동 `cp + commit` 워크플로우 유지

### M-4. `graphify`가 multi-worktree 전체를 corpus에 흡수 가능성 — 중복 노드

`graphify update .`을 SilverWorker(parent, non-git) 루트에서 실행 → `SilverWorkerNow/`, `SilverWorkerNow_dev/`, `_design/`, `_devops/`, `_qa/` 5개 워크트리의 동일 코드(`main.dart`, `lib/models/*.dart`)가 **5중 추출됐을 가능성**.

`.graphifyignore`에 `SilverWorkerNow_*/` 또는 `SilverWorkerNow_{dev,design,devops,qa}/` 패턴이 없음.

**검증 명령**:
```bash
cd /home/dudxo13/Projects/SilverWorker
grep -oE '"file_path": "[^"]*main\.dart"' graphify-out/graph.json | sort -u
# → 1줄이면 OK, 5줄이면 중복
```

**수정**: `.graphifyignore`에 워크트리 제외 추가 후 재빌드:
```
SilverWorkerNow_dev/
SilverWorkerNow_design/
SilverWorkerNow_devops/
SilverWorkerNow_qa/
```

### M-5. post-commit hook이 master worktree(`SilverWorkerNow`)에만 설치됨

`graphify hook install`이 한 번만 실행되어 `SilverWorkerNow/.git/hooks/post-commit`에만 hook 존재. **실제 작업이 일어나는 `SilverWorkerNow_dev/`** (implementer 워크트리)에서 commit 시 그래프 자동 재빌드 미동작.

worktree는 `.git`이 별도 파일/디렉토리이므로 hook도 worktree별로 설치해야 함.

**수정 옵션**:
- **(A)** 각 워크트리에서 `graphify hook install` 실행 (4회)
- **(B)** master에서만 갱신하는 정책 명시 (PR merge 후 `sync_worktrees.sh` 또는 master에서 수동 `graphify update`)

---

## 3. 🟡 Minor / Nit

- **N-1**: `docs/index.html`이 인터랙티브 graph.html만 노출. `GRAPH_REPORT.md`(god nodes/communities 정리본)와 진입 navigation은 없음. 사람용 wiki로 사용하려면 landing 페이지 추가 권장.
- **N-2**: GitHub Pages **Repository Settings → Pages → Source: GitHub Actions** 활성화가 사용자 수동 작업으로 deferred (history §9.1). 미수행 시 workflow success해도 publish 안 됨. `PROGRESS.md` 미완 작업 절에 명시 권장.
- **N-3**: GitHub Pages는 **public**. graph에 Firebase project ID(`silverworker-d8d64`), firestore.rules, spec/PR 리뷰 내용이 모두 색인되어 노출됨. 본 프로젝트가 OSS 의도라면 OK이지만 의식적 결정 기록은 history에 없음. 1줄 코멘트로 공개 의도 확인 권장.
- **N-4**: history.md §6 "Created: `.github/workflows/pages.yml`"에 정확한 경로(`SilverWorkerNow/.github/workflows/`) 미기재. 후속 작업자가 SilverWorker 루트에서 찾을 수 있음. history 문서 정정 권장.
- **N-5**: GRAPH_REPORT의 god node 1·7위가 외부 패키지(`flutter/material.dart`, `firebase_core`). spec_01 skeleton 단계라 정상이나, spec_05 이후 자체 추상화가 god node로 떠오르지 않으면 의존성 디자인 재검토 신호.

---

## 4. 사용자 결정 필요 사항

1. **M-1 (`.gitignore`의 AGENTS.md)** — 즉시 수정 권장 (위험)
2. **M-2 (google-services.json 정책)** — 옵션 (A)/(B) 중 선택
3. **M-3 (pages.yml 자동 배포)** — 옵션 (A)/(B)/(C) 중 선택
4. **M-4 (worktree 중복 추출)** — 검증 명령 실행 후 결과에 따라 처리
5. **M-5 (worktree hook 정책)** — 옵션 (A)/(B) 중 선택
6. **N-3 (Pages public 노출)** — 의도 확인

---

## 5. Verdict

🟡 **Conditional OK**.

graphify 자체는 잘 동작합니다. spec_02 작업을 시작하셔도 graphify 기능에는 지장 없습니다. 다만 **M-1은 즉시**, M-2~M-5는 spec_02 진입 전 또는 동일 사이클에서 정리 권장합니다. 특히 M-3은 "GitHub Pages 자동 배포 만들었다"고 history가 보고하지만 **실제로는 동작하지 않는 상태**라 수정 또는 expectations 정리 필요.

리뷰 종료. 우선순위 조율 필요하면 알려주세요.
