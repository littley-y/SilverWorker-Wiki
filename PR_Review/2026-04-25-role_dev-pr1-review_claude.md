# PR Review — Day 1 / spec_01 (Claude)

> **PR**: #1 (`role/dev` → `master`)
> **Spec**: `General/planning/spec_01_project_setup.md`, `General/planning/overview/04_db_schema.md`
> **Reviewer**: Claude
> **Date**: 2026-04-25
> **Verdict**: ❌ **Request Changes (REJECTED)** — Blocker 4건

---

## 0. 종합 의견

뼈대 디렉토리 구조와 Riverpod Provider 시그니처(`authStateProvider`, `userProfileProvider`, `jobListProvider`, `myApplicationsProvider`)는 spec_01과 정확히 일치하며, `firestore.rules`도 `04_db_schema.md` §3과 1:1 동일하다. `flutter analyze` 0경고도 확인했다.

그러나 **3개 핵심 모델(`UserModel`, `JobModel`, `ApplicationModel`)이 모두 `04_db_schema.md`의 필드 정의와 어긋난다.** spec_01 자체는 모델 필드 명세를 직접 포함하지 않지만, 헤더에서 `overview/04_db_schema.md`를 명시 참조하고 있어 DB 스키마가 모델의 source of truth다. PR 본문 §4.1의 "Model 구조: `overview/04_db_schema.md`의 필드 정의와 1:1 매핑되는가?" 셀프 체크 항목에 대해 **현재 코드는 매핑되지 않는다**. Day 2(spec_02 인증/프로필 등록) 진입 시 Firestore에 저장할 필드 자체가 부족해 즉시 블록된다.

또한 spec_01 §6 DoD 1번(`flutter doctor` 무오류)과 3번(`flutter run` 빌드 성공)이 PR §3 검증 표에 누락 또는 "미시도"로 표기되어 있어 **DoD 미충족** 상태다.

---

## 1. Blocker

### B-1. `UserModel` — DB 스키마 필드 대거 누락
`lib/models/user_model.dart` (전체)

`04_db_schema.md` §2.1과 비교한 누락/변형 필드:

| 스키마 필드 | 타입 | 현재 코드 | 문제 |
|---|---|---|---|
| `gender` | string | 없음 | 누락 |
| `address` | map `{sido, sigungu}` | `region: String` | 구조 불일치 (단일 문자열로 평탄화) |
| `birthDate` | timestamp | `String` (YYYY-MM-DD) | 타입 불일치 |
| `physicalConditions` | array&lt;string&gt; | 없음 | 누락 — spec_04/05 필터링 불가 |
| `preferredJobTypes` | array&lt;string&gt; | 없음 | 누락 |
| `preferredLocations` | array&lt;string&gt; | 없음 | 누락 — 추천/필터 핵심 |
| `isPushEnabled` | boolean | 없음 | 누락 — spec_07 마이페이지 토글 |
| `createdAt` / `updatedAt` | timestamp | 없음 | 누락 |

**영향**: spec_02 프로필 등록 화면이 저장할 필드 자체가 정의되지 않음. spec_04/05의 "내 신체 조건/선호 지역 기반 필터링"은 모델에 데이터가 없어 구현 불가.

**수정 방향**: `04_db_schema.md` §2.1 필드를 1:1 추가. `address`는 별도 `Address` value object로 분리(`sido`, `sigungu`). `birthDate`/`createdAt`/`updatedAt`은 `Timestamp`(Firestore) ↔ `DateTime` 변환 처리.

---

### B-2. `JobModel` — DB 스키마와 구조적 불일치
`lib/models/job_model.dart` (전체)

`04_db_schema.md` §2.2와 핵심 차이:

