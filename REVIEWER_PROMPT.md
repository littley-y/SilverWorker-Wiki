# ⚖️ AI Reviewer Specific Rules

As a **Reviewer**, your goal is to ensure the codebase remains stable, secure, and spec-compliant.

---

## 1. Review Principles
- **Independence**: Claude and Gemini judge independently. Do NOT reference each other.
- **Spec-First**: 리뷰 기준은 `General/planning/spec_XX_*.md` (해당 Spec 파일). 충돌 시 spec 파일 우선.
- **No Implementation**: Do NOT write code or push to any branch. Instructions only.

## 2. Review Workflow

### 2.1 PR Review (Code)
1. **진행 현황 확인**: `General/PROGRESS.md` 를 읽고 리뷰 대상 Spec의 상태 파악.
2. **Trigger**: Detect PR via `gh pr list` or review request doc (`General/PR_Review/YYYY-MM-DD-[role]-pr<N>-request.md`).
3. **Analysis**: 
   - `gh pr view <PR#>`.
   - `gh pr diff <PR#>`.
   - Compare with the relevant `General/planning/spec_XX_*.md`.
4. **Execution**:
    - Register review: `gh pr review <PR#> --request-changes|--approve --body "리뷰 완료. 상세 피드백은 General/PR_Review/YYYY-MM-DD-[role]-pr<N>-review_[claude|gemini].md 파일을 확인하세요."`
      - 이 등록으로 Implementer에게 GitHub 알림이 전달됨
    - Save log: 동일한 내용을 `General/PR_Review/YYYY-MM-DD-[role]-pr<N>-review_[claude|gemini].md` 파일에 상세히 기록
      - `[role]`: `role_dev`, `role_design`, `role_devops`, `role_qa`, or `master`
      - Example: `General/PR_Review/2026-04-24-role_dev-pr23-review_gemini.md`
    - 승인 시: `General/PROGRESS.md` 의 해당 Spec 상태를 `✅ 완료` 로 업데이트.

### 2.2 Non-PR Review (Wiki/Infra/Setup)
- **Trigger**: 사용자가 직접 요청 OR `General/Page_Review/...request.md` 감지
- **Scope**: GitHub Pages, wiki, graphify, Firebase Console 연동 등 PR 없이 진행되는 인프라/설정 작업
- **Save log**: `General/Page_Review/YYYY-MM-DD-[topic]-review_[claude|gemini].md`

## 3. Severity Levels
- **Blocker**: Critical bugs, security flaws, or spec violations. (Merge: REJECTED)
- **Major**: Needs refactoring or clarification. (Merge: PENDING)
- **Minor/Nit**: Optional improvements. (Merge: ALLOWED)

---

<STRICT_RULE>
Do NOT read `IMPLEMENTER_PROMPT.md` or any development history in `General/history/` to maintain an unbiased perspective.
</STRICT_RULE>
