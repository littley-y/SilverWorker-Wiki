# Pre-Push Review — Wiki & General Repo Setup (Claude)

> **검토 대상**: `General/history/2026-04-25-wiki-setup.md` + 실제 staged 파일 + `.github/workflows/pages.yml`
> **검토 시점**: push 직전 (General/는 git init 후 48파일 staged 상태)
> **Reviewer**: Claude
> **Date**: 2026-04-25
> **Verdict**: 🟡 **Push 전 6건 정리 권장 — 보안 사고 없음, 정책 일관성·UX·동기 일치 이슈**

---

## 0. 종합

graphify와 wiki를 완전 분리한 결정은 합리적입니다 (이전 `pages.yml`이 working tree 외부 graphify-out 참조로 실패했던 문제를 정공법으로 해결). 다만 staged 파일 목록과 분리 동기("기획 문서 전용 위키")가 일치하지 않고, sidebar/cp 누락 + 마크다운 internal link 깨짐 등 UX 결함이 있습니다.

**push 자체를 막는 보안 사고는 없으나, scripts/config/.env.example 내용 확인은 push 전 필수**입니다 (public repo 가정).

---

## 1. ⚠️ Major

### M-1. `PR_Review/` 9개 파일이 staged됐지만 wiki에 노출 안 됨 — 일관성 결함

`pages.yml` line 32-35:
```yaml
cp -r planning _site/
cp -r history _site/
cp PROGRESS.md _site/
cp AGENTS.md _site/
```

`PR_Review/`가 cp 누락. 동시에 sidebar에도 등록 안 됨.

**결과**:
- git에는 public commit (누구나 GitHub repo의 raw 파일로 접근 가능)
- wiki UI에는 안 나타남 (sidebar/네비게이션 부재)

**영향**: PR_Review 파일들에는 firebase project ID(`silverworker-d8d64`), 패키지명, 보안 사고 분석(B-1 OPEN ACCESS), API key 정책 결정 등 정보가치 큰 내용이 포함됨. 외부 노출 시 의식적 결정 기록 필요.

**수정 옵션**:
- **(A) wiki 노출 의도** → `cp -r PR_Review _site/` 추가 + sidebar에 "PR Review" 섹션 추가 + `wiki-setup.md`에 "PR_Review는 투명한 리뷰 기록으로 wiki에 포함" 명시
- **(B) commit-only-archive 의도** → wiki-setup.md에 "PR_Review는 git에는 commit하되 wiki sidebar 노출 안 함 (이유: ...)" 명시

현 history §6 Notes에 "PR_Review/도 위키에 포함되어 있어 투명한 리뷰 기록 제공"이라 적혀 있으나, 실제 workflow는 미포함 → **history 문서와 실제 workflow가 모순**.

### M-2. wiki repo에 부적합한 파일들 staged — 분리 동기와 불일치

| 파일 | 분류 | 적합성 | 권장 |
|---|---|---|---|
| `scripts/hook_pre_git.py`, `intercept_git.py`, `init_session.py`, `cleanup_session.py`, `hook_agent_notify.py`, `hook_session_start.py`, `requirements.txt` | 코드 repo의 git/CI 자동화 hook | ❌ | unstage |
| `config/analysis_options.yaml`, `l10n.yaml` | Flutter 빌드 설정 | ❌ | unstage |
| `config/.env.example` | 환경변수 placeholder | ⚠️ | 내용 확인 후 결정 |
| `verify_local.sh` | CI 검증 스크립트 | ❌ | unstage |
| `notify.py` | 알림 스크립트 | ⚠️ | 내용 확인 (Slack webhook/이메일 등 개인정보 가능성) |
| `planning/AGENTS.md` | 기획 폴더 안의 별도 AGENTS.md | ⚠️ | top-level과 역할 명시 |

분리 결정의 핵심 동기("기획 문서 전용 위키")와 일치시키려면 `scripts/`, `config/`, `verify_local.sh`, `notify.py`는 **wiki repo에서 제외**가 맞다. 이들은 코드 repo(`SilverWorkerNow`)에 속함.

