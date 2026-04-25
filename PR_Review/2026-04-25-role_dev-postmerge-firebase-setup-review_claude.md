# Post-Merge Review — Firebase Console 연동 (Claude)

> **검토 커밋**: `86242b4 chore(firebase): connect Firebase Console + flutterfire configure`
> **요청서**: `General/PR_Review/2026-04-25-role_dev-postmerge-firebase-setup.md`
> **히스토리**: `General/history/2026-04-25-firebase-setup.md`
> **Reviewer**: Claude
> **Date**: 2026-04-25
> **Verdict**: 🛑 **BLOCKER — 즉시 수정 필요. 현재 상태는 보안 사고.**

---

## 0. 종합

post-merge 리뷰 요청서 §3.1의 자가 체크 항목 "`firestore.rules`가 실제 Console에 배포되었는가?"에 대해 **배포는 됐지만 배포된 내용이 spec_01 §5와 정반대인 "전체 공개" 데모 규칙**입니다. 현재 Firebase Console의 Firestore는 **인증 없이 누구나 모든 문서를 읽고 쓸 수 있는 상태로 30일간 노출 중**(2026-05-25까지). 이는 단순한 spec 위반이 아니라 **production-equivalent 환경의 보안 사고**이며, spec_02 진입을 막는 Blocker입니다.

추가로 패키지명 drift, API key 제한 미설정, gitignore 정책 미결정 등 Major 이슈가 있습니다.

좋은 소식: `main.dart`의 try/catch 제거, `DefaultFirebaseOptions.currentPlatform` 연결, Phone Auth 활성화, 테스트 번호 등록은 모두 정확히 처리되었습니다.

---

## 1. 🛑 BLOCKER

### B-1. `firestore.rules`가 spec_01 §5 규칙을 OPEN ACCESS 데모 규칙으로 **덮어쓰고 deploy까지 완료**됨

**파일**: `firestore.rules` (커밋 `86242b4`)

**현재 내용**:
```
match /{document=**} {
  allow read, write: if request.time < timestamp.date(2026, 5, 25);
}
```

**spec_01 §5 / 04_db_schema §3 요구사항**:
```
match /users/{userId}                  → auth.uid == userId
match /users/{userId}/{sub}/{doc}      → auth.uid == userId
match /jobs/{jobId}                    → read: true, write: false
match /cached_api_responses/{cacheKey} → read: true, write: false
```

**의미**:
- 2026-05-25까지 **인증 없이 누구나** 모든 컬렉션 읽기/쓰기 가능
- 누군가 `silverworker-d8d64` 프로젝트 ID를 알면(API key는 `firebase_options.dart`에 commit되어 GitHub public repo에 노출됨) Firestore 전체를 변조 가능
- `users` 컬렉션이 존재하기도 전이라 실데이터 유출은 없지만, **악의적 데이터 주입(예: jobs 컬렉션에 스팸 공고 대량 삽입)이 가능한 상태**
- spec_02에서 phone auth 사용자 데이터가 들어오는 순간부터는 PII 노출까지 가능

**원인 추정**:
사용자가 Firebase Console에서 Firestore Database를 생성할 때 "테스트 모드로 시작" 옵션을 선택 → Console이 자동 생성한 `firestore.rules`가 **로컬의 spec 규칙을 덮어씀** → `firebase deploy --only firestore:rules`가 그 덮어쓴 파일을 그대로 배포함. v1 리뷰에서 정확히 일치한다고 칭찬했던 규칙이 본 커밋에서 사라졌다.

**수정 절차** (즉시):
1. 로컬 `firestore.rules`를 spec_01 §5 / 04_db_schema §3 내용으로 복원
2. `cd /home/dudxo13/Projects/SilverWorker/SilverWorkerNow_dev && firebase deploy --only firestore:rules` 재실행
3. Firebase Console → Firestore Database → 규칙 탭에서 실제 배포된 내용이 spec과 일치하는지 **육안 확인**
4. Console 알림(만료 예정 30일 카운트다운)이 사라졌는지 확인

**검증 명령**:
```bash
# 배포된 rules를 다시 pull해서 로컬과 비교
firebase firestore:rules:get
```

이 단계가 끝나기 전까지 spec_02 진입 금지. 사용자 데이터를 OPEN bucket에 넣게 됨.

---

## 2. ⚠️ MAJOR

### M-1. 패키지명 spec drift — 결정은 됐으나 spec 문서 미갱신
- spec_01 §3: `com.silverworker.now`
- 실제 등록: `com.silverworkernow.app`
- 히스토리 §3-1에 결정 기록은 됨 (build.gradle.kts 따라감)

