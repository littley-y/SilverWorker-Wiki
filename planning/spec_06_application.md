# Spec 06. 지원 기능

> 대상 Day: Day 8
> 참조: `overview/03_mvp_specs.md` APP-01~02, APP-04

---

## 1. 화면 목록

| 화면 | 라우트 | 설명 |
|---|---|---|
| ApplicationFormScreen | `/apply/:jobId` | 자기소개 입력 + 지원 제출 |
| ApplicationResultScreen | `/apply/:jobId/done` | 지원 완료 확인 화면 |

---

## 2. ApplicationFormScreen

### UI 요소
- AppBar: "지원서 작성"
- 공고 요약 카드 (제목, 회사명) — 수정 불가, 상단 고정
- 자기소개 텍스트 에어리어
  - 플레이스홀더: "간단한 자기소개를 작성해 주세요 (선택)"
  - 최대 200자, 글자 수 카운터 (우측 하단: "0 / 200")
  - 폰트 18pt, 최소 높이 120dp
- "지원하기" 버튼 (56dp, 하단 고정)

### 오터치 방어 (APP-04)

```dart
class _ApplicationFormScreenState extends State<ApplicationFormScreen> {
  bool _isSubmitting = false;

  Future<void> _submit() async {
    if (_isSubmitting) return;  // 중복 클릭 차단
    setState(() => _isSubmitting = true);

    try {
      await ref.read(applicationRepositoryProvider).submitApplication(
        jobId: widget.jobId,
        selfIntroduction: _controller.text,
      );
      context.go('/apply/${widget.jobId}/done');
    } catch (e) {
      // 에러 스낵바
    } finally {
      await Future.delayed(const Duration(milliseconds: 1500));  // 1.5초 쿨다운
      if (mounted) setState(() => _isSubmitting = false);
    }
  }
}
```

버튼 상태: `_isSubmitting == true` 일 때 → 배경 회색, 텍스트 "지원 중...", 로딩 스피너 표시.

---

## 3. Firestore 저장

### 저장 경로
`/users/{uid}/applications/{autoId}`

```dart
Future<void> submitApplication({
  required String jobId,
  required String selfIntroduction,
}) async {
  final uid = FirebaseAuth.instance.currentUser!.uid;

  // 중복 지원 방지: 동일 jobId 기존 문서 존재 여부 확인
  final existing = await FirebaseFirestore.instance
      .collection('users')
      .doc(uid)
      .collection('applications')
      .where('jobId', isEqualTo: jobId)
      .limit(1)
      .get();

  if (existing.docs.isNotEmpty) {
    throw Exception('already_applied');
  }

  // 공고 denormalized 필드 조회
  final jobDoc = await FirebaseFirestore.instance.collection('jobs').doc(jobId).get();

  await FirebaseFirestore.instance
      .collection('users')
      .doc(uid)
      .collection('applications')
      .add({
    'jobId': jobId,
    'jobTitle': jobDoc['title'],
    'companyName': jobDoc['companyName'],
    'selfIntroduction': selfIntroduction,
    'status': 'submitted',
    'submittedAt': FieldValue.serverTimestamp(),
    'updatedAt': FieldValue.serverTimestamp(),
  });
}
```

---

## 4. ApplicationResultScreen

### UI 요소
- 중앙에 체크 아이콘 (초록, 72dp)
- "지원이 완료되었습니다!" (22pt Bold)
- 공고명 + 회사명 (18pt)
- "마이페이지에서 지원 현황을 확인하세요" (16pt 회색)
- "확인" 버튼 → HomeScreen으로 이동 (back stack 초기화)

---

## 5. 에러 처리

| 상황 | 처리 |
|---|---|
| 이미 지원한 공고 | 스낵바: "이미 지원한 공고입니다" (버튼 비활성화) |
| 네트워크 오류 | 스낵바: "지원에 실패했습니다. 다시 시도해 주세요" + 재시도 가능 |
| 공고 마감 | 스낵바: "마감된 공고입니다" |

---

## 6. 완료 기준 (Day 8 DoD)

- [ ] 지원 버튼 연속 탭 시 1회만 Firestore에 저장됨 (Console 확인)
- [ ] 버튼 클릭 시 1.5초간 회색 + 스피너 표시 확인
- [ ] `/users/{uid}/applications/` 에 status: "submitted" 문서 생성 확인
- [ ] 같은 공고 재지원 시 "이미 지원한 공고입니다" 표시 확인
