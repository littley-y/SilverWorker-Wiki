# PR Review System (Strict Reviewer)

이 디렉토리는 각 에이전트의 기여에 대한 비판적이고 엄격한 코드 리뷰를 담고 있습니다.
모든 에이전트는 자신의 평가를 반드시 읽고, 지적 사항을 모두 해결한 후에만 다시 제출하거나 머지할 수 있습니다.

---

## 폴드 구조

`General/PR_Review/`는 **단일 폴드**입니다. 역할별 서브폴드는 사용하지 않습니다.
대신 **파일 제목에 브랜치(역할) 정보를 명시**합니다.

---

## 파일 명명 규칙

모든 리뷰 로그는 다음 형식을 따릅니다:

```
YYYY-MM-DD-[role]-pr<N>-review_[claude|gemini].md
```

| 요소 | 설명 | 예시 |
|---|---|---|
| `YYYY-MM-DD` | 리뷰 작성일 | `2026-04-24` |
| `[role]` | 대상 브랜치(역할) | `role_dev`, `role_design`, `role_devops`, `role_qa`, `master` |
| `pr<N>` | Pull Request 번호 | `pr23`, `pr7` |
| `review_[claude|gemini]` | 리뷰어 식별자 | `review_claude`, `review_gemini` |

### 예시
- `2026-04-24-role_dev-pr23-review_gemini.md` — Gemini가 role/dev 브랜치의 PR #23을 리뷰
- `2026-04-25-role_design-pr8-review_claude.md` — Claude가 role/design 브랜치의 PR #8을 리뷰
- `2026-04-26-master-pr30-review_claude.md` — Claude가 master 브랜치의 PR #30을 리뷰

---

## 리뷰 프로세스

1. **Implementer(OpenCode)**가 PR을 생성한다.
    - `gh pr create`로 GitHub PR 등록
    - `General/PR_Review/YYYY-MM-DD-[role]-pr<N>-request.md` 에 리뷰 요청 문서 작성 (Reviewer가 감지용)
    - `General/PROGRESS.md` 해당 Spec 상태를 `🔄 리뷰 대기` 로 업데이트
2. Reviewer(Claude/Gemini)가 코드를 검수한다.
    - `gh pr list` 또는 `General/PR_Review/`의 request.md 파일로 PR 감지
    - `gh pr view <PR#>` / `gh pr diff <PR#>`
    - `General/planning/`의 명세서와 비교
3. Reviewer가 GitHub PR에 리뷰를 먼저 등록한다.
    - `gh pr review <PR#> --request-changes|--approve --body "리뷰 완료. 상세 피드백은 General/PR_Review/...md 파일을 확인하세요."`
    - Implementer에게 GitHub 알림이 전달됨
4. Reviewer가 동일한 내용을 `General/PR_Review/`에 마크다운 로그로 저장한다.
    - 파일명: `YYYY-MM-DD-[role]-pr<N>-review_[claude|gemini].md`
5. **Implementer(OpenCode)**는 GitHub 알림을 받고 `General/PR_Review/`의 리뷰 파일을 읽어 내용을 반영하여 코드를 수정한다.
6. **승인(Approve)된 경우에만**: Reviewer가 `General/PROGRESS.md` 해당 Spec 상태를 `✅ 완료` 로 최종 업데이트.

---

## 리뷰 심각도 분류

| 등급 | 의미 | 머지 가능 여부 |
|---|---|---|
| **Blocker** | 치명적 버그, 보안 결함, 스펙 위반 | ❌ REJECTED |
| **Major** | 리팩토링 또는 추가 설명 필요 | ⏸️ PENDING |
| **Minor/Nit** | 선택적 개선 사항 | ✅ ALLOWED |
