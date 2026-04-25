# KNOWLEDGE BASE — General/

**Domain:** Shared non-code assets — specs, configs, scripts, history

## OVERVIEW
Central repository for all non-code project assets. Shared across all 5 worktrees.

## STRUCTURE
```
General/
├── PROGRESS.md               # ★ 개발 진행 현황 대시보드 — 세션 시작 시 반드시 확인
├── planning/                 # Specs, blueprints, test cases
│   ├── overview/             # 고수준 기획 문서 (01~05)
│   ├── spec_01~10.md         # 구현 세부 스펙 (Day별)
│   └── AGENTS.md             # planning 디렉토리 안내
├── config/                   # Shared Flutter config
│   ├── analysis_options.yaml # Lint rules (zero-warning policy)
│   └── l10n.yaml             # Localization config
├── scripts/                  # Automation scripts
│   ├── sync_worktrees.sh     # Post-merge worktree sync
│   ├── hook_pre_git.py       # Blocks direct master push
│   └── hook_post_pr_merge.py # Auto-syncs worktrees after merge
├── history/                  # Session history (all roles, flattened)
├── ERROR/                    # Fatal error logs
├── PR_Review/                # Gemini reviewer feedback
├── docs/                     # Reference documentation
├── verify_local.sh           # CI validation script (pub get → format → analyze → test)
└── notify.py                 # Notification utility
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| **현재 진행 상황** | `PROGRESS.md` | **세션 시작 시 첫 번째로 확인** |
| 기능 우선순위 | `planning/overview/03_mvp_specs.md` | P0/P1/P2 기준 |
| 구현 세부 스펙 | `planning/spec_01~10.md` | Day별 화면/로직/DoD |
| 전체 일정 | `planning/overview/05_implementation_plan.md` | 2주 Day별 로드맵 |
| CI validation | `verify_local.sh` | Run before any PR |
| Worktree sync | `scripts/sync_worktrees.sh` | Post-merge sync |
| Lint config | `config/analysis_options.yaml` | Zero-warning policy |
| Error logs | `ERROR/` | Track fatal errors |
| Review history | `PR_Review/` | Reviewer feedback |

## CONVENTIONS
- **NEVER** edit `General/` from within a worktree — it's the shared source
- **History files**: `General/history/YYYY-MM-DD-[topic].md`
- **Error logs**: `General/ERROR/YYYY-MM-DD-[issue].md`
- **Config files**: Shared in `General/config/`, referenced by all worktrees

## ANTI-PATTERNS
- **NEVER** duplicate specs across worktrees — single source in `General/planning/`
- **NEVER** skip `verify_local.sh` before PR push
- **NEVER** edit l10n generated files — they're auto-generated from `l10n.yaml`
