# Spec 09. 시니어 특화 UI 시스템

> 대상 Day: Day 4 (기반), Day 11 (마무리)
> 참조: `overview/03_mvp_specs.md` UI-01~05

---

## 1. 타이포그래피

| 이름 | 크기 | 굵기 | 용도 |
|---|---|---|---|
| `headline` | 24pt | Bold | 화면 주 타이틀 |
| `title` | 20pt | Bold | 카드 제목, 섹션 헤더 |
| `body` | 18pt | Regular | 본문, 입력 필드 |
| `caption` | 14pt | Regular | 보조 텍스트, 날짜 |
| `button` | 20pt | Bold | 주요 버튼 텍스트 |

> 최소 폰트: **14pt**. 이 이하 크기는 사용 금지.

```dart
// app_text_styles.dart
class AppTextStyles {
  static const headline = TextStyle(fontSize: 24, fontWeight: FontWeight.bold, color: AppColors.textPrimary);
  static const title = TextStyle(fontSize: 20, fontWeight: FontWeight.bold, color: AppColors.textPrimary);
  static const body = TextStyle(fontSize: 18, color: AppColors.textPrimary);
  static const caption = TextStyle(fontSize: 14, color: AppColors.textSecondary);
  static const button = TextStyle(fontSize: 20, fontWeight: FontWeight.bold, color: Colors.white);
  static const sectionTitle = TextStyle(fontSize: 16, fontWeight: FontWeight.w600, color: AppColors.textSecondary);
}
```

---

## 2. 컬러 시스템

```dart
// app_colors.dart
class AppColors {
  // Primary — WCAG AA 대비율 충족 (배경 흰색 기준 4.5:1 이상)
  static const primary = Color(0xFF1565C0);       // 진한 파랑
  static const primaryLight = Color(0xFFE3F2FD);  // 연한 파랑 (배지 배경)

  // Text
  static const textPrimary = Color(0xFF212121);   // 대비율 16:1
  static const textSecondary = Color(0xFF757575); // 대비율 4.6:1

  // Background
  static const background = Color(0xFFF5F5F5);
  static const cardBackground = Colors.white;
  static const divider = Color(0xFFE0E0E0);

  // Status (세이프티 배지)
  static const intensityLight = Color(0xFF4CAF50);
  static const intensityModerate = Color(0xFFFF9800);
  static const intensityHeavy = Color(0xFFF44336);

  // Application Status (지원 상태 배지)
  static const statusSubmitted = Color(0xFF1976D2);
  static const statusReviewing = Color(0xFFFF9800);
  static const statusAccepted = Color(0xFF4CAF50);
  static const statusRejected = Color(0xFFF44336);
  static const statusCancelled = Color(0xFF9E9E9E);
}
```

---

## 3. 터치 타겟

| 컴포넌트 | 최소 크기 |
|---|---|
| 주요 CTA 버튼 | 56dp 높이, 전체 너비 |
| 보조 버튼 | 48dp 높이 |
| 아이콘 버튼 | 48dp × 48dp |
| 필터 칩 | 40dp 높이, 수평 패딩 16dp |
| 리스트 아이템 | 64dp 이상 높이 |
| Bottom Nav 탭 | 56dp 높이 |

```dart
// 주요 CTA 버튼 공통 위젯
class PrimaryButton extends StatelessWidget {
  final String label;
  final VoidCallback? onPressed;
  final bool isLoading;

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: double.infinity,
      height: 56,
      child: ElevatedButton(
        onPressed: isLoading ? null : onPressed,
        style: ElevatedButton.styleFrom(
          backgroundColor: AppColors.primary,
          disabledBackgroundColor: Colors.grey,
          shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
        ),
        child: isLoading
            ? const SizedBox(width: 24, height: 24, child: CircularProgressIndicator(color: Colors.white, strokeWidth: 2))
            : Text(label, style: AppTextStyles.button),
      ),
    );
  }
}
```

---

## 4. 햅틱 피드백 (UI-05)

버튼 탭 시 진동 + 시각적 하이라이트 (`InkWell` 의 splash 활용):

```dart
// 주요 버튼 탭 시
HapticFeedback.lightImpact();

// 지원 완료 등 중요 이벤트
HapticFeedback.mediumImpact();
```

---

## 5. 로딩 / 에러 / 빈 상태 공통 위젯

```dart
// 로딩
class LoadingView extends StatelessWidget {
  @override
  Widget build(BuildContext context) =>
      const Center(child: CircularProgressIndicator(color: AppColors.primary));
}

// 에러
class ErrorView extends StatelessWidget {
  final String message;
  final VoidCallback onRetry;
  // 중앙 아이콘 + 메시지 + "다시 시도" 버튼
}

// 빈 상태
class EmptyView extends StatelessWidget {
  final String message;
  final String? actionLabel;
  final VoidCallback? onAction;
  // 중앙 아이콘 + 메시지 + 선택적 액션 버튼
}
```

---

## 6. 완료 기준 (Day 11 DoD)

- [ ] 전체 화면에서 14pt 미만 텍스트 없음 확인
- [ ] 주요 버튼 높이 56dp 이상 확인
- [ ] 버튼 탭 시 햅틱 진동 확인 (실제 기기)
- [ ] 네트워크 끊은 상태에서 에러 화면 표시 확인
- [ ] 공고 없는 필터 조건 적용 시 빈 상태 화면 표시 확인
