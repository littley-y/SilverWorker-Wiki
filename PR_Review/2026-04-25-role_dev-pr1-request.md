# PR Review Request — Day 1 / spec_01

> **PR**: [#1](https://github.com/littley-y/SilverWorker/pull/1)  
> **Branch**: `role/dev` → `master`  
> **Spec**: `General/planning/spec_01_project_setup.md`  
> **Requested**: 2026-04-25  
> **Author**: Implementer (OpenCode)

---

## 1. 변경 요약

Day 1 (spec_01) 프로젝트 초기 세팅 완료. Flutter + Firebase 기반 시니어 구인구직 앱 MVP의 뼈대 코드를 생성했습니다.

---

## 2. 주요 변경 사항

### 2.1 의존성 (`pubspec.yaml`)
- `firebase_core: ^3.13.0`
- `firebase_auth: ^5.7.0`
- `cloud_firestore: ^5.6.12`
- `cloud_functions: ^5.6.2`
- 기존: `flutter_riverpod`, `logger`, `intl`, `shared_preferences`, `flutter_slidable` 등 유지

### 2.2 `lib/` 디렉토리 구조 (spec_01 100% 준수)

```
lib/
├── main.dart                          # ProviderScope + Firebase.initializeApp
├── firebase_options.dart              # FlutterFire CLI placeholder
├── models/
│   ├── user_model.dart                # name, birthDate, region, careerSummary, phoneNumber
│   ├── job_model.dart                 # title, company, salary, workHours, badges, intensity...
│   └── application_model.dart         # status(접수/검토/합격/불합격), selfIntroduction
├── constants/
│   ├── app_colors.dart                # WCAG AA 대비 색상 팔레트
│   └── app_text_styles.dart           # 18pt+ 기반 시니어 폰트 스타일
├── repositories/
│   ├── auth_repository.dart           # fetchProfile, saveProfile
│   ├── job_repository.dart            # fetchJobs, fetchJobById (skeleton)
│   └── application_repository.dart    # fetchApplications, submitApplication
├── providers/
│   ├── auth_provider.dart             # authStateProvider (StreamProvider<User?>)
│   ├── job_provider.dart              # jobFilterProvider (StateProvider) + jobListProvider
│   └── application_provider.dart      # myApplicationsProvider (FutureProvider.family)
├── screens/
│   ├── auth/login_screen.dart
│   ├── auth/profile_register_screen.dart
│   ├── job/job_list_screen.dart
│   ├── job/job_detail_screen.dart
│   ├── application/application_form_screen.dart
│   └── mypage/my_page_screen.dart
└── widgets/
    ├── job_card.dart
    ├── badge_widget.dart
    └── loading_overlay.dart
```

### 2.3 보안 규칙 (`firestore.rules`)

```
/users/{userId}                      → auth.uid == userId
/users/{userId}/{subcollection}      → auth.uid == userId
/jobs/{jobId}                        → read: true, write: false
/cached_api_responses/{cacheKey}     → read: true, write: false
```

---

## 3. 검증 결과

| 항목 | 명령 | 결과 |
|---|---|---|
| 의존성 해결 | `flutter pub get` | ✅ 성공 |
| 정적 분석 | `flutter analyze` | ✅ **0 issues** |
| 빌드 | `flutter run` | ⚠️ 미시도 (Firebase Console 미연결) |

---

## 4. 리뷰 포인트 (Reviewer Focus)

### 4.1 아키텍처
- [ ] **Riverpod Provider 설계**: `authStateProvider`를 `StreamProvider<User?>`로 구현한 방식이 spec_01과 일치하는가?
- [ ] **Repository 패턴**: Firestore 직접 접근 vs. Cloud Functions 프록시 간의 경계가 명확한가?
- [ ] **Model 구조**: `overview/04_db_schema.md`의 필드 정의와 1:1 매핑되는가?

### 4.2 코드 품질
- [ ] **Zero-Warning**: `flutter analyze` 0경고 유지가 지속 가능한 구조인가?
- [ ] **Type Safety**: `as String? ?? ''` 기본값 처리 방식이 적절한가?
- [ ] **const 생성자**: Model/Widget에서 `const` 사용이 충분한가?

### 4.3 보안
- [ ] **Firestore Rules**: 실제 인증 흐름(Firebase Phone Auth)과 연동 시 누락된 규칙은 없는가?
- [ ] **캐시 컬렉션**: `/cached_api_responses` write 권한이 `false`로 고정되어 있는데, Cloud Functions에서의 쓰기는 어떻게 처리할 것인가?

### 4.4 확장성
- [ ] **freezed 미사용**: spec_01에서 "freezed 미사용, 단순 클래스"로 명시했으나, `copyWith`를 수동 구현했음. 유지보수 시 문제는 없는가?
- [ ] **JobFilter**: 현재 `region`, `category`, `workType` 3개 필드. spec_03~05에서의 필터 요구사항을 모두 수용할 수 있는가?

---

## 5. 알려진 제한 사항

1. **Firebase Console 미연결**: `google-services.json` 및 `firebase_options.dart`가 아직 실제 Firebase 프로젝트와 연동되지 않음. Day 2 (spec_02) 진행 시 `flutterfire configure` 필요.
2. **빌드 미검증**: `flutter run`은 Firebase 초기화 실패로 인해 아직 시도하지 않음. skeleton 코드 수준의 구조 검증만 완료.
3. **플랫폼 파일 변경**: `android/app/build.gradle.kts`, `ios/Runner.xcodeproj` 등은 `flutter pub get` 실행 시 자동 수정된 것으로, 본 PR의 핵심 변경사항은 아님.

---

## 6. 다음 단계

리뷰 승인 후 `role/dev` → `master` 머지 → Day 2 (spec_02: 인증 및 프로필 등록) 진행 예정.
