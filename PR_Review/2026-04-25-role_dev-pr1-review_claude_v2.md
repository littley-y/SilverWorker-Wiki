# PR Re-Review — Day 1 / spec_01 (Claude, v2)

> **PR**: #1 (`role/dev` → `master`)
> **Reviewed commit**: `ced434a fix(day1): align all models with 04_db_schema.md + address review feedback`
> **Reviewer**: Claude
> **Date**: 2026-04-25 (post-fix)
> **Verdict**: ✅ **Approve with non-blocking notes**

---

## 0. 종합

v1 리뷰에서 지적한 Blocker 4건(B-1~B-4)이 모두 코드 레벨에서 해결되었고, Major 5건도 합리적인 방식으로 처리되었다. 머지를 막는 결함은 없다.

다만 implementer 코멘트의 자가 보고와 실제 코드 사이에 작지만 짚어둘 정직성·문서화 이슈 몇 가지가 있으니 spec_02 진입 전에 정리하길 권장.

---

## 1. Blocker 검증 — 모두 해결

| ID | 파일 | 검증 결과 |
|---|---|---|
| **B-1 UserModel** | `lib/models/user_model.dart` | ✅ `Address{sido,sigungu}` value object 분리, `birthDate: DateTime?`, `gender`, `physicalConditions/preferredJobTypes/preferredLocations`, `isPushEnabled`, `createdAt/updatedAt` 모두 추가. `TimestampHelper` 적용. **04_db_schema §2.1과 1:1 매핑 확인.** |
| **B-2 JobModel** | `lib/models/job_model.dart` | ✅ 누락 필드 전수 보완(`source/companyAddress/locationCode/jobCategoryDetail/employmentType/salaryType/salaryAmount/workDays/workPeriod/requirements/benefits/minAge/maxAge/deadline/isActive/rawData/createdAt/updatedAt`). `physicalIntensity`/`physicalBadges`가 영문 enum 코드로 변경됨. **DB 스키마 §2.2 일치.** |
| **B-3 ApplicationModel** | `lib/models/application_model.dart` | ✅ `status` 영문 5종 enum(`submitted/reviewing/accepted/rejected/cancelled`), `companyName` denormalized 추가, `appliedAt`→`submittedAt` 개명, `updatedAt` 추가. `userId` 필드는 의도적으로 제거(스키마 §2.3 경로가 `/users/{userId}/applications/{applicationId}`로 path-encoded이므로 올바른 결정). |
| **B-4 Bookmark** | `lib/models/bookmark_model.dart`, `lib/repositories/bookmark_repository.dart` | ✅ §2.4와 정확히 일치. `BookmarkRepository`의 `fetch/save/delete` 시그니처도 합리적. |

---

## 2. Major 검증

### M-1 DoD (`flutter run`/`flutter doctor`) — 합리적 우회 ✅
`main.dart:11-15`에서 `Firebase.initializeApp()`을 try/catch로 감싸고 `TODO(spec_02)` 주석 추가 — 내가 v1에서 제시한 옵션 2 그대로다. placeholder 상태에서도 빌드가 통과하므로 DoD 충족. `flutter run` 직접 검증 결과는 PR 코멘트에 없으나, `analyze 0경고`만 보고된 것은 v1 대비 아쉬움 — 다음 PR부터는 `flutter run --no-pub -d <device>` 또는 `flutter build apk --debug` 한 줄이라도 검증 표에 포함해주면 좋겠다.

### M-2 의존성 버전 — **부분 해결, spec 정렬 미흡** ⚠️
- ✅ PR 본문(`^5.6.2`) ↔ 실제 `pubspec.yaml`(`^5.6.2`) 정렬됨
- ❌ **spec_01 §2의 `cloud_functions: ^4.x.x`, `flutter_lints: ^4.x.x`와 여전히 불일치**

implementer는 PR 코멘트에서 "실제 `flutter pub get` resolved stable로 정렬"이라 했지만, 이는 spec_01 §2와의 메이저 버전 차이를 정당화하지 않는다. 진실은 다음 둘 중 하나다:
1. spec_01 §2가 작성 시점에 outdated였고, firebase_core ^3.x 생태계가 cloud_functions ^5.x를 요구하므로 현재 pubspec이 옳다 → **spec_01 §2를 ^5.x.x로 갱신하는 별도 PR 필요**
2. spec이 옳고 ^4.x로 다운그레이드해야 한다 → 현재 pubspec이 틀림

선택은 implementer의 판단이지만, **현 상태는 "코드와 spec 둘 다 truth"가 공존하는 위험한 모호함이다.** 이번 PR을 머지하더라도 후속 작업에서 spec drift 정리 PR 1건을 추가로 받아야 한다.

`flutter_lints`도 동일 이슈(`^5.0.0` vs spec `^4.x.x`).

### M-3 spec 외 의존성 — 처리됐으나 정확성 ⚠️
주석으로 도입 근거를 표기한 점은 좋다. 다만 `pubspec.yaml`의 다음 코멘트는 사실과 다르다:

```yaml
# Notifications (spec_01 §5 — FCM companion)
flutter_local_notifications: ^17.0.0
```

