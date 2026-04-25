# Spec 01. 프로젝트 초기 세팅

> 대상 Day: Day 1
> 참조: `overview/02_tech_stack.md`, `overview/04_db_schema.md`

---

## 1. Flutter 프로젝트 구조

```
lib/
├── main.dart
├── firebase_options.dart          # FlutterFire CLI 자동 생성
├── providers/                     # Riverpod 전역 Provider
│   ├── auth_provider.dart
│   ├── job_provider.dart
│   └── application_provider.dart
├── repositories/                  # Firestore 접근 계층
│   ├── auth_repository.dart
│   ├── job_repository.dart
│   └── application_repository.dart
├── models/                        # 데이터 모델 (freezed 미사용, 단순 클래스)
│   ├── user_model.dart
│   ├── job_model.dart
│   └── application_model.dart
├── screens/                       # 화면 단위
│   ├── auth/
│   ├── job/
│   ├── application/
│   └── mypage/
├── widgets/                       # 재사용 위젯
│   ├── job_card.dart
│   ├── badge_widget.dart
│   └── loading_overlay.dart
└── constants/
    ├── app_colors.dart
    └── app_text_styles.dart
```

---

## 2. pubspec.yaml 의존성

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.x.x
  firebase_auth: ^5.x.x
  cloud_firestore: ^5.x.x
  cloud_functions: ^4.x.x
  flutter_riverpod: ^2.x.x
  shared_preferences: ^2.x.x
  intl: ^0.19.x
  logger: ^2.x.x

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^4.x.x
```

> 버전은 pub.dev 최신 stable 기준으로 확정. `flutter pub get` 후 `flutter analyze` 0경고 확인.

---

## 3. Firebase 프로젝트 설정

| 항목 | 값 |
|---|---|
| 요금제 | **Blaze (종량제)** — Cloud Functions 사용을 위해 필수 |
| 앱 등록 | Android 패키지명: `com.silverworker.now` |
| SHA-1 | `keytool`로 디버그 키스토어 SHA-1 추출 후 Console에 등록 (Phone Auth 필수) |
| google-services.json | `android/app/` 에 배치 |

---

## 4. Riverpod Provider 설계

### auth_provider.dart
```dart
// 인증 상태 스트림
final authStateProvider = StreamProvider<User?>((ref) {
  return FirebaseAuth.instance.authStateChanges();
});

// 현재 유저 프로필
final userProfileProvider = FutureProvider.family<UserModel?, String>((ref, uid) {
  return ref.read(authRepositoryProvider).fetchProfile(uid);
});
```

### job_provider.dart
```dart
// 공고 목록 (필터 파라미터 포함)
final jobListProvider = FutureProvider.family<List<JobModel>, JobFilter>((ref, filter) {
  return ref.read(jobRepositoryProvider).fetchJobs(filter);
});

// 필터 상태
final jobFilterProvider = StateProvider<JobFilter>((ref) => JobFilter.empty());
```

### application_provider.dart
```dart
// 내 지원 내역
final myApplicationsProvider = FutureProvider.family<List<ApplicationModel>, String>((ref, uid) {
  return ref.read(applicationRepositoryProvider).fetchApplications(uid);
});
```

---

## 5. Firestore 보안 규칙

`firestore.rules` — `overview/04_db_schema.md` 3절 기준 그대로 적용:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    match /users/{userId}/{subcollection}/{docId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    match /jobs/{jobId} {
      allow read: if true;
      allow write: if false;
    }
    match /cached_api_responses/{cacheKey} {
      allow read: if true;
      allow write: if false;
    }
  }
}
```

---

## 6. 완료 기준 (Day 1 DoD)

- [ ] `flutter doctor` 오류 없음
- [ ] Firebase Console에서 Android 앱 연결 확인
- [ ] `flutter run` 빌드 성공 (빈 화면이라도 OK)
- [ ] `lib/providers/`, `lib/repositories/`, `lib/models/` 디렉토리 골격 생성
- [ ] `flutter analyze` 경고 0개
