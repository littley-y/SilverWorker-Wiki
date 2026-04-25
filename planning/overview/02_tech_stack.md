# 02. SilverWorkerNow 기술 스택 확정 문서

> 확정 일자: 2026-04-24
> 대상: MVP (AI 기능 제외)

---

## 1. 개요

비개발자(사용자) 및 소규모 팀이 60대 이상 시니어 전용 구인구직 앱 MVP를 개발하는 것을 전제로, 학습 곡선이 낮고 빠른 개발이 가능한 기술 스택을 확정합니다.

---

## 2. 프론트엔드

### 선택: Flutter (Dart)

| 항목 | Flutter | React Native |
|---|---|---|
| 학습 곡선 | 중간 (Dart는 JS와 유사) | 쉬움 (JS 알 경우) |
| 환경 설정 | 단순 (flutter doctor) | 복잡 (Node, npm, CocoaPods) |
| UI 일관성 | 픽셀 단위 완벽 동일 | 플랫폼별 네이티브 컴포넌트 |
| 성능 | 우수 (60~120 FPS) | 양호 |
| 시니어 UI 적합성 | 커스텀 위젯/애니메이션에 강함 | 가능하나 번거로움 |

### 확정 이유
- 이미 프로젝트 템플릿으로 Flutter가 구축되어 있음
- iOS/Android UI가 100% 동일하게 구현되어 시니어 UX 통일에 유리
- `flutter_slidable`, `flutter_local_notifications` 등 시니어 앱에 필요한 패키지 생태계 풍부
- 설정 단순: `flutter doctor` 한 번으로 환경 진단 완료

---

## 3. 백엔드 / BaaS

### 선택: Firebase (Google)

| 항목 | Firebase | Supabase | Backendless |
|---|---|---|---|
| DB | Cloud Firestore (NoSQL) | PostgreSQL (SQL) | Hybrid |
| 인증 | 내장 (전화번호, 이메일) | 내장 (이메일, OAuth) | 내장 |
| 푸시 알림 | FCM (묻지도 따지지도 않는 1등) | 외부 연동 필요 | 내장 |
| 실시간 동기화 | 기본 제공 | 지원 | 지원 |
| 오프라인 지원 | 기본 제공 | 추가 설정 필요 | 추가 설정 필요 |
| 묶음 가격 | 사용량 기반 (예측 어려움) | 저장량 기반 (예측 쉬움) | 고정 요금 |
| 묶음 등급 | 1GB Firestore, 50K 일일 읽기 | 500MB DB, 1GB 스토리지 | 제한적 |
| 서버리스 함수 | Cloud Functions | Edge Functions | Cloud Code |

### 확정 이유
- 서버를 직접 구축/관리할 필요 없음 (NestJS 서버 운영은 비개발자에게 매우 어려움)
- 푸시 알림(FCM), 인증, DB, 파일 저장이 한 곳에서 모두 해결됨
- 오프라인 지원 기본 제공: 노후 단말기/불안정한 네트워크 환경에서 필수
- Google Cloud와의 연동으로 고용정보원 API 프록시 구축 용이

### ⚠️ Firebase Blaze 요금제 필수
- Cloud Functions(Node.js 10+) 및 외부 API(고용24) 아웃바운드 네트워크 요청은 **Spark(묶음) 요금제에서 불가능**
- **반드시 Blaze(종량제) 요금제로 업그레이드**하고 결제 수단을 등록해야 함
- 예상 비용: MVP 개발 기간 중 월 $5~20 (트래픽에 따라 변동)

---

## 4. 고용24 API 연동

### API 현황 (2026년 4월 기준)

| 항목 | 내용 |
|---|---|
| **워크넷 Open API** | 종료됨. 고용24(work24.go.kr)로 통합 |
| **고용24 Open API** | 현재 서비스 중. `work24.go.kr` |
| **인증** | 정적 API Key (`authKey`) URL 파라미터에 평문 노출 |
| **응답 형식** | 일부 API는 XML만 지원 (`returnType=XML` 필수) |
| **트래픽 제한** | 계정당 일 100,000회 등 제한 있음 |
| **CORS** | 명시적 CORS 헤더 없음 |