**문제**: spec_01 §3의 표가 그대로 남아있어, 미래의 누군가가 spec을 truth로 보고 패키지명을 되돌리려 시도할 수 있음. v2 리뷰의 M-2(cloud_functions 버전 drift)와 동일 패턴.

**수정**: spec_01 §3 표의 패키지명을 `com.silverworkernow.app`로 갱신하는 별도 PR 또는 본 커밋에 amend.

### M-2. Firebase Console에 orphan 클라이언트(`com.silverworker.now`)가 남아있고, `google-services.json`에 둘 다 포함됨

`google-services.json`을 보면 클라이언트 배열에 두 개가 공존:
```json
"client": [
  { "package_name": "com.silverworker.now", ... },   // orphan
  { "package_name": "com.silverworkernow.app", ... } // 실제 사용
]
```

**문제**:
- Firebase Android SDK는 패키지명 매칭으로 클라이언트를 고르므로 빌드는 동작
- 다만 orphan 클라이언트도 **동일한 API key**를 공유 → API key가 두 패키지명에 대해 유효
- 향후 `com.silverworker.now`로 누군가 사이드 빌드 시 동일 Firebase 프로젝트에 접근 가능 (key 공유)
- 깔끔하지 않고 디버깅 시 혼란

**수정**: Firebase Console → 프로젝트 설정 → 일반 → orphan Android 앱 삭제 → `flutterfire configure` 재실행하여 `google-services.json` 갱신.

### M-3. API key 제한(Android package + SHA-1)이 GCP Console에 적용되지 않음

post-merge 요청서 §3.1에서 자가 질문 ("API key가 제한 없이 노출된 상태")만 하고 답을 안 한 항목.

`firebase_options.dart`와 `google-services.json` 모두 GitHub public repo에 commit되어 **누구나 API key를 볼 수 있는 상태**입니다. Firebase의 client-side key는 "노출되어도 OK"라는 일반론이 맞지만, 그 전제는 **API key application restriction**(Android의 경우 패키지명 + SHA-1 fingerprint)이 GCP Cloud Console에서 설정되어 있을 때입니다.

**미설정 시 리스크**:
- 누군가 API key를 추출해 자신의 앱에서 silverworker-d8d64 Firebase에 요청 가능
- 무료 quota 소진 (Phone Auth SMS 10건/월 등)
- Blaze 전환 후에는 비용 폭탄 가능

**수정 절차**:
1. GCP Console → API 및 서비스 → 사용자 인증 정보 → Android key 선택
2. 애플리케이션 제한사항 → "Android 앱"
3. 패키지명 `com.silverworkernow.app` + 디버그 SHA-1 추가
4. (릴리스 시점에) 릴리스 SHA-1 추가
5. iOS key도 동일하게 bundle ID 제한
6. (선택) API 제한사항으로 사용 API만 화이트리스트 (Identity Toolkit, Firestore, Cloud Functions 등)

본 PR 범위는 아니지만 **B-1 수정과 동일 사이클에 처리할 것**을 강력 권장. 안 하면 Blaze 전환 시점에 비용 사고 가능.

### M-4. `.gitignore` 정책 결정 미수행 → 기본값(commit) 채택은 됐으나 명시적 결정 없음

post-merge 요청서 §3.4의 "정책 결정"이 답 없이 commit된 상태. v2 리뷰에서 내가 "보통 commit OK"라고 했고 implementer가 그대로 따라간 것은 OK지만, **이는 M-3(API key 제한) 적용을 전제로 한 결정**이었다. M-3 미설정 + commit 상태는 무방비 노출이다.

**대응**: M-3 적용 후 현 상태(commit 유지)가 정식 정책으로 확정되었다는 한 줄을 `history/`에 기록하거나 `firebase_options.dart` 헤더 주석에 명시.

---

## 3. Minor / Nit

### N-1. macOS 설정의 잘못된 bundle ID
`firebase_options.dart`:
```dart
static const FirebaseOptions macos = FirebaseOptions(
  ...
  iosBundleId: 'com.silverworkernow.app.RunnerTests',  // ← test target ID
);
```

`RunnerTests`는 iOS/macOS 테스트 타겟 bundle ID이지 앱 bundle ID가 아니다. flutterfire CLI가 macOS 자동 등록 시 잘못된 bundle을 매칭한 케이스. **MVP가 Android-only이므로 실질 영향 없음**. spec_10 이후 데스크톱 지원 시점에 정정 또는 macOS 설정 자체 제거.

