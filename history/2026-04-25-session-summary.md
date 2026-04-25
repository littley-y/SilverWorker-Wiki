# 2026-04-25 — 전체 세션 요약 (Day 1 + Firebase 연동)

> **세션 기간**: 2026-04-25  
> **담당 역할**: Implementer (OpenCode) / dev  
> **세션 목표**: spec_01 완료 + Firebase Console 연동 + 리뷰 사이클 완료  
> **최종 상태**: ✅ spec_02 진입 가능 (모든 BLOCKER 해결)

---

## 타임라인

| 시간 | 작업 | 산출물 |
|---|---|---|
| 오전 | spec_01 코드 구현 | lib/ 디렉토리 구조 + 골격 코드 |
| 오전 | PR #1 생성 | `role/dev` → `master` |
| 오후 | Claude/Gemini v1 리뷰 | 리뷰 문서 2건 |
| 오후 | v1 리뷰 반영 (모델 재구성) | `ced434a` 커밋 |
| 오후 | PR #1 머지 | `2855bb4` + `ced434a` |
| 오후 | Firebase Console 연동 (사용자 협업) | 프로젝트 생성, 앱 등록, Phone Auth, Firestore |
| 저녁 | Claude/Gemini v2 리뷰 | 리뷰 문서 2건 |
| 저녁 | v2 리뷰 반영 (Minor nits) | `1a06824` 커밋 |
| 저녁 | Post-merge review 요청 | Firebase 연동 문서화 |
| 저녁 | Claude/Gemini post-merge 리뷰 | BLOCKER 발견 (`firestore.rules`) |
| 저녁 | **BLOCKER 긴급 수정** | `60a20a7` 커밋 |

---

## 커밋 이력 (master)

| 커밋 | 메시지 | 비고 |
|---|---|---|
| `2855bb4` | `feat(day1): spec_01 project setup skeleton` | 최초 PR #1 |
| `ced434a` | `fix(day1): align all models with 04_db_schema.md + address review feedback` | v1 리뷰 반영 |
| `1a06824` | `chore(day1): address v2 review nits — Claude approved` | v2 리뷰 반영 |
| `86242b4` | `chore(firebase): connect Firebase Console + flutterfire configure` | Firebase 연동 (master 직접 커밋) |
| `60a20a7` | `fix(security): restore firestore.rules to spec_01 §5 / 04_db_schema §3` | **BLOCKER 긴급 수정** |

---

## 생성된 문서

### PR Review
- `General/PR_Review/2026-04-25-role_dev-pr1-request.md` — PR #1 리뷰 요청
- `General/PR_Review/2026-04-25-role_dev-pr1-review_claude.md` — Claude v1 리뷰 (REJECTED)
- `General/PR_Review/2026-04-25-role_dev-pr1-review_gemini.md` — Gemini v1 리뷰 (APPROVED)
- `General/PR_Review/2026-04-25-role_dev-pr1-review_claude_v2.md` — Claude v2 리뷰 (Approve with notes)
- `General/PR_Review/2026-04-25-role_dev-pr1-review_gemini_v2.md` — Gemini v2 리뷰 (APPROVED)
- `General/PR_Review/2026-04-25-role_dev-postmerge-firebase-setup.md` — Post-merge 리뷰 요청
- `General/PR_Review/2026-04-25-role_dev-postmerge-firebase-setup-review_claude.md` — Claude post-merge (BLOCKER)
- `General/PR_Review/2026-04-25-role_dev-firebase-setup-review_gemini.md` — Gemini post-merge (ACCEPTED)

### History
- `General/history/2026-04-25-day1-review-fixes.md` — Day 1 리뷰 반영 상세
- `General/history/2026-04-25-firebase-setup.md` — Firebase 연동 상세

---

## 핵심 교훈

1. **모델 = DB 스키마의 1:1 매핑**: v1 리뷰에서 `UserModel`, `JobModel`, `ApplicationModel`이 `04_db_schema.md`와 불일치한 것이 가장 큰 실수였음. 향후 모든 모델 생성 시 스키마를 source of truth로 삼을 것.

2. **Firebase Console "테스트 모드" 위험성**: `firebase init` + `firebase deploy` 순서에서 Console의 테스트 모드가 로컬 rules를 덮어쓸 수 있음. 배포 후 반드시 Console에서 rules 내용 육안 확인 필요.

3. **flutterfire configure의 패키지명 자동 생성**: `build.gradle.kts`의 `applicationId`를 기준으로 자동 생성되므로, 사전에 Gradle 파일과 spec의 정합성을 확인해야 함.

---

## 미해결 항목 (spec_02 시작 시점 또는 별도 PR)

| ID | 내용 | 우선순위 | 비고 |
|---|---|---|---|
| M-1 | 패키지명 spec drift → spec_01 §3 갱신 | 중간 | |
| M-2 | Firebase Console orphan 클라이언트 삭제 | 낮음 | |
| M-3 | GCP Console API key application restriction 설정 | **중요** | Blaze 전환 전 반드시 |
| M-4 | `.gitignore` 정책 명시적 결정 | 낮음 | |
| N-1 | macOS bundle ID 오류 (`RunnerTests`) | 낮음 | MVP Android-only |
| N-2 | `firebase.json` prettify | 낮음 | |
| N-3 | `firestore.indexes.json` 복합 인덱스 5개 추가 | 중간 | spec_04 필수 |
| N-4 | `flutter run` 실제 기기 검증 | 중간 | spec_02 첫 작업 |

---

## 다음 세션 예정

- **spec_02 (인증 및 프로필 등록)**: PR #2로 분리하여 순차 진행
- **필요한 사용자 준비**: Android 에뮬레이터 또는 실제 기기
