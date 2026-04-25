# Post-Merge Review Request — Firebase Console 연동

> **대상 커밋**: `86242b4 chore(firebase): connect Firebase Console + flutterfire configure`  
> **브랜치**: `master` (직접 커밋) → `role/dev` 동기화 완료  
> **요청일**: 2026-04-25  
> **리뷰어**: Claude, Gemini

---

## 1. 작업 개요

spec_01 완료 후 spec_02 진입 전, Firebase Console 연동이 필수 선행 작업이었으나 사용자 대화 기반의 인프라 세팅 특성상 PR 사이클 없이 master에 직접 커밋되었습니다. 본 문서는 post-merge review를 요청합니다.

---

## 2. 변경 요약

### 2.1 Firebase 프로젝트 설정
- **프로젝트**: `silverworker-d8d64`
- **Android 패키지**: `com.silverworkernow.app`
- **Firestore**: `asia-northeast3` (서울), rules 배포 완료
- **Phone Auth**: 활성화 + 테스트 번호 (`+82 10-1234-5678` / `123456`)
- **요금제**: Spark (묶음) — Blaze는 spec_03 직전 deferred

### 2.2 코드 변경
| 파일 | 변경 내용 |
|---|---|
| `lib/firebase_options.dart` | FlutterFire CLI 생성. 5플랫폼(android/ios/macos/web/windows) 설정 포함 |
| `lib/main.dart` | `Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform)` 연결. v1 리뷰 M-1(try/catch placeholder) 완전 제거 |
| `android/app/google-services.json` | `com.silverworkernow.app`용 다운로드 및 배치 |
| `firestore.rules` | spec_01 §5 기준 rules, `firebase deploy --only firestore:rules`로 Console 적용 완료 |
| `.firebaserc`, `firebase.json`, `firestore.indexes.json` | Firebase CLI 프로젝트 설정 |

---

## 3. 리뷰 포인트

### 3.1 보안
- [ ] `google-services.json`에 **client secret이 노출**되어 있지 않은가? (Firebase Android config는 client-side key라 노출 OK이지만, 검토 필요)
- [ ] `firebase_options.dart`의 API key가 **제한 없이 노출**된 상태. Firebase Console → 프로젝트 설정 → API 및 서비스 → Android key 제한 설정이 필요한가?
- [ ] `firestore.rules`가 실제 Console에 배포되었는가? (`firebase deploy --only firestore:rules` 성공 확인)

### 3.2 설정 정확성
- [ ] `com.silverworkernow.app` vs `com.silverworker.now`: 패키지명 불일치가 spec_01 §3과 충돌하는가? `build.gradle.kts`의 `applicationId`를 따라간 것이 올바른 결정인가?
- [ ] `google-services.json`에 두 개의 클라이언트(`com.silverworker.now`, `com.silverworkernow.app`)가 공존. orphan 클라이언트 제거 필요?
- [ ] `firebase_options.dart`의 iOS/macos/web/windows 설정이 MVP 범위에서 **불필요한 노출**인가? Android-only로 제한 가능한가?

### 3.3 DoD
- [ ] `flutter analyze` 0경고는 확인되었으나, `flutter run`은 실제 기기에서 미검증. spec_02 진입 시 첫 빌드로 충분한가?
- [ ] `flutter doctor` 결과는 확인되지 않음 (사용자 환경에서 실행 필요).

### 3.4 정책
- [ ] `google-services.json`과 `firebase_options.dart`를 `.gitignore`에 추가할 것인가, 아니면 commit 유지할 것인가?
- [ ] Blaze 업그레이드를 spec_03까지 deferred한 것이 리스크인가?

---

## 4. 알려진 제한 사항

1. **빌드 미검증**: `flutter run`은 Android 에뮬레이터/기기 없이는 불가. spec_02 첫 빌드 시 검증 예정.
2. **SHA-1**: 디버그 키스토어만 등록됨. 릴리스 빌드 시 별도 등록 필요(spec_10 이후).
3. **iOS 미등록**: Console에는 iOS 앱이 등록되어 있으나 실제 iOS 빌드는 MVP에서 제외.

---

## 5. 리뷰 반영 및 수정 이력

### 5-1. Claude B-1 BLOCKER 수정 완료
- **문제**: `firestore.rules`가 Firebase Console "테스트 모드"에 의해 OPEN ACCESS 규칙으로 덮어쓰여짐
- **수정**: spec_01 §5 / 04_db_schema §3 규칙으로 복원 후 `firebase deploy --only firestore:rules` 재배포
- **커밋**: `60a20a7 fix(security): restore firestore.rules to spec_01 §5 / 04_db_schema §3`

### 5-2. 미해결 항목 (spec_02 시작 시점 또는 별도 PR로 처리)
- M-1: 패키지명 spec drift → spec_01 §3 갱신 필요
- M-2: Firebase Console orphan 클라이언트(`com.silverworker.now`) 삭제
- M-3: GCP Console API key application restriction 설정
- M-4: `.gitignore` 정책 명시적 결정
- N-1~N-4: macOS bundle ID, firebase.json formatting, firestore.indexes.json, flutter run 검증

---

## 6. 후속 작업

- 리뷰어 승인 시 별도 수정 없이 spec_02 진행
- Blocker/Major 발견 시 새로운 PR로 수정
