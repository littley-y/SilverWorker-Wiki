# Graphify Setup Review & Improvement Suggestions

**Reviewer**: Gemini
**Date**: 2026-04-25
**Target Document**: `General/history/2026-04-25-graphify-setup.md`

---

## 1. 종합 평가
AI 에이전트(Claude, OpenCode, Gemini)들을 하나로 묶고 프로젝트 전체의 컨텍스트를 지식 그래프화(Graphify)하여 공유한다는 발상과 아키텍처는 매우 훌륭합니다. 다만, 로컬 개발 환경의 성능 저하 및 CI/CD 자동화 관점에서 몇 가지 병목(Bottleneck)과 환경 의존성 이슈가 발견되어 개선을 제안합니다.

---

## 2. 주요 개선 제안 (Action Items)

### 2.1 Git Hook 블로킹 문제 (Performance Risk)
- **이슈**: `.git/hooks/post-commit`과 `post-checkout`에 `graphify` 빌드 명령어를 추가했습니다. 프로젝트 규모가 커질수록 커밋이나 브랜치 변경 시마다 수 초~수십 초의 딜레이가 발생하여 로컬 개발 생산성을 크게 떨어뜨립니다.
- **해결 방안**: 로컬 Git Hook을 제거하고, 코드가 원격 저장소에 Push 되었을 때 **GitHub Actions (`ci.yml`) 내부에서 Graphify를 실행하여 지식 그래프를 갱신**하도록 파이프라인을 변경하는 것을 강력히 권장합니다.

### 2.2 하드코딩된 로컬 경로 (Environment Dependency)
- **이슈**: `graphify update . --obsidian --obsidian-dir /mnt/e/SilverWorker-Graph` 명령어에 특정 작업자의 로컬 WSL 환경(`mnt/e/...`)에 종속된 절대 경로가 하드코딩되어 있습니다. 다른 개발자나 CI 서버에서는 무조건 실패합니다.
- **해결 방안**: Obsidian 연동은 개인 환경 전용 스크립트로 분리하거나, 해당 경로를 환경 변수(Environment Variable)로 처리하여 유연성을 확보해야 합니다.

### 2.3 GitHub Pages 자동화 누락 (Manual Step Risk)
- **이슈**: `graph.html`을 `SilverWorkerNow/docs/index.html`로 복사하는 과정이 수동으로 기재되어 있습니다. 그래프가 갱신되어도 누군가 복사 명령을 실행하지 않으면 배포된 사이트는 과거 상태에 머뭅니다.
- **해결 방안**: 생성하신 `.github/workflows/pages.yml` 내부의 Build Step에 파일 복사 로직을 추가하여 CI/CD 동작 시 자동으로 최신 HTML이 배포되도록 구성해야 합니다.
  ```yaml
  - name: Prepare Docs
    run: |
      mkdir -p SilverWorkerNow/docs
      cp graphify-out/graph.html SilverWorkerNow/docs/index.html
  ```

### 2.4 에이전트 초기화 정책 보완 (.gitignore 양면성)
- **이슈**: 플랫폼 종속 파일(`GEMINI.md`, `CLAUDE.md`)을 `.gitignore`에 추가한 것은 워킹트리 관리에 좋으나, 신규 팀원이 레포지토리를 클론(Clone) 받을 때 에이전트를 위한 컨텍스트 명령(Hook) 자체가 유실되는 문제가 생깁니다.
- **해결 방안**: 프로젝트 클론 시 에이전트 환경을 구성해주는 공통 초기화 스크립트(예: `init_agents.sh`)를 작성하고, `General/README.md` 등에 **"클론 후 반드시 `graphify <agent> install` 또는 초기화 스크립트를 실행하라"**는 온보딩 가이드를 명시해야 합니다.

---

## 3. 결론
위에서 제안한 자동화 및 로컬 환경 성능 최적화(Git Hook 제거, 하드코딩 탈피, CI 연동)만 반영된다면, 현재 프로젝트는 AI 에이전트와 완벽하게 상호작용하는 훌륭한 개발 환경으로 거듭날 것입니다.
