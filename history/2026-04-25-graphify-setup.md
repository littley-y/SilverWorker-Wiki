# Graphify Setup & Configuration

**Date**: 2026-04-25
**Session**: Graphify 지식 그래프 구축 및 3플랫폼 연동
**Status**: ✅ Completed

---

## 1. Goal

- SilverWorker 프로젝트의 지식 그래프(Knowledge Graph) 구축
- 3개 AI 플랫폼(Claude Code, OpenCode, Gemini CLI) always-on hook 설치
- Git post-commit hook으로 자동 재빌드
- GitHub Pages로 사람용 위키 호스팅

## 2. Installation

```bash
# graphify 설치 (user scope, sudo 불필요)
pip install --user graphifyy

# PATH 확인
export PATH="$PATH:$HOME/.local/bin"
```

## 3. Knowledge Graph Build

```bash
cd /home/dudxo13/Projects/SilverWorker

# Initial build with Obsidian export (optional, personal environment only)
# Set OBSIDIAN_DIR env var or use default local path
graphify update . --obsidian --obsidian-dir "${OBSIDIAN_DIR:-/mnt/e/SilverWorker-Graph}"
```

> **Note**: Obsidian export uses a local WSL path (`/mnt/e/...`). Other developers should set `OBSIDIAN_DIR` or skip this flag.

**Result**:
- Initial: 247 nodes, 242 edges, 52 communities (included duplicate worktree extractions)
- After `.graphifyignore` fix: 147 nodes, 139 edges, 29 communities
- Outputs: `graphify-out/graph.json`, `graphify-out/graph.html`, `graphify-out/GRAPH_REPORT.md`

## 4. Always-On Hooks (3 Platforms)

### Claude Code
```bash
graphify claude install
```
- Hook: `PreToolUse` (Glob|Grep matcher)
- Config: `.claude/settings.json`
- Behavior: 코드베이스 검색 전 GRAPH_REPORT.md 참고 안내

### OpenCode
```bash
graphify opencode install
```
- Plugin: `.opencode/plugins/graphify.js`
- Config: `.opencode/opencode.json`
- Behavior: tool.execute.before hook

### Gemini CLI
```bash
graphify gemini install
```
- Hook: `BeforeTool` (read_file|list_directory matcher)
- Config: `.gemini/settings.json`
- Behavior: 파일 읽기 전 그래프 컨텍스트 제공

## 5. Git Hooks (Auto-Rebuild)

```bash
# Master worktree
cd /home/dudxo13/Projects/SilverWorker/SilverWorkerNow
graphify hook install

# Dev worktree (worktrees share gitdir but have separate hooks directories)
HOOKS_DIR="/home/dudxo13/Projects/SilverWorker/SilverWorkerNow/.git/worktrees/SilverWorkerNow_dev/hooks"
mkdir -p "$HOOKS_DIR"
cp /home/dudxo13/Projects/SilverWorker/SilverWorkerNow/.git/hooks/post-commit "$HOOKS_DIR/"
chmod +x "$HOOKS_DIR/post-commit"
```

Installed:
- `SilverWorkerNow/.git/hooks/post-commit` → 코드 변경 시 자동 재빌드
- `SilverWorkerNow/.git/hooks/post-checkout` → 브랜치 체크아웃 시 재빌드
- `SilverWorkerNow/.git/worktrees/SilverWorkerNow_dev/hooks/post-commit` → dev worktree 커밋 시 재빌드

## 6. GitHub Pages Setup

### Workflow
Created: `SilverWorkerNow/.github/workflows/pages.yml`
- Trigger: `push` to `master` (docs/**, graphify-out/** paths)
- Build steps: install graphify → rebuild graph (`--no-llm`) → copy `graph.html` to `docs/index.html`
- Action: `actions/deploy-pages@v4`
- Source: `./docs` directory

### Docs Directory
```bash
mkdir -p SilverWorkerNow/docs
cp graphify-out/graph.html SilverWorkerNow/docs/index.html
```

> **Note**: GitHub Pages is public. The graph contains Firebase project IDs, firestore.rules, and spec/review content. This is acceptable for an OSS project but should be a conscious decision.

## 7. Project Root Cleanup

Issue: `graphify * install` creates platform-specific markdown files (GEMINI.md, CLAUDE.md) in project root.

Solution:
```bash
rm GEMINI.md CLAUDE.md
```

> **Note**: `.gitignore` was initially used but corrected. These files belong in `.graphifyignore` (prevents graphify self-extraction), not `.gitignore`. `AGENTS.md` must remain tracked by git — it is the Master Orchestration Document.

Final `.graphifyignore` includes:
```
AGENTS.md
CLAUDE.md
GEMINI.md
.graphifyignore
SilverWorkerNow_dev/
SilverWorkerNow_design/
SilverWorkerNow_devops/
SilverWorkerNow_qa/
```

## 8. Verification

| Check | Status |
|-------|--------|
| graphify-out/graph.json exists | ✅ |
| graphify-out/GRAPH_REPORT.md readable | ✅ |
| Claude PreToolUse hook active | ✅ |
| OpenCode plugin registered | ✅ |
| Gemini BeforeTool hook active | ✅ |
| Git post-commit hook installed (master) | ✅ |
| Git post-commit hook installed (dev) | ✅ |
| GitHub Pages workflow committed | ✅ |
| docs/index.html copied | ✅ |
| .graphifyignore has worktree exclusions | ✅ |
| Duplicate worktree nodes removed | ✅ (147 vs 247) |

## 9. Next Steps

1. GitHub Pages 활성화 (Repository Settings → Pages → Source: GitHub Actions)
2. Git push 후 workflow 동작 확인
3. spec_02 (인증 및 프로필 등록) PR #2 시작

## 10. Post-Review Fixes

Applied after Claude/Gemini review:
- **M-1**: Removed `AGENTS.md` from `.gitignore` → added to `.graphifyignore` only
- **M-2**: `google-services.json` commit 유지로 복귀 + GCP API key restriction 권장. `.gitignore`에서 `android/app/google-services.json` 제거, `git rm --cached` 후 `git add`로 재트래킹
- **M-3**: Added graphify build step to `pages.yml` for automatic deployment
- **M-4**: Added worktree exclusions to `.graphifyignore`, rebuilt graph (147 nodes)
- **M-5**: Manually installed post-commit hook in `SilverWorkerNow_dev` worktree
- **N-4**: Corrected workflow path in history to `SilverWorkerNow/.github/workflows/`
- **Gemini-2.2**: Noted Obsidian path as personal-environment-only

## 10. References

- `.graphifyignore` — Graphify 추출 제외 파일 목록
- `graphify-out/` — 지식 그래프 출력물
- `SilverWorkerNow/docs/` — GitHub Pages 소스
