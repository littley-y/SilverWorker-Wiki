# Page Review — Wiki 배포 검증 + 거버넌스 문서 (Claude)

> **검토 대상**: `littley-y/SilverWorker-Wiki` Pages 배포 + General/AGENTS.md + REVIEWER_PROMPT.md + IMPLEMENTER_PROMPT.md
> **배포 URL**: https://littley-y.github.io/SilverWorker-Wiki/
> **Reviewer**: Claude
> **Date**: 2026-04-25
> **Verdict**: 🟢 **배포 정상 동작 + ⚠️ 거버넌스 문서 3건 정합성 이슈**

---

## 0. 종합

GitHub Pages 배포는 정상입니다. 직전 push의 workflow가 success(22초), 사이트는 https로 enforce되어 활성 상태(`https://littley-y.github.io/SilverWorker-Wiki/`). 이전 v1 리뷰의 M-2/M-3/M-4/N-1 모두 정확히 반영되었습니다.

다만 **거버넌스 문서 3건(AGENTS.md / REVIEWER_PROMPT.md / IMPLEMENTER_PROMPT.md)과 실제 폴더 구조 사이에 정합성 이슈**가 있고, **세션 시작 hook이 stale한 옛 프로젝트(MustGoOutNow / final_blueprint.md) 정보를 serving 중**입니다. 이는 AI 에이전트 오케스트레이션의 신뢰도를 직접 떨어뜨리는 문제라 별도 fix 권장.

---

## 1. ✅ 배포 작업 검증

### 1-1. Workflow 실행 결과
```
2026-04-25 (master push, 22s)  ✅ success — 직전 nav fix 커밋
2026-04-25 (workflow_dispatch, 23s)  ✅ success — wiki.html 분리
2026-04-25 (master push, 11s)  ❌ failure  — 초기 inline HTML 시도
2026-04-25 (master push, 0s)   ❌ failure  — 초기 setup 실패
```
2회 실패 → 2회 성공으로 안정화. 마지막 master push에 deploy 성공.

### 1-2. Pages 메타데이터 (정상)
| 항목 | 값 |
|---|---|
| URL | https://littley-y.github.io/SilverWorker-Wiki/ |
| build_type | workflow ✅ |
| https_enforced | true ✅ |
| public | true (의도적) |
| source | branch=master, path=/ |

### 1-3. v1 리뷰 항목 검증
| 항목 | 결과 |
|---|---|
| **M-2** scripts/config/verify_local.sh/notify.py 제외 | ✅ git ls-files 결과 40개 파일 모두 wiki 콘텐츠. 코드-repo 전용 파일 0개 |
| **M-3** sidebar 누락 4개 보완 | ✅ History 섹션 9개 모두 포함 (graphify-setup-review_{claude,gemini}, wiki-setup-review_{claude,gemini} 모두 등록) |
| **M-4** 마크다운 internal link 처리 | ✅ `currentPath` 추적 + `loadMarkdown(path, fromRoot)` 시그니처로 sidebar(absolute) vs in-doc(relative) 분기. baseDir prepend 정확 |
| **N-1** graphify-out gitignore | ✅ |

### 1-4. 구조 개선 (Keep)
- **HTML을 wiki.html로 분리** (e93cf32) — workflow를 165줄 → 50줄로 축소. YAML escape 문제 회피, diff 추적 명확.
- **Miro-inspired 디자인 시스템** — Inter+Noto Sans KR, 카드 기반 sidebar (border-radius 16px + box-shadow), 섹션별 색상 구분 (overview blue / planning teal / history coral / pr yellow). 가독성 ↑
- **Responsive** — `@media (max-width: 768px)`에서 sidebar 숨김, h1/h2 축소. 모바일 대응 OK
- **Light theme + 적절한 contrast** — 이전 다크 강제에서 light 기본으로 전환. 사용자 환경 가독성 ↑

---

## 2. ⚠️ Major — 거버넌스 문서 정합성

### M-1. **세션 시작 hook이 stale 콘텐츠 serving 중** 🚨