**수정 명령**:
```bash
cd /home/dudxo13/Projects/SilverWorker/General
git rm --cached -r scripts/ config/ verify_local.sh notify.py
echo -e "\n# Code-repo only (do not include in wiki)\nscripts/\nconfig/\nverify_local.sh\nnotify.py" >> .gitignore
```

### M-3. history sidebar — 신규 파일 3개 누락

`pages.yml` line 119-122의 history 섹션:
```html
<a class="nav-link" data-md="history/2026-04-25-session-summary.md">Session Summary</a>
<a class="nav-link" data-md="history/2026-04-25-day1-review-fixes.md">Day 1 Review Fixes</a>
<a class="nav-link" data-md="history/2026-04-25-firebase-setup.md">Firebase Setup</a>
<a class="nav-link" data-md="history/2026-04-25-graphify-setup.md">Graphify Setup</a>
```

**누락**:
- `wiki-setup.md` (이번 작업 기록)
- `graphify-setup-review_claude.md`
- `graphify-setup-review_gemini.md`

sidebar가 hardcoded라 자동 갱신 안 됨. push 전 4개 항목 추가 또는 자동 generation 도입.

### M-4. 마크다운 내부 링크 깨짐 — UX 결함

marked.js가 `[spec_02](spec_02_auth.md)` 링크를 그대로 `<a href="spec_02_auth.md">` 출력 → 사용자가 클릭하면 **fetch(.md) 후 raw markdown을 브라우저가 다운로드**하거나 직접 표시. nav 시스템(`data-md` 속성 + click handler)을 거치지 않음.

영향:
- `spec_01_project_setup.md` 안의 "참조: `overview/04_db_schema.md`" 링크 → 동작 안 함
- `PROGRESS.md`의 spec 1~10 표 링크 → raw .md 다운로드 또는 404

**수정 옵션**:
- **(A)** marked custom renderer 도입:
  ```js
  const renderer = new marked.Renderer();
  renderer.link = (href, title, text) => {
    if (href.endsWith('.md')) return `<a href="#" onclick="loadMarkdown('${href}');return false">${text}</a>`;
    return `<a href="${href}" target="_blank">${text}</a>`;
  };
  marked.setOptions({ renderer });
  ```
- **(B)** 빌드 단계에서 `.md` → `.html` 변환 (정적 사이트)
- **(C)** 사용자에게 사이드바 클릭만 사용하라고 안내 (UX 타협)

### M-5. General/ 별도 repo 분리 후 multi-worktree 동기화 정책 부재

history §3.1에서 `General/ → git init`. 그러나 multi-agent 워크플로우는 `SilverWorkerNow_dev/_design/...` 등 4개 worktree가 **filesystem path로 General/를 참조**한다는 가정에 의존.

다른 협업자가 SilverWorkerNow를 clone할 때:
- General/는 자동으로 안 옴 → AGENTS.md 기반 multi-agent orchestration 작동 안 함
- 별도 clone 필요 → setup 가이드가 어디에도 없음
- git submodule로 등록은 미적용

**수정**: General/`README.md`에 "이 repo는 SilverWorkerNow의 동반 wiki이며 별도 clone 필요. 협업자는 `git clone <silverworker-wiki-url> General/` 실행" 명시. 또는 SilverWorkerNow에 git submodule로 등록.

현재 history §2 결정("코드 프로젝트와 문서 프로젝트를 완전히 분리")은 단일 사용자 환경에서는 OK이지만 multi-agent/multi-collaborator 가정과 충돌. 후속 협업 시나리오 명시 필요.

### M-6. 환경별 파일 push 전 내용 검증 필수

push 직후 secret이 들어가면 git history에 영구 남음. 다음 파일은 push 전 1회 cat 검증 권장:

