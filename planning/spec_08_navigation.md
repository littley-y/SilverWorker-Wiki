# Spec 08. 네비게이션 및 화면 연결

> 대상 Day: Day 10
> 참조: `overview/03_mvp_specs.md` 5절 화면 흐름

---

## 1. 라우팅 구조

패키지: `go_router` (Flutter 공식 권장)

```dart
final routerProvider = Provider<GoRouter>((ref) {
  final authState = ref.watch(authStateProvider);

  return GoRouter(
    initialLocation: '/home',
    redirect: (context, state) {
      final isLoggedIn = authState.value != null;
      final isAuthRoute = state.matchedLocation.startsWith('/auth');

      if (!isLoggedIn && !isAuthRoute) return '/auth/phone';
      if (isLoggedIn && isAuthRoute) return '/home';
      return null;
    },
    routes: [
      GoRoute(path: '/auth/phone', builder: (_, __) => PhoneInputScreen()),
      GoRoute(path: '/auth/otp', builder: (_, state) => OtpInputScreen(verificationId: state.extra as String)),
      GoRoute(path: '/auth/profile', builder: (_, __) => ProfileSetupScreen()),
      ShellRoute(
        builder: (_, __, child) => MainShell(child: child),
        routes: [
          GoRoute(path: '/home', builder: (_, __) => HomeScreen()),
          GoRoute(path: '/applications', builder: (_, __) => ApplicationListScreen()),
          GoRoute(path: '/mypage', builder: (_, __) => MypageScreen()),
        ],
      ),
      GoRoute(path: '/job/:jobId', builder: (_, state) => JobDetailScreen(jobId: state.pathParameters['jobId']!)),
      GoRoute(path: '/apply/:jobId', builder: (_, state) => ApplicationFormScreen(jobId: state.pathParameters['jobId']!)),
      GoRoute(path: '/apply/:jobId/done', builder: (_, state) => ApplicationResultScreen(jobId: state.pathParameters['jobId']!)),
    ],
  );
});
```

---

## 2. Bottom Navigation (MainShell)

### 탭 구성

| 인덱스 | 아이콘 | 라벨 | 라우트 |
|---|---|---|---|
| 0 | 🏠 home | 홈 | `/home` |
| 1 | 📋 list_alt | 지원현황 | `/applications` |
| 2 | 👤 person | 마이페이지 | `/mypage` |

### UI 규칙
- 탭 아이콘 크기: 28dp
- 탭 레이블 폰트: 14pt
- 선택 탭 색상: 메인 컬러 (primary)
- 미선택 탭 색상: 회색

```dart
class MainShell extends StatelessWidget {
  final Widget child;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: child,
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _getCurrentIndex(context),
        onTap: (index) => _onTap(context, index),
        items: const [...],
        selectedFontSize: 14,
        unselectedFontSize: 14,
        iconSize: 28,
      ),
    );
  }
}
```

---

## 3. 화면 간 데이터 전달

| 출발 화면 | 도착 화면 | 전달 데이터 | 전달 방법 |
|---|---|---|---|
| HomeScreen (카드 탭) | JobDetailScreen | jobId | path parameter |
| JobDetailScreen (지원 버튼) | ApplicationFormScreen | jobId | path parameter |
| ApplicationFormScreen (완료) | ApplicationResultScreen | jobId | path parameter |
| OtpInputScreen | - | verificationId | `extra` |

---

## 4. 뒤로가기 처리

| 화면 | 뒤로가기 동작 |
|---|---|
| JobDetailScreen | 홈으로 pop |
| ApplicationFormScreen | JobDetailScreen으로 pop |
| ApplicationResultScreen | 홈으로 이동 (back stack 초기화, `go()` 사용) |
| 인증 화면 | Android 뒤로가기 → 앱 종료 (`PopScope` 처리) |

---

## 5. 로딩 인디케이터

전역 로딩 오버레이는 사용하지 않음. 각 화면에서 개별 처리:

```dart
// 공통 패턴
if (asyncValue.isLoading) {
  return const Center(child: CircularProgressIndicator());
}
```

---

## 6. 완료 기준 (Day 10 DoD)

- [ ] Bottom Nav 3개 탭 전환 동작 확인
- [ ] 홈 → 상세 → 뒤로가기 → 홈 복귀 확인
- [ ] 지원 완료 → 홈으로 이동 (뒤로가기로 지원 화면 재진입 불가) 확인
- [ ] 미로그인 상태에서 앱 진입 시 자동으로 로그인 화면으로 이동 확인