본 세션 시작 시 SessionStart hook이 inject한 orchestration map:
```
Project Name: MustGoOutNow  ← OLD
Master Spec: General/planning/final_blueprint.md  ← 존재하지 않는 파일
```

실제 `/home/dudxo13/Projects/SilverWorker/AGENTS.md`:
```
Project Name: SilverWorkerNow  ← 정확
Master Spec: General/planning/overview/03_mvp_specs.md + General/planning/spec_01~10.md
```

**의미**: AI 에이전트(Claude/Gemini/OpenCode)가 세션 시작 시 hook으로 받는 컨텍스트가 **구 MustGoOutNow 프로젝트의 final_blueprint.md를 master spec으로 신뢰**하게 됨. 실제로는 spec_01~10이 truth인데 hook은 단일 blueprint를 가리키므로 **에이전트가 spec_*.md를 무시하고 존재하지 않는 final_blueprint.md를 찾으러 갈 가능성**.

**원인 추정**: SessionStart hook이 `~/.claude/...` 또는 프로젝트 어딘가의 정적 텍스트 블록을 serving. SilverWorker 프로젝트로 fork/rename했지만 hook 콘텐츠는 미갱신.

**수정**:
1. SessionStart hook 정의 위치 추적 (보통 `~/.claude/settings.json` 또는 `.claude/settings.json`의 hooks 섹션)
2. `additionalContext` 안의 텍스트를 현재 `/SilverWorker/AGENTS.md` + `/REVIEWER_PROMPT.md` 내용으로 교체
3. `final_blueprint.md` → `spec_01~10.md` 명시
4. `MustGoOutNow` → `SilverWorkerNow`

이 fix가 안 되면 향후 모든 새 세션에서 동일 혼선 발생.

### M-2. **AGENTS.md 폴더 구조 spec ↔ 실제 불일치**

`AGENTS.md` line 45 (folder structure 섹션):
```
├── [role]/history/        # Agent session history
```

`REVIEWER_PROMPT.md` `<STRICT_RULE>`:
```
Do NOT read ... any development history in `General/[role]/history/`
```

`IMPLEMENTER_PROMPT.md` line 10, 27:
```
Read your previous decision logs in `General/[role]/history/`
Record technical design choices in `General/[role]/history/YYYY-MM-DD-[topic].md`
```

**3개 문서 모두 `[role]/history/` 패턴 (예: `General/dev/history/`, `General/qa/history/`) 명시.**

**그러나 실제 구조**:
```
General/
├── dev/         ← 빈 폴더 (legacy?)
└── history/     ← 모든 history 파일이 여기 flat 적재
```

직전 사이클들에서 모두 `General/history/`(flat)에 저장됨:
- `2026-04-25-firebase-setup.md`
- `2026-04-25-graphify-setup.md`
- `2026-04-25-wiki-setup.md`
- 등 9개 파일

→ **3개 거버넌스 문서가 거짓말을 하고 있음**. spec과 reality 중 하나를 선택해야:

**옵션 A — 거버넌스 문서를 reality에 맞춤** (권장):
- AGENTS.md folder structure: `[role]/history/` → `history/`
- REVIEWER_PROMPT.md `<STRICT_RULE>`의 path 갱신
- IMPLEMENTER_PROMPT.md line 10, 27 갱신
- 빈 `General/dev/` 디렉토리 삭제

**옵션 B — 실제 폴더를 spec에 맞춤**:
- 모든 history 파일을 `General/dev/history/`, `General/qa/history/` 등으로 재배치 (role 구분)
- 파일명 prefix(`role_dev-`, `role_qa-` 등) 도입

옵션 A가 압도적으로 비용 효율적. 본 프로젝트의 실제 작업 흐름이 단일 implementer (OpenCode for dev) + 2 reviewer (Claude, Gemini) 구조라 [role] 분리 필요성 낮음.

### M-3. **거버넌스 문서가 wiki/page review 트랙을 모름**

