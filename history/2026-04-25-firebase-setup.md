# 2026-04-25 — Firebase Console 연동 세션 (Post-spec_01)

> **세션 목표**: Firebase 프로젝트 생성 + Android 앱 등록 + flutterfire configure + 빌드 검증  
> **특이사항**: 본 작업은 spec_01의 DoD 보강 및 spec_02 진입 전 필수 선행 작업으로, **master 브랜치에 직접 커밋**됨 (PR 없이).  
> **관련 스펙**: `spec_01_project_setup.md` §3, §6, `spec_02_auth.md` 선행 요건

---

## 1. 체크리스트 완료 현황

| # | 작업 | 담당 | 상태 |
|---|---|---|---|
| 1 | Firebase 프로젝트 생성 | 사용자 | ✅ |
| 2 | Android 앱 등록 (`com.silverworkernow.app`) | 사용자 | ✅ |
| 3 | Authentication → Phone provider 활성화 | 사용자 | ✅ |
| 4 | 테스트 전화번호 등록 (`+82 10-1234-5678` / `123456`) | 사용자 | ✅ |
| 5 | Firestore Database 생성 + `firebase deploy --only firestore:rules` | 사용자 + 구현자 | ✅ |
| 6 | `google-services.json` 다운로드 및 `android/app/` 배치 | 사용자 + 구현자 | ✅ |
| 7 | Firebase CLI / flutterfire CLI 설치 | 구현자 | ✅ |
| 8 | `flutterfire configure` 실행 | 구현자 | ✅ |
| 9 | `main.dart` 정리 (try/catch 제거 + `DefaultFirebaseOptions.currentPlatform`) | 구현자 | ✅ |
| 10 | `flutter analyze` 0경고 확인 | 구현자 | ✅ |

---

## 2. 프로젝트 정보

| 항목 | 값 |
|---|---|
| **Firebase 프로젝트 ID** | `silverworker-d8d64` |
| **프로젝트 번호** | `174114586214` |
| **Android 패키지명** | `com.silverworkernow.app` |
| **Firestore 위치** | `asia-northeast3` (서울) |
| **요금제** | Spark (묶음) — Blaze는 spec_03 직전 업그레이드 예정 |
| **등록된 플랫폼** | Android, iOS, macOS, Web, Windows |

---

## 3. 주요 이슈 및 결정 사항

### 3-1. 패키지명 변경
- **원래 계획**: `com.silverworker.now` (spec_01 §3 표기)
- **실제 등록**: `com.silverworkernow.app` (flutterfire configure에서 자동 생성)
- **이유**: `android/app/build.gradle.kts`의 `applicationId`가 `com.silverworkernow.app`로 되어 있었음 (이전 MustGoOut 프로젝트 템플릿에서 수정된 상태)
- **결정**: 기존 `build.gradle.kts`의 `applicationId`를 따라감. Firebase Console의 기존 `com.silverworker.now` 앱은 orphan 상태로 남음.
- **히스토리**: `google-services.json`에 두 개 클라이언트(`com.silverworker.now`, `com.silverworkernow.app`)가 공존하지만, flutterfire가 생성한 `firebase_options.dart`는 `com.silverworkernow.app`를 참조.

### 3-2. Git 커밋 방식
- 본 작업은 **master에 직접 커밋** (`86242b4`) 후 `role/dev`에 머지함.
- PR 없이 진행된 이유: Firebase Console 연동은 사용자 대화 + 실시간 터미널 작업이 필요한 인프라 세팅으로, PR 리뷰 사이클이 비효율적이었음.
- **대응**: Post-merge review 문서(`General/PR_Review/2026-04-25-role_dev-postmerge-firebase-setup.md`)를 별도 작성하여 리뷰어 검토를 유도.

### 3-3. SHA-1 등록 시기
- **단계 2**에서 SHA-1 입력 화면이 보이지 않음 → flutterfire configure 이후 `com.silverworkernow.app`가 자동 등록되면서 SHA-1이 Console에 반영되었을 것으로 추정.
- **리스크**: 디버그 키스토어(`~/.android/debug.keystore`)의 SHA-1만 등록됨. 릴리스 빌드 시 별도 등록 필요(spec_10 이후).

---

## 4. 생성된 파일 목록

| 파일 | 설명 | 변경 유형 |
|---|---|---|
| `.firebaserc` | Firebase 프로젝트 alias (`default`: `silverworker-d8d64`) | 추가 |
| `firebase.json` | Firebase CLI 설정 (Firestore rules/indexes 경로) | 추가 |
| `firestore.indexes.json` | Firestore 복합 인덱스 정의 (비어 있음) | 추가 |
| `firestore.rules` | 보안 규칙 (spec_01 §5 기준, `firebase deploy` 완료) | 수정 |
| `android/app/google-services.json` | Android Firebase 구성 (2개 클라이언트 포함) | 추가 |
| `android/app/build.gradle.kts` | `applicationId` 및 `google-services` 플러그인 확인 | 수정 |
| `android/settings.gradle.kts` | FlutterFire 플러그인 의존성 | 수정 |
| `lib/firebase_options.dart` | FlutterFire CLI 자동 생성 (5플랫폼) | 대폭 수정 |
| `lib/main.dart` | `DefaultFirebaseOptions.currentPlatform` 연결, try/catch 제거 | 수정 |

---

## 5. 검증 결과

```bash
$ flutter analyze
Analyzing SilverWorkerNow...
No issues found! (ran in 2.2s)
```

**빌드 미검증 사항**: `flutter run`은 실제 Android 에뮬레이터/기기가 필요하여 본 세션에서는 시도하지 않음. spec_02 진입 시 첫 빌드 검증 예정.

---

## 6. 리뷰 및 수정 이력

### 6-1. Claude Post-Merge Review (BLOCKER)
- **B-1 (BLOCKER)**: `firestore.rules`가 Firebase Console "테스트 모드"에 의해 OPEN ACCESS 규칙으로 덮어쓰여짐
  - 원인: Firestore Database 생성 시 "테스트 모드" 선택 → Console이 자동 생성한 규칙이 로컬 spec 규칙을 덮어씀
  - 영향: 2026-05-25까지 인증 없이 누구나 모든 문서 읽기/쓰기 가능 → 보안 사고
  - 수정: `firestore.rules`를 spec_01 §5 / 04_db_schema §3 내용으로 복원 후 `firebase deploy --only firestore:rules` 재배포
  - 커밋: `60a20a7 fix(security): restore firestore.rules to spec_01 §5 / 04_db_schema §3`
- **M-1**: 패키지명 spec drift (`com.silverworker.now` → `com.silverworkernow.app`) → spec_01 §3 갱신 필요
- **M-2**: Firebase Console orphan 클라이언트(`com.silverworker.now`) 삭제 권장
- **M-3**: GCP Console에서 API key application restriction(Android 패키지명 + SHA-1) 설정 권장
- **M-4**: `.gitignore` 정책 명시적 결정 필요
- **N-1~N-4**: macOS bundle ID 오류, firebase.json formatting, firestore.indexes.json 비어있음, flutter run 미검증

### 6-2. Gemini Post-Merge Review
- ✅ ACCEPTED (with minor recommendations)
- API key 제한 설정, orphan 클라이언트 삭제, spec_02 시작 전 빌드 테스트 권장

---

## 7. 다음 단계

1. **spec_02 (인증 및 프로필 등록)**: PR #2로 분리하여 순차 진행
2. **M-1~M-4, N-1~N-4**: spec_02 시작 시점 또는 별도 PR로 처리
3. **Blaze 요금제 업그레이드**: spec_03 직전에 수행
