# 04. SilverWorkerNow DB 스키마 설계

> 기준: Firebase Cloud Firestore (NoSQL)
> 작성 일자: 2026-04-24
> AI 기능 제외 MVP 기준

---

## 1. 개요

Cloud Firestore는 문서(Document) 기반 NoSQL 데이터베이스입니다. 컬렉션(Collection) 안에 문서가 있고, 문서 안에 하위 컬렉션(Subcollection)을 중첩할 수 있습니다.

MVP 단계에서는 관계형 DB의 복잡한 Join 없이도 충분히 표현 가능하며, 향후 AI 기능 추가 시에도 유연하게 확장할 수 있습니다.

### ⚠️ Firestore 배열 쿼리 제한
- Firestore는 쿼리당 **단 하나의 `array-contains` 조걸만** 허용 (최대 10개 요소)
- `physicalBadges` 등의 다중 배열 필터링은 **DB 레벨에서 불가능**
- 해결책: `locationCode`, `jobCategory` 등 큰 범주로 DB 쿼리 → 세부 조건(`physicalBadges`)은 **클리언트(Flutter) 단에서 필터링**

---

## 2. 컬렉션 구조

### 2.1 users (구직자)

```
/users/{userId}
```

| 필드 | 타입 | 설명 | 예시 |
|---|---|---|---|
| `userId` | string (문서ID) | Firebase Auth UID | `abc123def456` |
| `phoneNumber` | string | 휴대폰 번호 (인증용) | `+821012345678` |
| `name` | string | 이름 | `김OO` |
| `birthDate` | timestamp | 생년월일 | 1958-03-15 |
| `gender` | string | 성별 | `male` / `female` |
| `address` | map | 거주 지역 | `{ sido: "서울특별시", sigungu: "종로구" }` |
| `careerSummary` | string | 간단한 경력 소개 (500자 내외) | "30년 경비 업무 경험..." |
| `physicalConditions` | array&lt;string&gt; | 신체 조건 (필터링용). **Firestore는 array-contains를 1개만 지원하므로 클라이언트에서 2차 필터링** | `["knee_pain", "cannot_stand_long"]` |
| `preferredJobTypes` | array&lt;string&gt; | 선호 근무 형태 | `["part_time", "daily"]` |
| `preferredLocations` | array&lt;string&gt; | 선호 지역 코드 목록 | `["11110", "11140"]` |
| `isPushEnabled` | boolean | 푸시 알림 수신 여부 | `true` |
| `createdAt` | timestamp | 가입 일시 | 2026-04-24T10:00:00Z |
| `updatedAt` | timestamp | 최종 수정 일시 | 2026-04-24T10:00:00Z |

---

### 2.2 jobs (공고)

```
/jobs/{jobId}
```

| 필드 | 타입 | 설명 | 예시 |
|---|---|---|---|
| `jobId` | string (문서ID) | 고용24 공고번호 또는 자체 UUID | `KJJ_12345678` |
| `source` | string | 데이터 출처 | `goyong24` |
| `title` | string | 공고 제목 | `아파트 경비원 모집` |
| `companyName` | string | 기업명 | `OO아파트 관리사무소` |
| `companyAddress` | string | 근무지 주소 | `서울 종로구 OO로 123` |
| `locationCode` | string | 지역 코드 (행정동 코드) | `11110` |
| `jobCategory` | string | 직종 대분류 | `security_management` |
| `jobCategoryDetail` | string | 직종 소분류 | `apt_security` |
| `employmentType` | string | 근무 형태 | `part_time` / `daily` / `short_term` / `full_time` |
| `salaryType` | string | 급여 형태 | `hourly` / `daily` / `monthly` |
| `salaryAmount` | number | 금액 | `12000` |
| `workHours` | string | 근무 시간 설명 | `09:00 ~ 18:00 (휴게 1시간)` |
| `workDays` | string | 근무 요일 | `월~금` |
| `workPeriod` | string | 근무 기간 | `6개월` |
| `requirements` | string | 자격 요건 | `경비원 신분증 소지자` |
| `benefits` | string | 복리후생 | `중식 제공, 유니폼 지급` |
| `description` | string | 상세 업무 내용 | `아파트 출입 관리...` |
| `physicalIntensity` | string | 업무 강도 등급 | `light` / `moderate` / `heavy` |
| `physicalBadges` | array&lt;string&gt; | 체력 소모 배지 | `["standing", "outdoor"]` |
| `minAge` | number | 최소 연령 | `60` |
| `maxAge` | number | 최대 연령 | `75` |
| `deadline` | timestamp | 마감일 | 2026-05-31 |
| `isActive` | boolean | 공고 활성화 여부 | `true` |
| `rawData` | map | 고용24 원본 데이터 (백업) | `{ ... }` |
| `createdAt` | timestamp | DB 적재 일시 | 2026-04-24T10:00:00Z |
| `updatedAt` | timestamp | 최종 갱신 일시 | 2026-04-24T10:00:00Z |

#### physicalBadges enum
| 값 | 의미 |
|---|---|
| `standing` | 계속 서있기 |
| `sitting` | 좌식 업무 |
| `heavy_lifting` | 무거운 짐 운 반기 |
| `outdoor` | 야외 근무 |
| `repetitive` | 반복 동작 |
| `stairs` | 계단 오르내림 |

---

### 2.3 applications (지원 내역)

```
/users/{userId}/applications/{applicationId}
```

하위 컬렉션으로 저장하여 사용자별 지원 내역을 자연스럽게 그룹화합니다.

