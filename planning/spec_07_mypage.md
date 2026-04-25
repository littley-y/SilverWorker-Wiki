# Spec 07. 마이페이지

> 대상 Day: Day 9
> 참조: `overview/03_mvp_specs.md` MY-01a, APP-02

---

## 1. 화면 목록

| 화면 | 라우트 | 설명 |
|---|---|---|
| MypageScreen | `/mypage` | 프로필 요약 + 지원 내역 진입점 |
| ApplicationListScreen | `/mypage/applications` | 전체 지원 내역 목록 |

---

## 2. MypageScreen

### UI 구성

```
MypageScreen
├── 프로필 요약 카드
│   ├── 이름 (22pt Bold)
│   ├── 거주 지역 (18pt)
│   ├── 경력 소개 (16pt, 최대 2줄 + 말줄임)
│   └── 지원 횟수 뱃지 (예: "총 3건 지원")
├── 메뉴 리스트
│   ├── "지원 내역" → ApplicationListScreen
│   └── (추후: 찜한 공고, 알림 설정)
└── 로그아웃 버튼 (하단, 텍스트 버튼 스타일, 빨간색)
```

### 프로필 데이터 로드

```dart
// userProfileProvider 사용 (spec_01에서 정의)
final profile = ref.watch(userProfileProvider(uid));
```

### 지원 횟수 표시

```dart
// myApplicationsProvider 사용 (spec_01에서 정의)
final applications = ref.watch(myApplicationsProvider(uid));
final count = applications.value?.length ?? 0;
// 표시: "총 $count건 지원"
```

### 로그아웃 처리

```dart
Future<void> _logout() async {
  await FirebaseAuth.instance.signOut();
  // authStateProvider가 null 반환 → 자동으로 PhoneInputScreen으로 이동
}
```

로그아웃 버튼 탭 시 확인 다이얼로그 표시:
- 메시지: "로그아웃 하시겠습니까?"
- 버튼: "취소" / "로그아웃"

---

## 3. ApplicationListScreen

### UI 구성

```
ApplicationListScreen
├── AppBar: "지원 내역"
└── ListView: ApplicationCard × N
    또는 EmptyState: "아직 지원한 공고가 없습니다"
```

### ApplicationCard

| 항목 | 스타일 |
|---|---|
| 공고 제목 | 18pt Bold |
| 회사명 | 16pt 회색 |
| 지원 일시 | 14pt 회색 (MM월 DD일 형식) |
| 상태 배지 | 우측, 색상으로 구분 |

### 상태 배지 색상

| status 값 | 표시명 | 배지 색상 |
|---|---|---|
| `submitted` | 접수 | 파랑 |
| `reviewing` | 검토 중 | 주황 |
| `accepted` | 합격 | 초록 |
| `rejected` | 불합격 | 빨강 |
| `cancelled` | 취소됨 | 회색 |

### 정렬
`submittedAt` 내림차순 (최신 지원 순)

---

## 4. 완료 기준 (Day 9 DoD)

- [ ] 마이페이지에서 프로필 이름, 지역 표시 확인
- [ ] 지원 내역 탭 → 지원한 공고 목록 표시 확인
- [ ] 각 지원 항목에 상태 배지 표시 확인
- [ ] 로그아웃 → 다이얼로그 → 확인 시 로그인 화면으로 이동 확인
