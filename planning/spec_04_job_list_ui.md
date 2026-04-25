# Spec 04. 공고 목록 UI

> 대상 Day: Day 4
> 참조: `overview/03_mvp_specs.md` JOB-01~03, UI-01~05

---

## 1. 화면 구성

```
HomeScreen
├── AppBar: "은빛일자리" (로고 + 타이틀)
├── FilterBar: 지역 칩 + 직종 칩
├── ListView: JobCard × N
└── (빈 결과) EmptyStateWidget
```

---

## 2. JobCard 위젯

### 표시 정보

| 항목 | 위치 | 스타일 |
|---|---|---|
| 공고 제목 | 상단, Bold | 20pt, 기본 텍스트 색 |
| 회사명 | 제목 아래 | 16pt, 회색 |
| 급여 | 우측 상단 | 18pt Bold, 메인 컬러 |
| 근무 형태 칩 | 하단 좌측 | 12pt, 배경 연회색 |
| 마감일 | 하단 우측 | 14pt, 회색 (D-n 형식) |
| physicalIntensity 배지 | 우측 하단 | 아이콘 + 텍스트 (spec_05 참조) |

### 레이아웃 규칙
- 카드 패딩: 16dp 사방
- 카드 간 간격: 8dp
- 카드 모서리 반경: 12dp
- 카드 그림자: elevation 2
- 터치 영역: 카드 전체 (최소 88dp 높이 보장)

### 코드 구조

```dart
class JobCard extends StatelessWidget {
  final JobModel job;

  // 탭 → JobDetailScreen으로 이동
  // 카드 전체가 InkWell
}
```

---

## 3. 필터바 (FilterBar)

### UI
- 가로 스크롤 가능한 칩 Row
- 지역 칩: 선택 시 파란 배경, 미선택 시 흰 배경 + 테두리
- 직종 칩: 동일 스타일
- 칩 높이: 40dp, 수평 패딩: 16dp

### 지역 옵션 (데모용)

| 표시명 | locationCode |
|---|---|
| 전체 | null |
| 종로구 | 11110 |
| 중구 | 11140 |
| 용산구 | 11170 |

### 직종 옵션

| 표시명 | jobCategory 값 |
|---|---|
| 전체 | null |
| 경비/관리 | security_management |
| 청소/미화 | cleaning |
| 단순노무 | simple_labor |
| 서비스 | service |
| 사무/문서 | office_work |

### 선택 동작
- **단일 선택** (2주 데모 범위)
- 칩 선택 시 `jobFilterProvider` 업데이트 → `jobListProvider` 자동 재조회
- 이미 선택된 칩 재탭 → 선택 해제 (전체로 초기화)

---

## 4. 시니어 UI 적용

```dart
// app_text_styles.dart
static const TextStyle title = TextStyle(fontSize: 20, fontWeight: FontWeight.bold);
static const TextStyle body = TextStyle(fontSize: 18);
static const TextStyle caption = TextStyle(fontSize: 14, color: Colors.grey);

// app_colors.dart
static const Color primary = Color(0xFF1B6CA8);      // 메인 파란색 (고대비)
static const Color background = Color(0xFFF5F5F5);
static const Color cardBackground = Colors.white;
static const Color textPrimary = Color(0xFF212121);  // WCAG AA 대비율 충족
static const Color textSecondary = Color(0xFF757575);
```

---

## 5. 로딩 / 에러 / 빈 상태

| 상태 | 표시 |
|---|---|
| 로딩 중 | 스켈레톤 카드 3개 (shimmer 효과 없이 단순 회색 블록) |
| 네트워크 오류 | 중앙에 아이콘 + "공고를 불러올 수 없습니다" + "다시 시도" 버튼 |
| 결과 없음 | 중앙에 아이콘 + "해당 조건의 공고가 없습니다" + "필터 초기화" 버튼 |

---

## 6. 완료 기준 (Day 4 DoD)

- [ ] 공고 목록이 카드 형태로 표시됨
- [ ] 지역/직종 필터 칩 탭 시 목록 변경 확인
- [ ] 18pt 이상 폰트 전체 적용 확인
- [ ] 에뮬레이터에서 스크롤 동작 확인
