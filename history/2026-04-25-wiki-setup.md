# Wiki & General Repo Setup

**Date**: 2026-04-25
**Session**: 사람용 위키 구축 + General/ 별도 git repo 분리
**Status**: ✅ Completed (pending push)

---

## 1. Background

기존 구조에서는 사람용 위키(GitHub Pages)를 `SilverWorkerNow/docs/`에 두려 했으나 다음 문제가 있었음:
- `General/`에 있는 기획/스펙/히스토리 문서를 `docs/`로 중복 복사해야 함
- `SilverWorker/`는 git repo가 아니라서 GitHub Actions로 배포 불가
- graphify(지식 그래프)는 AI 전용이므로 사람용 위키에 포함할 필요 없음

## 2. Decision

**General/를 별도 git repo로 분리**하여 기획 문서 전용 위키로 운영.
코드 프로젝트(`SilverWorkerNow/`)와 문서 프로젝트(`General/`)를 완전히 분리.

## 3. Changes

### 3.1 SilverWorker/General/ → git repo 초기화
```bash
cd /home/dudxo13/Projects/SilverWorker/General
git init
```

### 3.2 GitHub Pages workflow 이동
- **삭제**: `SilverWorkerNow/.github/workflows/pages.yml`
- **생성**: `General/.github/workflows/pages.yml`
  - 사람용 위키 전용 (graphify 제외)
  - marked.js 클라이언트 렌더링 + **Miro-inspired 디자인 시스템** 적용
  - `PROGRESS.md`, `planning/`, `history/`, `AGENTS.md` 포함

#### Design System (Miro-inspired)
- **Typography**: Inter (geometric sans-serif) + Noto Sans KR
- **Colors**: White canvas (#ffffff), Near-black text (#1c1c1e), Blue 450 (#5b76fe) interactive
- **Pastel accents**: Coral, Teal, Pink, Yellow for section headers
- **Elevation**: Ring shadow border (`rgb(224,226,232) 0px 0px 0px 1px`)
- **Radius**: 8px (links), 12px–16px (cards), 12px (code blocks), 16px (sidebar sections)

### 3.3 Cleanup
- `SilverWorkerNow/docs/` 삭제
- `SilverWorkerNow/_wiki_source/` 삭제
- `General/.obsidian/` → `.gitignore` 추가 (개인 에디터 설정)

#### 3.4 .gitignore
```
# OS
.DS_Store
Thumbs.db
desktop.ini

# IDE
.idea/
.vscode/
*.iml

# Obsidian (personal editor config)
.obsidian/

# AI Agents (auto-generated context files)
.claude/
.gemini/
.superpowers/
CLAUDE.md
GEMINI.md

# Code-repo only (not part of wiki content)
scripts/
config/
verify_local.sh
notify.py

# Graphify output (AI-only knowledge graph)
graphify-out/

# Logs & Environments
*.log
.env
```

## 4. Staged Files

General/ repo에 `git add -A` 완료. 총 ~35개 파일 스테이지:
- `.github/workflows/pages.yml`
- `README.md`
- `AGENTS.md`
- `PROGRESS.md`
- `planning/` (overview + spec_01~10)
- `history/` (7개 파일)
- `PR_Review/` (9개 파일)

**Unstaged (code-repo only)**: `scripts/`, `config/`, `verify_local.sh`, `notify.py`

## 5. Next Steps (Pending)

1. GitHub에 새 repo 생성 (`SilverWorker-Wiki` 등)
2. Remote 추가 후 push:
   ```bash
   git remote add origin https://github.com/dudxo13/SilverWorker-Wiki.git
   git push -u origin master
   ```
3. GitHub Pages 활성화 (Settings → Pages → Source: GitHub Actions)
4. Wiki 확인: `https://dudxo13.github.io/SilverWorker-Wiki/`

## 6. Notes

- `.obsidian/`은 GitHub에 push하지 않음 (Obsidian 개인 설정)
- graphify 지식 그래프는 `SilverWorker/` parent 폰더에 남아 있음 (AI 전용)
- `PR_Review/`도 위키에 포함되어 있어 투명한 리뷰 기록 제공

## 7. Post-Review Fixes (Claude + Gemini)

Applied after review:
- **M-1**: `PR_Review/` added to workflow (`cp -r PR_Review _site/`) + sidebar "PR Reviews" section with all 8 review files
- **M-2**: `scripts/`, `config/`, `verify_local.sh`, `notify.py` unstaged from wiki repo (code-repo only)
- **M-3**: Added missing history files to sidebar: `wiki-setup.md`
- **M-4**: Custom marked renderer for `.md` internal links — clicks now route through `loadMarkdown()` nav system instead of raw download
- **M-5**: `README.md` rewritten for separated wiki repo — includes collaborator setup guide and multi-worktree sync instructions
- **M-6**: Verified no secrets in unstaged files (`.env.example` = placeholder, `notify.py` = no hardcoded tokens, scripts = personal paths only)
- **Gemini-2.2**: `.gitignore` expanded with AI agent files (`.claude/`, `.gemini/`, `.superpowers/`, `CLAUDE.md`, `GEMINI.md`) and `graphify-out/`
- **N-1**: `graphify-out/` added to `.gitignore`

---

**Review Requested**: Claude + Gemini
