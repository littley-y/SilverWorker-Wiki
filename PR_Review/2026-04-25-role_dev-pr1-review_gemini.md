# PR Review: PR #1 (Day 1 / spec_01)

**Reviewer**: Gemini
**Date**: 2026-04-25
**Target Spec**: `General/planning/spec_01_project_setup.md`
**Outcome**: ❌ REJECTED (초기 승인 번복됨)

## 1. Spec 준수 여부 및 구조 검증
- **디렉토리 구조**: `lib/providers/`, `lib/repositories/`, `lib/models/` 등 `spec_01`에 명시된 구조가 구현되었습니다.
- **의존성 (pubspec.yaml)**: `firebase_core`, `firebase_auth`, `cloud_firestore`, `cloud_functions`, `flutter_riverpod` 등 필수 라이브러리가 모두 stable 버전으로 추가되었습니다.
- **보안 규칙 (firestore.rules)**: 사용자 문서 및 컬렉션 규칙, 그리고 `/jobs`, `/cached_api_responses` 규칙이 스펙과 일치하게 적용되었습니다.

## 2. Reviewer Focus (PR 요청서 기준)
- **크리티컬 이슈 (Blocker)**: `UserModel`, `JobModel`, `ApplicationModel`이 연관 기획 문서(`overview/04_db_schema.md`)에 명시된 필수 필드(`address`, `physicalConditions`, `preferredJobTypes` 등)를 대거 누락했거나, 타입(`Timestamp` -> `String`)을 잘못 매핑하고 있습니다.
- **영향도**: 현재 모델 구조로 Day 2 (인증/프로필 등록) 및 Day 4 (공고 필터링)를 진행할 경우 필수 데이터 부족으로 즉각적인 구현 불능 및 런타임 에러(Timestamp 파싱 실패)가 발생합니다.

## 3. DoD 및 제한 사항 피드백
- **Firebase Console 미연결 & flutter run 미시도**: Day 1 DoD 미충족 사항이므로, `main.dart`의 초기화 로직을 임시 예외 처리하거나 `flutterfire configure`를 진행하여 `flutter run` 빌드 성공을 반드시 입증해야 합니다.

## 4. 결론
- **Merge**: ❌ REJECTED (반려)
- **반려 사유**: 초기 리뷰에서는 `spec_01`의 단일 파일 구조 및 `flutter analyze`에만 집중하여 승인했으나, 다른 리뷰어(Claude)의 정밀한 교차 검증(`04_db_schema.md` 대조) 결과를 검토한 결과 중대한 스펙 누락이 있음을 확인하여 승인을 번복합니다.
- **조치 사항**: 모델(`models/*.dart`) 구조를 DB 스키마 명세에 맞게 전면 수정하고, Firestore 연동 시 발생하는 타입 오류를 방지한 후 PR을 갱신해주세요.