| 필드 | 타입 | 설명 | 예시 |
|---|---|---|---|
| `applicationId` | string (문서ID) | 자체 UUID | `app_abc123` |
| `jobId` | string (참조) | 지원한 공고 ID | `KJJ_12345678` |
| `jobTitle` | string | 공고 제목 (denormalized) | `아파트 경비원 모집` |
| `companyName` | string | 기업명 (denormalized) | `OO아파트 관리사무소` |
| `selfIntroduction` | string | 자기소개 (200자 내외) | "경비 업무 경험이 풍부합니다..." |
| `status` | string | 지원 상태 | `submitted` / `reviewing` / `accepted` / `rejected` / `cancelled` |
| `submittedAt` | timestamp | 지원 일시 | 2026-04-24T10:00:00Z |
| `updatedAt` | timestamp | 상태 변경 일시 | 2026-04-24T10:00:00Z |

---

### 2.4 bookmarks (찜한 공고)

```
/users/{userId}/bookmarks/{jobId}
```

| 필드 | 타입 | 설명 | 예시 |
|---|---|---|---|
| `jobId` | string (문서ID) | 공고 ID | `KJJ_12345678` |
| `jobTitle` | string | 공고 제목 (denormalized) | `아파트 경비원 모집` |
| `companyName` | string | 기업명 (denormalized) | `OO아파트` |
| `bookmarkedAt` | timestamp | 찜한 일시 | 2026-04-24T10:00:00Z |

---

### 2.5 cached_api_responses (API 캐시)

```
/cached_api_responses/{cacheKey}
```

고용24 API 응답을 캐싱하여 Rate Limit 대응 및 성능 향상

**⚠️ 비용 경고**: Firestore에 캐시를 저장하면 Blaze 요금제에서 **문서 쓰기(Write) 비용**이 발생함. MVP 데모 수준에서는 무관하나, 향후 트래픽 증가 시 Cloud Function 메모리 캐싱 또는 Firebase Hosting CDN으로 전환 검토 필요

| 필드 | 타입 | 설명 | 예시 |
|---|---|---|---|
| `cacheKey` | string (문서ID) | 요청 파라미터 해시 | `goyong24_list_11110_security_20260424` |
| `endpoint` | string | 호출 엔드포인트 식별자 | `job_list` |
| `params` | map | 요청 파라미터 | `{ region: "11110", category: "security" }` |
| `responseData` | map | 변환된 JSON 데이터 | `{ jobs: [...] }` |
| `expiresAt` | timestamp | 캐시 만료 시간 (기본 1시간) | 2026-04-24T11:00:00Z |
| `createdAt` | timestamp | 캐시 생성 시간 | 2026-04-24T10:00:00Z |

---

## 3. Firestore 보안 규칙 (Security Rules)

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // users: 본인 문서만 읽기/쓰기 가능
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    
    // users 하위 컬렉션 (applications, bookmarks)
    match /users/{userId}/{subcollection}/{docId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    
    // jobs: 누구나 읽기 가능, 쓰기는 관리자만 (Cloud Functions에서 처리)
    match /jobs/{jobId} {
      allow read: if true;
      allow write: if false; // Cloud Functions 전용
    }
    
    // cached_api_responses: 누구나 읽기 가능, 쓰기는 Cloud Functions 전용
    match /cached_api_responses/{cacheKey} {
      allow read: if true;
      allow write: if false;
    }
  }
}
```

---

## 4. 인덱스 설계

### Composite Indexes (복합 인덱스)

| 컬렉션 | 필드 조합 | 용도 |
|---|---|---|
| `jobs` | `locationCode` ASC, `employmentType` ASC, `isActive` ASC, `deadline` ASC | 지역 + 근무형태 필터 정렬 |
| `jobs` | `jobCategory` ASC, `physicalIntensity` ASC, `isActive` ASC, `deadline` ASC | 직종 + 강도 필터 |
| `jobs` | `isActive` ASC, `deadline` ASC | 활성 공고 마감순 |
| `applications` | `status` ASC, `submittedAt` DESC | 지원 상태별 조회 |
| `cached_api_responses` | `endpoint` ASC, `expiresAt` ASC | 캐시 만료 정리 |

---

## 5. 데이터 흐름

### 5.1 공고 동기화 (Cloud Functions)

```
[Cron Job: 매일 06:00, 18:00]
    ↓
[Cloud Function: syncGoyong24Jobs]
    ↓
[고용24 API 호출] → [XML → JSON 변환]
    ↓
[시니어 부적합 공고 필터링]
    ↓
[Firestore /jobs 컬렉션에 upsert]
    ↓
[캐시 만료 처리]
```

### 5.2 지원 흐름

```
[사용자: 지원하기 클릭]
    ↓
[Flutter App: 지원서 작성 (자기소개 200자)]
    ↓
[Firestore: /users/{userId}/applications/{appId} 생성]
    ↓
[Cloud Function: 지원 완료 트리거]
    ↓
[FCM: 사용자에게 지원 완료 알림]
    ↓
[Cloud Function: 고용24 시스템 연동 (가능한 경우)]
```

---

## 6. 확장성 고려사항

| 향후 기능 | 추가 컬렉션/필드 |
|---|---|
| AI 음성 이력서 | `users.voiceTranscript`, `users.aiKeywords` |
| AI 매칭 점수 | `jobs.matchScore` (Cloud Function에서 계산) |
| 기업 회원 | `companies` 컬렉션, `jobs.companyId` 참조 |
| 리뷰/평점 | `jobs/{jobId}/reviews` 하위 컬렉션 |
| 채팅 | `chats` 컬렉션 |
| 근태 관리 | `attendances` 컬렉션 (사용자-공고별 출퇴근 기록) |
