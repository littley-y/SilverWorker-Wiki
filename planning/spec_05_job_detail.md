# Spec 05. 공고 상세 및 세이프티 큐레이션

> 대상 Day: Day 5
> 참조: `overview/03_mvp_specs.md` JOB-05, SAFE-01~02

---

## 1. 화면 구성

```
JobDetailScreen
├── AppBar: 뒤로가기 버튼 + "공고 상세"
├── ScrollView
│   ├── 헤더 영역: 공고 제목, 회사명, 급여
│   ├── 세이프티 섹션: physicalIntensity 등급 + physicalBadges
│   ├── 근무 조건 섹션: 근무지, 시간, 요일, 기간, 형태
│   ├── 자격 요건 섹션
│   ├── 복리후생 섹션
│   └── 업무 내용 섹션
└── BottomBar: "지원하기" 버튼 (고정)
```

---

## 2. 세이프티 섹션 (핵심 차별화 기능)

### physicalIntensity 등급 표시

| 값 | 표시명 | 색상 | 아이콘 |
|---|---|---|---|
| `light` | 가벼움 | 초록 (#4CAF50) | 😊 또는 잎사귀 아이콘 |
| `moderate` | 보통 | 주황 (#FF9800) | 😐 또는 사람 걷기 아이콘 |
| `heavy` | 무거움 | 빨강 (#F44336) | 😓 또는 덤벨 아이콘 |

등급 박스: 배경 = 해당 색상 10% 투명도, 텍스트 + 아이콘 = 해당 색상. 높이 48dp.

### physicalBadges 배지 목록

| 값 | 표시명 | 아이콘 |
|---|---|---|
| `standing` | 계속 서있기 | 🧍 |
| `sitting` | 좌식 업무 | 🪑 |
| `heavy_lifting` | 무거운 짐 | 📦 |
| `outdoor` | 야외 근무 | ☀️ |
| `repetitive` | 반복 동작 | 🔄 |
| `stairs` | 계단 오르내림 | 🪜 |

배지 레이아웃: `Wrap` 위젯으로 줄바꿈 처리. 각 배지 = 아이콘 + 텍스트 칩 (배경 연회색, 높이 36dp).

### 코드 구조

```dart
class SafetyCurationSection extends StatelessWidget {
  final String physicalIntensity;
  final List<String> physicalBadges;

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text('업무 강도', style: AppTextStyles.sectionTitle),
        IntensityBadge(intensity: physicalIntensity),
        const SizedBox(height: 12),
        Text('신체 부담 항목', style: AppTextStyles.sectionTitle),
        Wrap(
          spacing: 8,
          runSpacing: 8,
          children: physicalBadges.map((b) => PhysicalBadgeChip(badge: b)).toList(),
        ),
      ],
    );
  }
}
```

---

## 3. 근무 조건 섹션

표 형태로 표시:

| 항목 | 값 |
|---|---|
| 근무지 | companyAddress |
| 급여 | salaryType + salaryAmount 조합 표시 (예: "월 200만원") |
| 근무 시간 | workHours |
| 근무 요일 | workDays |
| 근무 기간 | workPeriod |
| 고용 형태 | employmentType 한글 변환 |

급여 조합 규칙:
- `hourly` → "시급 N원"
- `daily` → "일급 N원"
- `monthly` → "월 N만원"

---

## 4. 하단 고정 "지원하기" 버튼

- 높이: 56dp
- 배경: 메인 컬러 (primary)
- 텍스트: "지원하기" 20pt Bold 흰색
- 탭 → ApplicationFormScreen으로 이동 (spec_06)
- `SafeArea` 로 감싸서 하단 노치/홈바 영역 침범 방지

---

## 5. 완료 기준 (Day 5 DoD)

- [ ] 공고 목록에서 카드 탭 → 상세 화면 진입 확인
- [ ] physicalIntensity 등급이 색상 + 텍스트로 표시됨
- [ ] physicalBadges 배지가 1개 이상 표시됨
- [ ] 하단 "지원하기" 버튼이 스크롤해도 고정되어 있음
- [ ] 뒤로가기 버튼으로 목록 복귀 확인
