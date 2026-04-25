# KNOWLEDGE BASE — General/planning/

**Domain:** Project specifications, blueprints, test cases

## OVERVIEW
Master planning documents — source of truth for all feature requirements and QA criteria.

## STRUCTURE
```
planning/
├── AGENTS.md                    # 이 파일 — 디렉토리 안내
│
├── overview/                    # 전체 기획 개요 (고수준)
│   ├── 01_business_plan.md      # 사업계획서, 시장 분석
│   ├── 02_tech_stack.md         # 기술 스택 확정 문서
│   ├── 03_mvp_specs.md          # MVP 기능 명세 (우선순위 P0/P1/P2)
│   ├── 04_db_schema.md          # Firestore 컬렉션 구조 및 보안 규칙
│   └── 05_implementation_plan.md # 2주 Day별 개발 로드맵
│
├── spec_01_project_setup.md     # Flutter 구조, Riverpod, Firebase 초기 세팅
├── spec_02_auth.md              # 인증, 프로필 등록, 자동 로그인
├── spec_03_job_data.md          # Mock 데이터 / Cloud Functions API 프록시
├── spec_04_job_list_ui.md       # 공고 목록 화면, 카드 위젯, 필터
├── spec_05_job_detail.md        # 공고 상세, 세이프티 큐레이션 배지
├── spec_06_application.md       # 지원하기 흐름, 오터치 방어
├── spec_07_mypage.md            # 마이페이지, 지원 내역
├── spec_08_navigation.md        # go_router, Bottom Nav, 화면 전환
├── spec_09_ui_system.md         # 시니어 UI 디자인 시스템 (색상, 폰트, 터치 타겟)
└── spec_10_test_criteria.md     # 테스트 케이스, DoD, 데모 체크리스트
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| 기능 우선순위 확인 | `overview/03_mvp_specs.md` | P0=필수, P1=중요, P2=선택 |
| DB 컬렉션 구조 | `overview/04_db_schema.md` | Firestore 필드 및 보안 규칙 |
| 전체 일정 | `overview/05_implementation_plan.md` | Day별 산출물 |
| 화면별 구현 스펙 | `spec_01` ~ `spec_10` | Riverpod Provider, 위젯, 로직 포함 |
| 테스트 기준 | `spec_10_test_criteria.md` | 시나리오별 테스트 케이스 + 데모 체크리스트 |

## 진행 현황 확인

스펙 구현 전 반드시 `General/PROGRESS.md` 를 먼저 확인하세요.
어떤 Spec이 완료/진행 중/블로커 상태인지 한눈에 파악할 수 있습니다.

---

## CONVENTIONS
- **스펙 변경 시**: `overview/03_mvp_specs.md`와 해당 `spec_*.md` 양쪽 반영 필요
- **Day 기준**: 각 spec 파일 상단에 "대상 Day" 명시됨 — 구현 순서 참조
- **DoD**: 각 spec 파일 마지막 절에 체크리스트 형태로 정의

## ANTI-PATTERNS
- `spec_*.md`에 없는 기능을 임의로 구현하지 않는다
- `overview/`의 스펙과 `spec_*.md`가 충돌 시, `spec_*.md`가 우선 (더 구체적이고 최신)
- Mock 데이터는 `overview/04_db_schema.md`의 필드 구조를 완전히 따라야 한다
