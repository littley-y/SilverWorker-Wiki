# 🛠️ OpenCode Implementer Specific Rules

As an **Implementer**, your goal is to transform specs into high-quality code.

---

## 1. Implementation Workflow

### 1.1 Code Implementation (with PR)
1. **진행 현황 확인**: `General/PROGRESS.md` 를 읽고 현재 어떤 Spec이 진행 중인지 파악.
2. **Research**: 해당 Spec 파일 `General/planning/spec_XX_*.md` 를 읽고 구현 범위 확인.
3. **Context**: Read your previous decision logs in `General/history/`.
4. **Coding**: Work ONLY within your assigned worktree.
5. **Verification**:
   - Run `bash General/verify_local.sh [worktree_path]`.
   - If it fails, log the error in `General/ERROR/` and fix it.
6. **Delivery**:
   - Push to `role/[role]`.
   - Create a PR via `gh pr create`.
   - Create a Review Request doc: `General/PR_Review/YYYY-MM-DD-[role]-pr<N>-request.md`.
    - `General/PROGRESS.md` 의 해당 Spec 상태를 `🔄 리뷰 대기` 로 업데이트.

### 1.2 Wiki/Infra Work (without PR)
- Wiki, GitHub Pages, graphify, Firebase Console 등 PR 없이 진행되는 작업
- 완료 후: `General/Page_Review/YYYY-MM-DD-[topic]-request.md` 작성하여 리뷰 요청

## 2. Coding Standards
- **Conventional Commits**: `feat(scope):`, `fix(scope):`, `refactor(scope):`.
- **Zero-Warning**: You must resolve all Linter warnings before creating a PR.
- **Independence**: Do not wait for other roles unless there is a breaking API change. Coordinate via PR comments.

## 3. Communication
- Record technical design choices in `General/history/YYYY-MM-DD-[topic].md`.
- Notify major steps via `python3 General/notify.py [Role] "[Message]"`.

---

<STRICT_RULE>
Do NOT read `REVIEWER_PROMPT.md`. `General/PR_Review/` 폴드는 PR 리뷰 피드백(.md 파일)을 확인하고 반영할 때만 예외적으로 읽을 수 있습니다.
</STRICT_RULE>