### 연동 아키텍처 (백엔드 프록시 필수)

```
[Flutter App] → HTTPS (API 키 없음)
    ↓
[Firebase Cloud Functions]
    ↓ authKey 포함 (서버측 관리)
[고용24 Open API]
    ↓ XML
[Cloud Functions]
    ↓ XML → JSON 변환 + 캐싱
[Flutter App]
```

### 프록시가 필요한 이유
1. **API 키 보호**: `authKey`가 URL에 평문으로 노출 → 앱에 직접 넣으면 탈취 위험
2. **CORS 대응**: 명시적 CORS 미지원 → 브라우저/앱 직접 호출 불안정
3. **XML → JSON 변환**: 일부 API는 XML만 반환 → 파싱 및 변환 필요
4. **캐싱/Rate Limiting**: 일 100,000회 제한 대응 및 중복 호출 방지
5. **2차 가공**: 시니어 부적합 공고(고난도 기술, 외국어 필수 등) 필터링

### ⚠️ 고용24 API IP 화이트리스트 위험
- 한국 공공 API는 종종 **고정 IP(Static IP) 화이트리스트**를 요구함
- Cloud Functions는 **유동 IP**를 사용하므로, 고용24가 유동 IP를 차단할 수 있음
- **대안**:
  1. 고용24에 유동 IP 허용 여부를 사전 확인
  2. 고정 IP가 필수일 경우: Cloud NAT + Serverless VPC Access (비용 증가)
  3. 또는 가벼운 VPS(AWS EC2, Oracle Cloud 묶음 티어 등)로 프록시 대체

---

## 5. 주요 패키지 / 라이브러리

### Flutter (pubspec.yaml)
| 패키지 | 용도 |
|---|---|
| `flutter_riverpod` | 상태 관리 |
| `cloud_firestore` | Firebase DB 연동 |
| `firebase_auth` | 사용자 인증 |
| `firebase_messaging` | 푸시 알림 (FCM) |
| `firebase_storage` | 이미지/파일 업로드 |
| `cloud_functions` | Firebase Functions 호출 |
| `intl` | 다국어/날짜 포맷 |
| `logger` | 디버깅 |
| `flutter_slidable` | 리스트 슬라이드 액션 |
| `shared_preferences` | 로컬 설정 저장 |

### Firebase Cloud Functions (Node.js 20)
| 기능 | 설명 |
|---|---|
| `fetchGoyong24Jobs` | 고용24 API 호출, XML→JSON 변환, 캐싱 |
| `filterSeniorJobs` | 시니어 부적합 공고 필터링 |
| `notifyNewJobs` | 새 공고 등록 시 푸시 알림 발송 |

---

## 6. 개발 환경

| 항목 | 요구사항 |
|---|---|
| OS | Windows 11 + WSL2 (Ubuntu) |
| Shell | bash (PowerShell 사용 금지) |
| Flutter SDK | 3.x (stable 채널) |
| Firebase CLI | 최신 버전 |
| Node.js | 20 LTS (Cloud Functions용) |
| Git | LF 개행 강제 (`.gitattributes` 설정) |

---

## 7. 결정 사항 요약

| 계층 | 기술 | 결정 이유 |
|---|---|---|
| 프론트엔드 | Flutter | 템플릿 기반, 설정 단순, UI 일관성, 시니어 UX 최적화 용이 |
| 백엔드 | Firebase (BaaS) | 서버리스, 푸시/인증/DB 일체화, 오프라인 지원, 학습 곡선 낮음 |
| API 프록시 | Firebase Cloud Functions | API 키 보호, XML→JSON 변환, 캐싱, Rate Limiting |
| DB | Cloud Firestore | NoSQL (유연한 스키마), 실시간 동기화, 오프라인 지원 |
