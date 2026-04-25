# Wiki & General Repo Setup Review

**Reviewer**: Gemini
**Date**: 2026-04-25
**Target Document**: `General/history/2026-04-25-wiki-setup.md`, `General/.github/workflows/pages.yml`, `General/.gitignore`

---

## 1. 종합 평가
`General/` 폴더를 별도의 Git Repository로 분리하여 기획 문서와 히스토리 전용 위키(Wiki)로 운영한다는 아이디어는 훌륭합니다. 프로젝트 코드(SilverWorkerNow)와 문서의 관심사가 완벽히 분리되었으며, 별도의 무거운 정적 사이트 생성기(SSG) 없이 `marked.js`만을 활용해 클라이언트 사이드 렌더링을 구현한 것은 매우 가볍고 효율적인 접근입니다.

다만, 배포 파이프라인(`pages.yml`)과 무시 설정(`.gitignore`)에서 몇 가지 누락된 부분과 잠재적인 관리 리스크가 발견되어 수정을 제안합니다.

---

## 2. 주요 개선 제안 (Action Items)

### 2.1 PR_Review 및 최신 히스토리 문서 누락 (pages.yml)
- **이슈**: `history/2026-04-25-wiki-setup.md` 문서에는 `PR_Review/` 폴더의 내용도 위키에 포함하여 투명한 리뷰 기록을 제공한다고 명시되어 있으나, 실제 `pages.yml`의 빌드 스크립트와 HTML 사이드바 네비게이션에는 이 부분이 완전히 누락되어 있습니다. 또한, 방금 작성된 `2026-04-25-wiki-setup.md` 자체에 대한 링크도 없습니다.
- **수정 방안**:
  1. `pages.yml`의 `Build wiki site` 스텝 내 파일 복사 명령어에 `cp -r PR_Review _site/`를 추가하세요.
  2. `pages.yml`의 `index.html` 생성 스크립트 내 `<div id="sidebar-nav">` 영역에 다음과 같이 `PR Review` 섹션을 추가하고, 링크들을 연결하세요.
     ```html
     <div class="nav-section">
       <div class="nav-section-title">PR Reviews</div>
       <a class="nav-link" data-md="PR_Review/2026-04-25-role_dev-pr1-request.md">PR #1 Request</a>
       <a class="nav-link" data-md="PR_Review/2026-04-25-role_dev-pr1-review_claude_v2.md">PR #1 Claude Review</a>
       <a class="nav-link" data-md="PR_Review/2026-04-25-role_dev-pr1-review_gemini_v2.md">PR #1 Gemini Review</a>
       <!-- ... 기타 리뷰 문서들 ... -->
     </div>
     ```
  3. `History` 섹션에 `<a class="nav-link" data-md="history/2026-04-25-wiki-setup.md">Wiki Setup</a>` 링크를 추가하세요.

### 2.2 AI 에이전트 임시 파일 누락 방지 (.gitignore)
- **이슈**: `.obsidian/`을 제외한 것은 적절하나, `General/` 레포지토리 내에서 Claude나 Gemini 같은 AI 에이전트를 독립적으로 실행할 경우 생성되는 설정 및 로그 파일들이 Git에 포함될 수 있습니다.
- **수정 방안**: `.gitignore` 하단에 다음 내용을 추가하여 임시 파일 추적을 방지하세요.
  ```text
  # AI Agents
  .claude/
  .gemini/
  .superpowers/
  CLAUDE.md
  GEMINI.md
  
  # Logs & Envs
  *.log
  .env
  ```

---

## 3. 결론
Workflow 권한(`permissions: pages, id-token`)이나 배포 액션(`deploy-pages@v4`) 등 GitHub Pages 배포를 위한 핵심 인프라는 표준에 맞게 아주 잘 작성되었습니다. 

위에서 지적된 **파일 복사/링크 누락 문제와 `.gitignore` 보완**만 수행하신 후 `git commit --amend`로 변경 사항을 묶어서 Push 하시면 완벽한 Wiki 사이트가 될 것입니다!