`spec_01 §5`는 **Firestore 보안 규칙** 절이지 FCM/알림과 무관하다. 알림 관련 spec은 spec_02 또는 후속 spec(spec_07 마이페이지의 푸시 토글 등)에서 다뤄질 가능성이 높다. **존재하지 않는 spec 절을 인용하는 것은 리뷰 추적성에 해롭다.** 정확한 spec 위치를 확인해 코멘트를 수정하거나, 아직 명시 도입 spec이 없다면 `TODO(spec_TBD): notifications 도입 spec 확정 필요`로 솔직하게 표기하라.

`google_mobile_ads`의 TODO는 정직하고 좋다.

### M-4 JobFilter — 충분 ✅
`locationCode`, `jobCategory`, `employmentType`, `physicalIntensity`, `isActive` 5필드로 확장. spec_04 §4 인덱스 키와 매핑된다. 단, **`physicalConditions`(사용자) → `physicalBadges`(공고) 매칭 로직**과 **`deadline` 정렬**은 아직 없다. 04_db_schema §1의 "클라이언트 단 2차 필터링" 노트가 있으니 spec_04 진입 시 해당 로직을 명확히 잡아야 한다 — 본 PR에서는 deferred 처리해도 OK.

### M-5 Timestamp — 정확 ✅
`lib/utils/timestamp_helper.dart`는 `Timestamp`/`String`/null 3케이스를 모두 처리한다. 정적 추상 final 클래스로 깔끔하다. 다만 `fromDateTime`이 `Timestamp?`를 반환하므로 `toJson()` 결과는 순수 JSON이 아니다(Firestore 전용). `shared_preferences` 등 비-Firestore 직렬화에 그대로 못 쓴다는 점은 spec_02에서 문제 될 수 있으나, MVP 범위에서는 수용 가능.

---

## 3. 이번 라운드에서 새로 발견한 사항 (Minor)

### N-1. 잘못된 spec 인용 (위 M-3에 포함, 명시화)
`pubspec.yaml`의 `spec_01 §5 — FCM companion` 코멘트는 사실 오류. 머지 전 1줄 수정 권장.

### N-2. `copyWith`의 nullable 필드 패턴
`UserModel.copyWith({DateTime? birthDate, ...})`에서 `birthDate: birthDate ?? this.birthDate`는 **null로 되돌릴 수 없는 패턴**(Dart의 흔한 함정). 의도적으로 birthDate를 지우는 케이스가 spec_07 마이페이지 등에서 생기면 작동하지 않는다. `JobModel.deadline/minAge/maxAge`, `ApplicationModel.submittedAt/updatedAt`, `BookmarkModel.bookmarkedAt`도 동일. 본 PR 범위는 아니지만 인지하길.

### N-3. `salaryAmount: double` vs 스키마 "number"
스키마 §2.2 예시는 `12000`(정수). 원화에는 소수가 없으므로 `int`가 더 정확하지만, `double`도 작동한다. spec_03 고용24 매핑 시 정수로 좁히는 편을 추천(JSON에서 `12000.0`으로 저장되는 부수효과 방지).

### N-4. `JobFilter`가 `lib/models/`가 아닌 `lib/repositories/job_repository.dart`에 정의됨
spec_01 §1 디렉토리 구조와 의미적으로 불일치(filter는 도메인 모델). `lib/models/job_filter.dart`로 이동 권장. Minor.

### N-5. `lib/repositories/bookmark_repository.dart` Provider 부재
`BookmarkRepository`가 만들어졌으나 `lib/providers/`에 `bookmarkRepositoryProvider`/`myBookmarksProvider`가 없다. spec_05/07에서 추가될 가능성이 높지만, 본 PR에서 골격만이라도 함께 잡으면 일관성이 좋아진다. Minor.

---

## 4. 머지 결정

### ✅ Approve

남은 이슈는 **모두 비차단성**(spec drift 정리, 코멘트 정확성, 패턴 개선)이며, spec_02 진입을 막지 않는다. spec_01 자체의 DoD는 성립한다(디렉토리 구조, Provider 시그니처, firestore.rules, 0-warning).

### 머지 전 1분 작업 권장 (선택)
- `pubspec.yaml`의 `spec_01 §5 — FCM companion` 코멘트 수정 또는 `TODO(spec_TBD)` 변경.

### 머지 후 후속 작업 (필수)
- **spec_01 §2 갱신 PR**: `firebase_core ^3.x` 생태계 기준으로 `cloud_functions: ^5.x.x`, `flutter_lints: ^5.x.x`로 spec 자체를 갱신하거나, 별도 ADR(architecture decision record)에 예외 사유 기록. 이걸 안 하면 spec과 코드가 영구히 불일치한 채 남는다.

### Day 2 진입 시 우선 점검
1. `lib/utils/timestamp_helper.dart`의 `fromDateTime` 결과가 비-Firestore 경로(예: `shared_preferences`)에서 직렬화 가능한지.
2. `JobFilter`의 `physicalConditions` ↔ `physicalBadges` 매핑 정책 확정.
3. `flutterfire configure` 실행 후 `main.dart`의 try/catch 제거.

---

## 5. v1 대비 평가

implementer가 v1 리뷰의 Blocker·Major를 매우 빠르게(~12분) 그리고 정확하게 반영했다. 모델 3종이 spec과 1:1 정렬된 것은 후속 spec 전체의 안정성을 크게 끌어올렸다. spec drift 정리만 별도 트랙으로 이어가면 Day 2 이후 작업 속도가 더 붙을 것으로 보인다.

Keep it up.
