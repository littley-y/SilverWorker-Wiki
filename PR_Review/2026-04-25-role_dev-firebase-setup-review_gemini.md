# Post-Merge Review: Firebase Console 연동 세션

**Reviewer**: Gemini
**Date**: 2026-04-25
**Target**: `86242b4` 커밋 (Firebase 설정)
**Outcome**: ✅ ACCEPTED (with minor recommendations for spec_02)

## 1. 전반적인 평가
PR 없이 master 브랜치에 직접 커밋된 사항이지만, 인프라 세팅이라는 특수성과 사용자 대화형 작업의 한계를 고려할 때 충분히 합리적인 결정이었습니다. `main.dart`의 placeholder `try/catch`가 제거되고, `DefaultFirebaseOptions.currentPlatform`이 정상적으로 연결된 것을 확인했습니다. `flutter analyze` 0경고도 잘 유지되었습니다.

## 2. 리뷰 포인트 피드백

### 2.1 보안 (Security)
- **API Key 노출**: `google-services.json` 및 `firebase_options.dart`에 포함된 API 키는 클라이언트 측 키이므로 소스 코드에 포함되는 것 자체가 치명적인 보안 결함은 아닙니다. 그러나 **Google Cloud Console에서 해당 API 키들에 대해 Android 패키지명(`com.silverworkernow.app`) 및 SHA-1 제한을 설정**하는 것을 강력히 권장합니다.
- **`.gitignore` 정책**: 만약 이 프로젝트가 Public Repository로 전환될 계획이 있다면, API 키 스크래핑을 방지하기 위해 두 설정 파일(`.json`, `.dart`)을 `.gitignore`에 추가하는 것이 좋습니다. 당장의 MVP나 Private 레포지토리에서는 생산성을 위해 커밋 상태를 유지해도 무방합니다.

### 2.2 설정 정확성 (Configuration)
- **패키지명 변경 결단**: `build.gradle.kts`에 명시되어 있던 `com.silverworkernow.app`로 패키지명을 통일한 것은 훌륭한 판단이었습니다. 기존 기획(`com.silverworker.now`)을 강행했다면 런타임 크래시가 발생했을 것입니다. 
- **Orphan 클라이언트**: `google-services.json`에 `com.silverworker.now` 클라이언트가 함께 포함되어 있습니다. 실행에 문제는 없으나, 추후 관리를 위해 Firebase Console에서 해당 안드로이드 앱을 삭제한 뒤 `google-services.json`을 다시 받아 교체하는 것을 추천합니다.
- **다중 플랫폼 옵션**: MVP에서 Android 위주로 개발되더라도, FlutterFire CLI가 전체 플랫폼을 생성해 둔 것은 추후 확장을 고려할 때 나쁘지 않은 선택입니다.

### 2.3 DoD 및 기타 정책
- **빌드 검증 지연**: `flutter run` 검증을 `spec_02`로 연기한 것은 환경적 한계(에뮬레이터 부재)를 고려할 때 타당합니다. 단, `spec_02` 작업 시작 시 **가장 첫 번째로 빌드 테스트를 수행**하여 크래시 여부를 점검해야 합니다.
- **Blaze 요금제 지연**: `spec_03`에서 Cloud Functions를 사용할 때 업그레이드하기로 한 결정은 비용 효율성 측면에서 적절합니다.

## 3. 결론 및 다음 단계
본 Firebase 연동 작업은 성공적으로 수행되었으며, `spec_02` 진입을 위한 기반이 완벽하게 마련되었습니다. 

**권장 Action Items (spec_02 시작 시 반영 권장):**
1. Firebase Console에서 안 쓰는 클라이언트(`com.silverworker.now`) 앱 삭제 후 json 재다운로드
2. Google Cloud Console에서 API Key 제한(Restriction) 활성화
3. `spec_02` 코딩 시작 전 실제 에뮬레이터에서 앱 구동(`flutter run`) 테스트 진행

머지된 작업을 승인하며, `spec_02` 작업을 시작하셔도 좋습니다!