| 스키마 필드 | 현재 코드 | 문제 |
|---|---|---|
| `source`, `companyAddress`, `locationCode`, `jobCategoryDetail` | 없음 | 누락 — `locationCode`는 §4 복합 인덱스의 ASC 키로 필수 |
| `employmentType` (enum: `part_time`/`daily`/...) | `workType: String` (한국어 자유 텍스트) | 명세된 enum이 아닌 자유 텍스트, 인덱스 키 호환 불가 |
| `salaryType` + `salaryAmount: number` | 단일 `salary: String` | 두 필드 병합 — 정렬/범위 쿼리 불가 |
| `jobCategory` (enum: `security_management` 등) | `jobCategory: String` (예: "경비/관리") | enum 코드 ≠ 한국어 라벨, 인덱스 키 불일치 |
| `physicalIntensity` (enum: `light`/`moderate`/`heavy`) | `String` (예: "가벼움"/"보통") | enum 위반, 인덱스 키 불일치 |
| `physicalBadges` (enum: `standing`/`sitting`/`heavy_lifting`/`outdoor`/`repetitive`/`stairs`) | `badges: List<String>` (한국어 자유 텍스트, 예: "계속 서있기") | 필드명·enum 모두 위반 |
| `workDays`, `workPeriod`, `requirements` | 없음 | 누락 — spec_05 상세 화면 표시 불가 |
| `benefits` | `welfare` | 필드명 변경 |
| `minAge`, `maxAge` | 없음 | 누락 |
| `deadline: timestamp` | 없음 | 누락 — §4 인덱스의 정렬 키 |
| `isActive: boolean` | 없음 | 누락 — §4 모든 인덱스의 ASC 키 |
| `rawData: map` | 없음 | 누락 — 고용24 원본 백업 |
| `createdAt`/`updatedAt` | 없음 | 누락 |

**영향**: §4의 모든 복합 인덱스가 현재 모델로는 작동하지 않음. spec_03(고용24 매핑), spec_04(필터/정렬), spec_05(상세+세이프티 배지) 전부 블록.

**수정 방향**: 한국어 라벨은 UI 표시 직전 변환 레이어(`JobLabelMapper` 등)에서 처리하고, 모델 자체는 스키마 enum 코드를 그대로 보존.

---

### B-3. `ApplicationModel` — status enum 및 컬렉션 경로 위반
`lib/models/application_model.dart:10`, `:23-34`

`04_db_schema.md` §2.3과 비교:

- **status enum 위반**: 코드는 `"접수"/"검토"/"합격"/"불합격"` 한국어 4개. 스키마는 `submitted`/`reviewing`/`accepted`/`rejected`/`cancelled` 영문 5개. 특히 **`cancelled`(지원 취소) 상태가 누락**되어 spec_06/07의 취소 기능을 막는다. 기본값 `'접수'` (line 30)도 영문 코드 `'submitted'`여야 한다.
- **`companyName` 누락**: 스키마에서 denormalized 표시 필드로 명시(§2.3). 마이페이지 지원 내역 카드에서 매번 jobs 컬렉션 조회가 필요해진다.
- **필드명 불일치**: `appliedAt` ≠ 스키마 `submittedAt`.
- **`updatedAt` 누락**: 상태 변경 추적 불가.

**수정 방향**: enum을 영문 코드로 통일, 한국어 라벨은 표시 레이어에서 매핑. denormalized 필드 추가.

---

### B-4. `bookmarks` 모델/Repository 골격 부재
`lib/models/`, `lib/repositories/`

`04_db_schema.md` §2.4에 `/users/{userId}/bookmarks/{jobId}` 컬렉션이 정의되어 있으나, spec_01에서 이를 명시 요구하진 않으므로 **단독 Blocker는 아니다**. 다만 spec_01이 §1에서 디렉토리 골격을 "데이터 모델" 단위로 요구하고 있어 이 PR에서 함께 잡거나, 명시적으로 "spec_05/06에서 추가"라는 주석/계획을 PR에 남기는 편이 후속 머지 노이즈를 줄인다. 본 항목은 위 B-1~B-3와 함께 수정될 때 같이 추가하기를 권장.

---

## 2. Major

### M-1. spec_01 §6 DoD 미충족 — 빌드/doctor 검증 누락
PR §3 검증 표:
- `flutter doctor` 결과 누락 (DoD 1번)
- `flutter run` "미시도" (DoD 3번)

PR §5.2에서 "Firebase 미연결로 미시도"라 했으나, spec §6은 "빈 화면이라도 OK"라고 명시한 빌드 검증이다. `firebase_options.dart`가 placeholder라면 `Firebase.initializeApp()`이 런타임에 실패해 앱이 뜨지 않는다. 두 가지 중 하나를 해야 한다:
1. `flutterfire configure`를 본 PR 범위에 포함시켜 실제 빌드 검증을 완료하거나,
2. `main.dart`에서 `Firebase.initializeApp()`을 try/catch로 감싸 placeholder 상태에서도 빌드만 통과하도록 임시 처리하고, **spec_02에서 즉시 제거**한다는 TODO 주석을 명시.