```bash
cat config/.env.example      # placeholder만 있어야 함, 실제 값 0개
cat notify.py                # Slack webhook URL/이메일/토큰 미포함 확인
cat scripts/hook_*.py        # 개인 경로 /home/dudxo13/... 하드코딩 확인
```

발견 시 (a) sanitize 후 commit OR (b) staging에서 제거.

---

## 2. 🟡 Minor

- **N-1**: `.gitignore`에 `graphify-out/` 추가 권장. 향후 General/만 대상으로 graphify 돌릴 경우 대비.
- **N-2**: sidebar에 검색 기능 없음. 47개 staged 파일 중 14개만 sidebar에 노출. 사용자가 나머지 발견 어려움. fuzzy search box 추가 또는 자동 sidebar generation 권장.
- **N-3**: dark theme 강제 (`prefers-color-scheme` 무시). 브라우저 설정 존중하는 light/dark 토글 추가 권장.
- **N-4**: `branches: [ master, main ]` 둘 다 추적 — `git init`의 default branch 명 모호성 대응으로 안전 (OK).
- **N-5**: `planning/AGENTS.md`와 top-level `AGENTS.md` 두 개 존재. 한쪽만 truth로 정리하거나 역할 차이 명시.
- **N-6**: 클라이언트-side rendering의 한계 — JS 비활성 환경에서 페이지 보이지 않음. 정적 사이트 generation 옵션도 고려 (M-4의 옵션 B와 연결).

---

## 3. ✅ 잘 처리된 부분 (Keep)

- **graphify ↔ wiki 분리 결정**: 이전 `pages.yml` 시도가 working tree 외부 graphify-out 참조로 실패한 문제를 정공법으로 해결. graphify는 AI 전용으로 격리, wiki는 사람용 문서 사이트로 분리.
- **`.obsidian/` ignore**: 개인 에디터 설정 분리 OK.
- **dark theme 자체**: 가독성·미감 좋음 (강제만 빼면).
- **client-side rendering with marked.js**: 빌드 의존성 최소화. cold start 빠름.
- **concurrency group**: pages 동시 deploy 충돌 방지 정확.
- **permissions** (`contents: read`, `pages: write`, `id-token: write`): GitHub Pages 표준 정확.
- **공개 결정 명시 (history §6)**: "GitHub Pages is public... a conscious decision" 한 줄로 N-3(이전 graphify-setup 리뷰의 public 노출 우려) 해소.

---

## 4. Push 전 권장 액션 (우선순위)

| 우선순위 | 작업 |
|---|---|
| 🛑 1 | `scripts/`, `config/`, `verify_local.sh`, `notify.py` unstage (M-2) — wiki 분리 동기 일치 |
| 🛑 2 | `config/.env.example`, `notify.py`, `scripts/*.py` 내용 확인 (M-6) — secret/개인정보 검사 |
| ⚠️ 3 | `PR_Review/` 노출 정책 결정 (M-1) — cp + sidebar 추가 OR commit-only 명시 |
| ⚠️ 4 | sidebar에 `wiki-setup.md`, `graphify-setup-review_{claude,gemini}.md` 추가 (M-3) |
| ⚠️ 5 | 마크다운 내부 링크 처리 (M-4) — 옵션 A/B/C 선택 |
| ⚠️ 6 | General/ 협업자 sync 가이드 README.md에 명시 (M-5) |
| 🟡 7 | `.gitignore`에 `graphify-out/` 추가 (N-1) |
| 🟡 8 | `planning/AGENTS.md` 정리 (N-5) |

---

## 5. Verdict

🟡 **Push 자체는 막을 수준 아님**. 다만 1·2번은 push 전 **반드시 확인**할 것을 강력 권고합니다. wiki repo가 public이라면 한 번 push된 secret은 git history에 영구 남아 외부 forking·archive·search engine indexing을 통해 회수 불가.

3~6번은 첫 push 후 follow-up commit으로 정리해도 OK이지만, 동일 사이클에서 묶는 편이 history 추적성에 좋습니다.

리뷰 종료.