### N-2. `firebase.json`이 한 줄 minified
diff 추적성·가독성 저하. `flutterfire configure`가 그렇게 생성한 것이 아니라 누군가 압축한 것으로 보임. `dart pub global run cli_util:format` 또는 단순 `jq . firebase.json | sponge firebase.json`로 prettify 권장.

### N-3. `firestore.indexes.json`이 비어있음
04_db_schema §4의 복합 인덱스 5개 미적용:
- `jobs`(locationCode, employmentType, isActive, deadline)
- `jobs`(jobCategory, physicalIntensity, isActive, deadline)
- `jobs`(isActive, deadline)
- `applications`(status, submittedAt)
- `cached_api_responses`(endpoint, expiresAt)

본 PR 범위는 아니지만, **spec_04 (공고 목록 UI) 첫 쿼리에서 즉시 필요**. spec_03 또는 spec_04 진입 시 추가하고 `firebase deploy --only firestore:indexes` 실행해야 함을 PROGRESS.md에 기록.

### N-4. `flutter run` 미검증 — 알려진 제한
사용자 환경에서 에뮬레이터 없으면 어쩔 수 없음. spec_02 첫 작업으로 `flutter run -d <device>` 시도 후 Firebase 초기화 정상 여부를 1차 확인할 것을 PROGRESS.md에 명시.

---

## 4. 잘 처리된 사항 (Keep)

- `lib/main.dart`: try/catch placeholder 깔끔하게 제거, `DefaultFirebaseOptions.currentPlatform` 정상 연결. v1 리뷰 M-1 완전 해결.
- Firebase Auth → Phone provider 활성화 + 테스트 번호(`+82 10-1234-5678` / `123456`) 등록. spec_02 리스크 표 대응 완료.
- Firestore 위치를 `asia-northeast3`(서울)로 선택. 한국 사용자 레이턴시 최적화.
- Blaze deferred 결정. spec_03 직전까지 결제수단 등록 안 해도 OK. 합리적.
- `google-services.json`을 정확한 경로(`android/app/`)에 배치. Gradle 플러그인이 자동 인식.
- 히스토리 문서 작성 깔끔. PR 없이 master 직접 커밋한 사유와 결정 과정을 명시한 점은 추적성 측면에서 좋음.
- post-merge review 요청서 작성 — PR 사이클을 우회한 작업도 리뷰 트랙에 올린 거버넌스 자세는 칭찬.

---

## 5. 머지/진입 결정

### 🛑 **spec_02 진입 차단.**

**즉시 수정 (B-1)**:
1. `firestore.rules`를 spec_01 §5 내용으로 복원
2. `firebase deploy --only firestore:rules` 재배포
3. Console 육안 검증 후 본 리뷰 파일에 "B-1 fixed in commit XXXXX" 추가 또는 신규 커밋 메시지에 명시

**B-1 fix 후 동일 사이클에서 권장 (M-1, M-2, M-3, M-4)**:
- spec_01 §3 패키지명 표 갱신
- Firebase Console orphan 클라이언트 삭제 + `flutterfire configure` 재실행 + `google-services.json` 갱신
- GCP Console에서 API key application restriction 적용 (Android: 패키지명 + 디버그 SHA-1)
- gitignore 정책 명시 한 줄 추가

**spec_02 진입 가능 조건**:
- B-1 해결 + Console에서 spec 규칙 배포 확인
- (M-3 미적용은 spec_02 개발 자체는 막지 않으나, **Blaze 전환 전까지는 반드시** 적용)

---

## 6. 사용자 결정 필요 사항

1. **패키지명 정책**: `com.silverworkernow.app`로 확정 → spec_01 §3 갱신할지, 아니면 패키지명을 `com.silverworker.now`로 되돌릴지 (`build.gradle.kts` 변경 + Firebase 재설정 비용 vs spec 갱신 1줄 비용)
2. **iOS/macOS/Web/Windows Firebase 등록 유지 여부**: 현재 5플랫폼 다 등록되어 `firebase_options.dart`에 다 있음. MVP가 Android-only라면 **iOS/macOS/Web/Windows를 Console에서 삭제하고 `flutterfire configure` 재실행**하여 `firebase_options.dart`를 Android-only로 슬림화 권장 (불필요한 attack surface 축소)
3. **B-1 수정 시점**: 지금 즉시 vs 내일 spec_02 작업 시작 시점
