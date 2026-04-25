# GitHub Pages Deployment & Wiki Setup Review

**Reviewer**: Gemini
**Date**: 2026-04-25
**Target Files**: `General/.github/workflows/pages.yml`, `General/wiki.html`, `General/AGENTS.md` 등

---

## 1. 전반적인 평가 (Overview)
`General` 폴더를 별도의 Wiki 레포지토리로 분리하여 `pages.yml`과 `wiki.html`을 통해 클라이언트 사이드 렌더링을 훌륭하게 구현했습니다. 이전 리뷰에서 제안된 PR_Review 문서 누락 문제도 `wiki.html`의 사이드바 라우팅과 함께 완벽하게 해결되었습니다.

하지만 요청하신 **AI 에이전트 연관 문서(`AGENTS.md`, `REVIEWER_PROMPT.md`, `IMPLEMENTER_PROMPT.md`) 반영 여부**를 비판적으로 분석한 결과, 치명적인 구조적 누락이 발견되었습니다.

---

## 2. 에이전트 문서 반영 여부 비판적 분석 (Critical Analysis)

### ✅ 2.1 `AGENTS.md` (반영 완료)
- `General/AGENTS.md` 파일이 존재하며, `pages.yml`의 `cp AGENTS.md _site/` 명령어로 정상 복사됩니다.
- `wiki.html` 사이드바의 `Overview` 섹션에도 `AGENTS.md`가 정상적으로 링크되어 있어 위키에서 바로 열람이 가능합니다. 완벽합니다.

### ❌ 2.2 `REVIEWER_PROMPT.md` (누락 및 아키텍처 충돌)
- **이슈**: 현재 이 파일은 `General/` 폴더 안이 아니라 부모 디렉토리인 **프로젝트 루트(`SilverWorker/`)에 존재**합니다.
- **영향**: `General/` 폴더를 별도의 Git Repository로 분리했기 때문에, GitHub Actions(`pages.yml`)가 돌 때 상위 폴더에 있는 `REVIEWER_PROMPT.md`에 접근할 수 없어 **위키에 배포가 불가능**합니다.
- **`wiki.html` 누락**: 당연히 사이드바 네비게이션에도 링크가 하드코딩되어 있지 않습니다.

### ❌ 2.3 `IMPLEMENTER_PROMPT.md` (완전 누락)
- **이슈**: 프로젝트 전체 디렉토리를 검색해본 결과, 현재 **`IMPLEMENTER_PROMPT.md` 파일 자체가 존재하지 않습니다** (혹은 `.gitignore` 등으로 소실됨).
- **영향**: 에이전트 오케스트레이션의 핵심인 Implementer 규칙이 없으며, 위키 배포 및 네비게이션(`pages.yml`, `wiki.html`)에서도 완전히 누락되어 있습니다.

---

## 3. 개선 제안 및 Action Items (How to Fix)

위키가 프로젝트의 "단일 진실 공급원(Single Source of Truth)" 역할을 온전히 수행하려면, 에이전트들의 프롬프트 파일도 `General/` 레포지토리 내에서 관리되고 위키에 노출되어야 합니다.

**Action Item 1. 프롬프트 파일 마이그레이션 및 복원**
- 부모 폴더에 있는 `REVIEWER_PROMPT.md`를 `General/` 내부로 이동시키세요. (예: `General/planning/REVIEWER_PROMPT.md` 또는 `General/AGENTS.md`와 동일 레벨)
- 유실된 `IMPLEMENTER_PROMPT.md`를 찾아(또는 새로 작성하여) `General/` 내부에 저장하세요.

**Action Item 2. `pages.yml` 배포 스크립트 갱신**
- 파일들이 `General/` 루트에 있다면 아래 복사 명령어를 추가하세요.
  ```bash
  cp REVIEWER_PROMPT.md _site/
  cp IMPLEMENTER_PROMPT.md _site/
  ```

**Action Item 3. `wiki.html` 사이드바 갱신**
- `wiki.html`의 `<div id="sidebar-nav">` 내에 에이전트 프롬프트를 위한 메뉴를 추가하거나 Overview 쪽에 묶어서 넣어주세요.
  ```html
  <div class="nav-section">
    <div class="nav-section-title overview">Agent Prompts</div>
    <a class="nav-link" data-md="AGENTS.md">AGENTS.md</a>
    <a class="nav-link" data-md="REVIEWER_PROMPT.md">Reviewer Prompt</a>
    <a class="nav-link" data-md="IMPLEMENTER_PROMPT.md">Implementer Prompt</a>
  </div>
  ```

---

## 4. 결론
현재 Wiki 시스템 렌더링 아키텍처는 매우 매끄럽게 구축되었으나, `General` 레포지토리의 경계(Boundary) 문제로 인해 **리뷰어 및 임플리멘터 프롬프트 문서가 위키 배포 흐름에서 통째로 소외(Orphaned)되는 병목**이 존재합니다. 해당 문서들을 `General/` 폴더 안으로 편입시키고 `wiki.html`에 링크를 연결해주시면 진정한 완성이 될 것입니다.