선택은 implementer 재량이지만, 현재 상태는 DoD 위반이다.

### M-2. PR 본문과 실제 코드의 의존성 버전 불일치
- PR §2.1: `cloud_functions: ^5.6.2`
- 실제 `pubspec.yaml:19`: `cloud_functions: ^5.4.0`

추가로 spec_01 §2는 `cloud_functions: ^4.x.x`로 명시. 메이저 버전 점프(4 → 5)는 의식적 선택일 수 있으나 PR에 근거가 없다. spec과 PR 본문, 실제 코드 세 곳을 정렬해야 한다.

### M-3. spec 외 의존성 다수 — 도입 근거 누락
`pubspec.yaml`에 spec_01 §2에 없는 의존성 다수:
- `flutter_slidable`, `flutter_local_notifications`, `timezone`, `flutter_timezone`, `google_mobile_ads`, `sqflite_common_ffi`, `cupertino_icons`

특히 `google_mobile_ads`(AdMob)는 `01_business_plan.md`/MVP 기획에서 명시 확인이 필요하다. 사용처 없는 의존성은 빌드 시간·앱 용량·리뷰 리스크(특히 광고 SDK는 권한·정책 영향)를 늘린다. 각 의존성에 대해 (a) 어느 spec에서 도입할지 또는 (b) 본 PR에서 제거할지 결정 필요.

### M-4. `JobFilter`의 spec_03~05 수용 능력
PR §4.4: "현재 `region`, `category`, `workType` 3개 필드". 그러나 spec_04/05의 핵심 필터는 `physicalConditions`(사용자) ↔ `physicalBadges`(공고) 매칭, `physicalIntensity`, `deadline` 정렬, `isActive=true` 등이다. 현 3필드로는 spec_04 진입 시 즉시 확장이 필요하다. 본 PR에서 `JobFilter`를 spec과 정렬시키거나, 최소한 확장 포인트(예: `Map<String, dynamic> extra`)를 명시하라.

### M-5. `fromJson`이 Firestore `Timestamp` 미처리
모든 모델의 `fromJson`이 날짜 필드를 `String`으로만 캐스팅(`DateTime.tryParse`). Firestore에서 `timestamp` 타입으로 저장된 값은 `Timestamp` 객체로 내려오며 `as String?` 캐스팅이 런타임 실패한다. spec_02 진입 시 즉시 터지는 결함이다. `Timestamp ⇨ DateTime` 변환 헬퍼를 모델에 포함하라.

---

## 3. Minor / Nit

- **N-1**: `UserModel`에는 `copyWith` 있고 `JobModel`에는 없음 — 일관성 통일 권장.
- **N-2**: `main.dart:24-32`의 placeholder Scaffold는 OK지만, 라우팅 도입 시점(spec_08?)을 주석으로 명시하면 좋다.
- **N-3**: `firestore.rules`의 `match /users/{userId}/{subcollection}/{docId}`는 `bookmarks/applications`를 함께 커버한다. 의도된 설계라면 한 줄 주석으로 명시.
- **N-4**: `JobFilter`의 위치/정의가 PR 본문에서 보이지 않음 — 코드 리뷰어가 빠르게 찾을 수 있도록 PR description에 파일 경로 명시.

---

## 4. 머지 결정

**REJECTED.** B-1~B-3을 수정하기 전까지 머지 불가. B-4와 Major는 동일 PR 사이클 내에서 함께 정리 권장.

리뷰 등록 후 implementer는 모델 3종 재구성 → `flutter analyze` 0경고 재확인 → 본 PR을 force-push 또는 후속 커밋으로 갱신 후 재요청 바람.

---

## 5. 좋았던 점 (Keep)

- 디렉토리 구조가 spec_01 §1과 거의 픽셀 단위로 일치.
- Provider 시그니처(`StreamProvider<User?>`, `FutureProvider.family`)가 spec §4와 정확히 동일.
- `firestore.rules`가 `04_db_schema.md` §3과 완전 동일.
- `flutter analyze` 0경고는 zero-warning policy를 처음부터 잘 잡았다.

후속 작업도 이 페이스로 가되, 모델 레이어는 **`overview/04_db_schema.md`를 단일 source of truth로 고정**하길 권장.