3개 문서 모두 **PR 리뷰만** 정의:
- AGENTS.md: `PR_Review/` 단일 폴더만 명시
- REVIEWER_PROMPT.md §2.2: `gh pr list`로 trigger, `gh pr review`로 register
- IMPLEMENTER_PROMPT.md §1.6: `gh pr create` + `PR_Review/...request.md`

그러나 본 사이클에서 발생한 작업들은 **PR 사이클 외부**:
- Firebase Console 연동 (master 직접 commit)
- graphify 설치 (인프라 작업, PR 없음)
- wiki repo 분리 (별도 repo)

이런 작업의 리뷰 트래킹 정책이 prompt에 없음. **사용자가 본 세션에서 `Page_Review/` 폴더를 새로 만들기로 결정한 것이 바로 이 gap을 메우려는 시도**.

**수정** (Page_Review 트랙 정식화):
1. AGENTS.md folder structure에 추가:
   ```
   ├── PR_Review/             # Code PR review feedback
   ├── Page_Review/           # Wiki/Pages/Infra review feedback (non-PR work)
   ```
2. REVIEWER_PROMPT.md §2 Workflow에 별도 트리거 명시:
   ```
   ### 2.5 Non-PR Review (Wiki/Infra/Setup)
   - Trigger: 사용자가 직접 요청 OR `General/Page_Review/...request.md` 감지
   - Save log: `General/Page_Review/YYYY-MM-DD-[topic]-review_[claude|gemini].md`
   ```
3. IMPLEMENTER_PROMPT.md §3에 wiki/infra 작업 시 review 요청 절차 추가:
   ```
   - Wiki/Infra 작업 완료 시: `General/Page_Review/YYYY-MM-DD-[topic]-request.md` 작성하여 리뷰 요청
   ```

---

## 3. 🟡 Minor — Wiki 자체

### N-1. marked.js 버전 미고정
`<script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js">` — latest 받아옴. v9+에서 `renderer.link`가 token 객체 받는 API로 breaking change 있었음. 향후 marked가 또 바꾸면 wiki 링크 전부 깨짐.

**수정**: `marked@15.0.0` 등 명시 버전 pin. 또는 SRI hash 함께 lock.

### N-2. Anchor 링크(`#section`) 처리 누락
`renderer.link`의 분기:
```js
if (typeof href === 'string' && href.endsWith('.md')) {
  return md-link 처리
}
return target="_blank" 처리
```

`#anchor` 같은 in-page anchor는 `.md`로 안 끝나므로 **`target="_blank"`로 빠짐 → 새 탭에 빈 페이지 열림**. spec 문서에서 `[자세히는 §3 참조](#section-3)` 류 링크가 있으면 깨짐.

**수정**: 첫 분기 위에 anchor 검사 추가:
```js
if (href.startsWith('#')) {
  return `<a href="${href}">${text}</a>`;  // browser native anchor
}
```

### N-3. 상위 경로(`..`) 마크다운 링크 미정규화
`baseDir + path` 단순 concat. 예: `currentPath = "history/foo.md"`, link = `../planning/spec.md` → `path = "history/../planning/spec.md"`. GitHub Pages는 서버에서 정규화하므로 fetch는 동작하지만, 다음 클릭 시 currentPath가 `history/../planning/spec.md`로 stored되어 그 다음 링크가 깨질 수 있음.

**수정**: `new URL(href, base).pathname` 사용해 정규화 후 저장.

### N-4. `firstLink.classList.add('active')` 중복
HTML에서 이미 첫 링크에 `class="nav-link active"` 적용되어 있음 (line 186). JS도 다시 add함 — idempotent라 OK이지만 코드 중복.

### N-5. 검색 기능 부재
47개 docs 중 sidebar 노출은 28개. 빠른 keyword 검색 없으면 사용자 탐색 어려움. `<input type="search">` + 마크다운 fulltext fetch + 매칭 highlight 도입 권장 (별도 사이클).

---

## 4. 거버넌스 문서별 정밀 분석

