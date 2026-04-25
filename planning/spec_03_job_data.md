# Spec 03. 공고 데이터 준비 (Mock + Cloud Functions)

> 대상 Day: Day 3
> 참조: `overview/02_tech_stack.md` 4절, `overview/04_db_schema.md` jobs 컬렉션

---

## 1. 분기 전략

Day 3 시작 시점에 고용24 API 키 발급 여부에 따라 분기합니다.

```
Day 3 시작
  ├─ API 키 미발급 → [경로 A] Mock 데이터 30개 Firestore 수동 등록
  └─ API 키 발급 완료 → [경로 B] Cloud Functions 프록시 구현
```

두 경로 모두 Flutter 앱에서 동일한 `/jobs` 컬렉션을 읽도록 설계하여 이후 코드 변경 없음.

---

## 2. 경로 A — Mock 데이터

### Mock 데이터 필수 필드 (DB 스키마 전체 필드 준수)

각 문서는 아래 필드를 모두 포함해야 합니다 (Day 5 배지 구현에 필요):

```json
{
  "jobId": "MOCK_001",
  "source": "mock",
  "title": "OO아파트 경비원 모집",
  "companyName": "OO아파트 관리사무소",
  "companyAddress": "서울 종로구 OO로 123",
  "locationCode": "11110",
  "jobCategory": "security_management",
  "jobCategoryDetail": "apt_security",
  "employmentType": "part_time",
  "salaryType": "monthly",
  "salaryAmount": 2000000,
  "workHours": "08:00 ~ 17:00 (휴게 1시간)",
  "workDays": "월~금",
  "workPeriod": "6개월",
  "requirements": "경비원 신임교육 이수자",
  "benefits": "중식 제공",
  "description": "공동주택 출입 및 순찰 관리 업무",
  "physicalIntensity": "moderate",
  "physicalBadges": ["standing", "outdoor"],
  "minAge": 60,
  "maxAge": 75,
  "deadline": "2026-06-30T00:00:00Z",
  "isActive": true,
  "rawData": {},
  "createdAt": "serverTimestamp",
  "updatedAt": "serverTimestamp"
}
```

### Mock 데이터 구성 (30개)

| locationCode | jobCategory | physicalIntensity | 수량 |
|---|---|---|---|
| 11110 (종로구) | security_management | moderate | 5 |
| 11110 (종로구) | cleaning | light | 4 |
| 11110 (종로구) | simple_labor | heavy | 3 |
| 11140 (중구) | security_management | light | 4 |
| 11140 (중구) | service | light | 4 |
| 11170 (용산구) | security_management | moderate | 4 |
| 11170 (용산구) | office_work | light | 3 |
| 기타 | 혼합 | 혼합 | 3 |

`physicalBadges` 는 각 공고에 1~3개 조합. `standing`, `sitting`, `heavy_lifting`, `outdoor`, `repetitive`, `stairs` 중 선택.

---

## 3. 경로 B — Cloud Functions 프록시

### 디렉토리 구조

```
functions/
├── package.json        # Node.js 20
├── index.js
└── src/
    ├── fetchGoyong24Jobs.js
    └── utils/
        └── xmlToJson.js
```

### fetchGoyong24Jobs 함수

```javascript
// HTTP onCall 함수
exports.fetchGoyong24Jobs = onCall(async (request) => {
  const { locationCode, jobCategory, page = 1 } = request.data;

  const url = `https://www.work24.go.kr/cm/openApi/call/wk/apiWkOccuySrchList.do`;
  const params = new URLSearchParams({
    authKey: process.env.GOYONG24_API_KEY,  // Firebase 환경변수로 관리
    returnType: 'XML',
    startPage: page,
    display: 50,
    region1: locationCode,
    occupation: jobCategory,
  });

  const response = await fetch(`${url}?${params}`);
  const xml = await response.text();
  const json = await parseXmlToJson(xml);  // xmlToJson.js 유틸

  // 시니어 부적합 공고 필터링 (고난도 기술, 외국어 필수)
  const filtered = json.jobs.filter(job => !isSeniorInappropriate(job));

  return { jobs: filtered };
});
```

### API 키 환경변수 설정

```bash
firebase functions:secrets:set GOYONG24_API_KEY
# 입력: 발급받은 인증키
```

### 배포 명령

```bash
cd functions
npm install
firebase deploy --only functions:fetchGoyong24Jobs
```

---

## 4. Flutter에서 데이터 수신 (양쪽 공통)

### job_repository.dart

```dart
Future<List<JobModel>> fetchJobs(JobFilter filter) async {
  Query query = FirebaseFirestore.instance
      .collection('jobs')
      .where('isActive', isEqualTo: true)
      .where('deadline', isGreaterThan: Timestamp.now());

  if (filter.locationCode != null) {
    query = query.where('locationCode', isEqualTo: filter.locationCode);
  }
  if (filter.jobCategory != null) {
    query = query.where('jobCategory', isEqualTo: filter.jobCategory);
  }

  query = query.orderBy('deadline').limit(50);

  final snapshot = await query.get();
  return snapshot.docs.map((doc) => JobModel.fromFirestore(doc)).toList();
}
```

---

## 5. 완료 기준 (Day 3 DoD)

- [ ] Firestore Console에서 `/jobs` 컬렉션에 30개 문서 확인
- [ ] 각 문서에 `physicalBadges`, `physicalIntensity`, `locationCode` 필드 존재 확인
- [ ] Flutter에서 `job_repository.fetchJobs()` 호출 시 데이터 수신 로그 확인
- [ ] (경로 B 해당 시) Cloud Functions 배포 후 Firebase Console에서 함수 정상 동작 확인
