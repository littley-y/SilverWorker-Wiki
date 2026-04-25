# Spec 02. 인증 및 프로필 등록

> 대상 Day: Day 2
> 참조: `overview/03_mvp_specs.md` AUTH-01~03, `overview/04_db_schema.md` users 컬렉션

---

## 1. 화면 목록

| 화면 | 라우트 | 설명 |
|---|---|---|
| PhoneInputScreen | `/auth/phone` | 휴대폰 번호 입력 |
| OtpInputScreen | `/auth/otp` | SMS 인증코드 6자리 입력 |
| ProfileSetupScreen | `/auth/profile` | 최초 1회 프로필 등록 |

---

## 2. 인증 흐름

```
앱 실행
  ↓
authStateProvider 확인
  ├─ 로그인 상태 + 프로필 있음 → MainScreen (Bottom Nav)
  ├─ 로그인 상태 + 프로필 없음 → ProfileSetupScreen
  └─ 미로그인 → PhoneInputScreen
```

**자동 로그인**: `FirebaseAuth.instance.authStateChanges()` 가 `User` 를 반환하면 자동으로 메인으로 진입. 별도 만료 처리 없음 (Firebase 세션 기본 유지).

---

## 3. PhoneInputScreen

### UI 요소
- 타이틀: "휴대폰 번호로 시작하세요" (24pt Bold)
- 번호 입력 필드: `+82` 국가 코드 고정, 나머지 10~11자리 입력
- "인증번호 받기" 버튼 (56dp 높이, 메인 컬러)
- 하단 안내문: "가입과 로그인에 모두 사용됩니다" (14pt, 회색)

### 로직
```dart
// 인증 시작
await FirebaseAuth.instance.verifyPhoneNumber(
  phoneNumber: '+82$number',
  verificationCompleted: (credential) { /* 자동 인증 처리 */ },
  verificationFailed: (e) { /* 오류 스낵바 표시 */ },
  codeSent: (verificationId, resendToken) {
    // verificationId를 OtpInputScreen으로 전달
    context.push('/auth/otp', extra: verificationId);
  },
  codeAutoRetrievalTimeout: (verificationId) {},
  timeout: const Duration(seconds: 60),
);
```

### 에러 처리
| 상황 | 처리 |
|---|---|
| 번호 형식 오류 | 입력 필드 빨간 테두리 + "올바른 번호를 입력하세요" |
| SMS 발송 실패 | 스낵바: "잠시 후 다시 시도해 주세요" |
| 네트워크 없음 | 스낵바: "인터넷 연결을 확인해 주세요" |

---

## 4. OtpInputScreen

### UI 요소
- 타이틀: "인증번호 6자리를 입력하세요" (22pt Bold)
- 6칸 분리 입력 필드 (각 칸 56dp × 64dp)
- 자동완성: SMS 수신 시 `smsRetriever` 또는 Android 자동 완성 API 활용 (가능한 경우)
- 재발송 버튼: 60초 카운트다운 후 활성화
- "확인" 버튼

### 로직
```dart
final credential = PhoneAuthProvider.credential(
  verificationId: verificationId,
  smsCode: enteredCode,
);
await FirebaseAuth.instance.signInWithCredential(credential);
// 성공 시 authStateProvider가 자동으로 화면 전환 처리
```

### 에러 처리
| 상황 | 처리 |
|---|---|
| 코드 불일치 | "인증번호가 맞지 않습니다. 다시 확인해 주세요" |
| 코드 만료 | "인증번호가 만료되었습니다. 재발송해 주세요" |

---

## 5. ProfileSetupScreen

### UI 요소
- 이름 입력 (필수, 최대 20자)
- 거주 지역 선택: 시/도 드롭다운 → 구/군 드롭다운 (2단계)
- 간단한 경력 소개 텍스트 에어리어 (선택, 최대 500자, 글자 수 카운터 표시)
- "시작하기" 버튼

### Firestore 저장 경로
`/users/{uid}` — `overview/04_db_schema.md` users 컬렉션 필드 기준

```dart
await FirebaseFirestore.instance.collection('users').doc(uid).set({
  'userId': uid,
  'phoneNumber': phoneNumber,
  'name': name,
  'address': {'sido': sido, 'sigungu': sigungu},
  'careerSummary': career,
  'isPushEnabled': true,
  'createdAt': FieldValue.serverTimestamp(),
  'updatedAt': FieldValue.serverTimestamp(),
});
```

### 에러 처리
| 상황 | 처리 |
|---|---|
| 이름 미입력 | "이름을 입력해 주세요" (버튼 비활성화) |
| 지역 미선택 | "거주 지역을 선택해 주세요" (버튼 비활성화) |
| Firestore 저장 실패 | 스낵바 + 재시도 버튼 |

---

## 6. 완료 기준 (Day 2 DoD)

- [ ] 실제 휴대폰 번호로 SMS 인증 완료
- [ ] 앱 재실행 시 로그인 화면 없이 메인으로 바로 진입 (자동 로그인)
- [ ] 프로필 저장 후 Firestore Console에서 `/users/{uid}` 문서 확인
- [ ] 잘못된 인증코드 입력 시 오류 메시지 표시 확인