### 4-1. `AGENTS.md`
- ✅ 프로젝트명·spec 경로 정확 (SilverWorkerNow / spec_01~10)
- ✅ PROGRESS.md 우선 명시 (line 39)
- ✅ graphify 섹션 추가 (line 65-73) — graphify-out 활용 가이드 명확
- ⚠️ folder structure의 `[role]/history/` 표기가 reality와 불일치 (M-2)
- ⚠️ `Page_Review/` 미정의 (M-3)
- 🟢 Master Spec 우선순위 명시 (§3 line 60)

### 4-2. `REVIEWER_PROMPT.md`
- ✅ Spec-First 원칙 명확 (`spec_XX_*.md` 우선)
- ✅ Independence 원칙 (Claude/Gemini 독립)
- ✅ No Implementation 원칙
- ✅ Severity Levels (Blocker/Major/Minor)
- ⚠️ STRICT_RULE의 `[role]/history/` 경로가 reality와 불일치 (M-2)
- ⚠️ Non-PR 리뷰 트랙(Page_Review) 미정의 (M-3)
- ⚠️ §2.4의 "승인 시 PROGRESS.md를 ✅ 완료로 업데이트" — reviewer가 PROGRESS.md를 직접 수정하라는 뜻인데, 이는 "No Implementation" 원칙과 미세 충돌. PR merge 시 implementer가 갱신하는 것이 더 자연스러움. 정책 명확화 필요.

### 4-3. `IMPLEMENTER_PROMPT.md`
- ✅ 워크플로우 정확 (PROGRESS.md 확인 → spec 읽기 → 코딩 → verify → PR → review request)
- ✅ Conventional Commits 강제
- ✅ Zero-Warning 강제
- ✅ Independence (다른 role 대기 금지)
- ⚠️ `[role]/history/` 경로 reality 불일치 (M-2)
- ⚠️ Non-PR 작업(infra/wiki) 시 review 요청 절차 미정의 (M-3)
- ⚠️ STRICT_RULE의 "Do NOT read REVIEWER_PROMPT.md" — 본 사이클에서 implementer가 reviewer feedback에 정확히 대응한 결과를 보면 사실상 PR_Review 파일을 읽고 반영함 (예외 조항이 line 33에 명시되어 있긴 하나 명확성 ↑ 권장)

---

## 5. 권장 액션 (우선순위)

| 우선 | 작업 | 이유 |
|---|---|---|
| 🛑 1 | **SessionStart hook 갱신** (M-1) — MustGoOutNow → SilverWorkerNow / final_blueprint.md → spec_01~10.md | AI 에이전트의 모든 세션이 stale 컨텍스트로 시작됨 |
| ⚠️ 2 | **AGENTS.md / REVIEWER_PROMPT.md / IMPLEMENTER_PROMPT.md의 `[role]/history/` 경로 갱신** (M-2 옵션 A) | 3개 거버넌스 문서가 동시에 거짓말 중 |
| ⚠️ 3 | **Page_Review 트랙을 3개 문서에 공식화** (M-3) | 본 사이클에서 발생한 wiki/infra 리뷰 전체가 거버넌스에 누락 |
| 🟡 4 | marked.js 버전 pin (N-1) | 미래 breaking change 리스크 |
| 🟡 5 | anchor 링크 + 상위경로 정규화 처리 (N-2, N-3) | UX 결함 |
| 🟡 6 | 빈 `General/dev/` 디렉토리 삭제 또는 사용 정책 명시 | 구조적 모호성 |
| 🟡 7 | REVIEWER_PROMPT.md §2.4의 PROGRESS.md 업데이트 책임 명확화 | reviewer/implementer 책임 경계 |

---

## 6. Verdict

🟢 **배포 자체는 production 준비 완료**. https://littley-y.github.io/SilverWorker-Wiki/ 정상 운영.

⚠️ **거버넌스 정합성 fix 권장** — 우선순위 1·2·3은 spec_02 시작 전 정리하면 향후 수개월간 누적될 정합성 부채 예방. 30분 작업입니다.

리뷰 종료. 본 리뷰는 사용자 요청대로 `General/Page_Review/2026-04-25-wiki-deployment-review_claude.md`에 저장되었습니다.
