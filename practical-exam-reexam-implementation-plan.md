# PracticalExamSubmission 재심사 시스템 구현 계획

## 📌 개요

온톨로지 기반 실습심사 재심사 시스템을 Spring Boot 백엔드에 구현합니다.

### 핵심 개념
- **PracticalQuestion.maxPassingCount**: 한 Training 내 최대 시도 횟수
- **TrainingTemplateVersion.maxPassingCount**: 전체 TrainingTemplate 최대 시도 횟수
- **globalAttemptCount**: 같은 Member + TrainingTemplate의 전체 누적 시도 횟수
- **reexamLevel**: 재심사 단계 (0: 초기, 1: 1차 재심사, 2: 2차 재심사)

### 비즈니스 로직
- PQ.max가 한 Training 내에서의 제한
- TT.max가 전체 TrainingTemplate에서의 절대 제한
- 예: PQ.max=2, TT.max=3인 경우
  - 첫 Training에서 2번 시도 가능
  - 실패 시 새 Training에서 1번만 더 시도 가능 (총 3번)

---

## 🏗️ 구현 작업

### 1. Entity 수정

#### 1.1 PracticalExamSubmission.java
- **PassStatus enum 확장**
  - 기존: PENDING, PASS, FAIL, REEXAM, REEXAM_PASS, REEXAM_FAIL
  - 추가: REEXAM2_PASS, REEXAM2_FAIL, FINAL_FAIL

- **새 필드 추가**
  - `globalAttemptCount`: 전체 시도 횟수 (Integer, default=0)
  - `reexamLevel`: 재심사 단계 (Integer, default=0)
  - `trainingTemplateId`: 쿼리 최적화용 (UUID)

#### 1.2 TrainingTemplateVersion.java
- 기존 `maxPassingCount` 필드 활용 (수정 불필요)
- 전체 최대 시도 횟수로 사용

#### 1.3 PracticalQuestion.java
- 기존 `maxPassingCount` 필드 활용 (수정 불필요)
- Training 내 최대 시도 횟수로 사용

### 2. Database 수정

#### 2.1 테이블 스키마 변경
```sql
ALTER TABLE practical_exam_submissions 
ADD COLUMN global_attempt_count INT DEFAULT 0,
ADD COLUMN reexam_level INT DEFAULT 0,
ADD COLUMN training_template_id CHAR(36);
```

#### 2.2 인덱스 추가
```sql
CREATE INDEX idx_submission_member_template 
ON practical_exam_submissions(member_id, training_template_id, pass_status);
```

### 3. Repository 계층

#### 3.1 PracticalExamSubmissionRepository 추가 메서드
- `countPreviousAttempts()`: 이전 시도 횟수 카운트 (합격/탈락 제외)
- `findByMemberAndTrainingTemplate()`: 멤버의 TrainingTemplate별 모든 submission 조회
- `countAttemptsByMembers()`: 배치 처리용 멤버별 attempt count

### 4. Service 계층

#### 4.1 AttemptCalculationService (신규)
- `calculateGlobalAttemptCount()`: 전체 시도 횟수 계산
- `calculateReexamLevel()`: 재심사 레벨 계산
- `calculateAvailableAttempts()`: 현재 Training에서 가능한 시도 횟수
- `calculateBatchAttemptCounts()`: 배치 처리용 카운트

#### 4.2 StatusTransitionService (신규)
- `updateSubmissionStatus()`: 평가 결과에 따른 상태 전이
- `handlePassResult()`: 합격 처리 로직
- `handleFailResult()`: 불합격 처리 및 재심사 판정

#### 4.3 PracticalExamService 수정
- `createSubmission()`: globalAttemptCount 계산 및 검증 추가
- `saveEvaluation()`: 상태 전이 로직 호출 추가
- `getSubmissionDetail()`: 남은 시도 횟수 정보 추가
- `getSubmissions()`: 배치 처리로 성능 최적화

### 5. API 수정

#### 5.1 Controller 수정 사항
- 제출 상세 조회: globalAttemptCount, reexamLevel, remainingAttempts 추가
- 제출 목록 조회: 각 submission의 상태 실시간 계산
- 제출 생성: 최대 시도 횟수 초과 체크

#### 5.2 DTO 수정
- `GetSubmissionDetailResponse`: 재심사 관련 필드 추가
- `PracticalExamSubmission`: attemptStatus 표시 추가

---

## 🧪 테스트 데이터 생성

### 1. PracticalExamReexamTestDataInitializer.java (신규)

#### 1.1 테스트 시나리오
| 시나리오 | PQ.max | TT.max | 상황 설명 | 테스트 목적 |
|---------|--------|---------|-----------|------------|
| 1 | 1 | 2 | Member A: 1번 실패, 재심사 대기 | 단순 재심사 가능 |
| 2 | 1 | 3 | Member B: 2번 실패, 2차 재심사 대기 | 다중 재심사 |
| 3 | 2 | 2 | Member C: 2번 실패, 재심사 불가 | 최대 도달 |
| 4 | 2 | 3 | Member D: 2번 실패, 마지막 시도 가능 | 경계 케이스 |

#### 1.2 데이터 구성
- 각 시나리오별 TrainingTemplateVersion 생성
- 테스트용 Member 4명 생성 (test_a, test_b, test_c, test_d)
- 실패 상태의 PracticalExamSubmission 생성
- PracticalExamItemResult로 실패 표시 (comments 추가)

### 2. 실행 방법
- TemplateBasedTrainingDataInitializer의 @PostConstruct에서 호출
- 애플리케이션 시작 시 자동으로 테스트 데이터 생성

---

## 🔄 상태 전이 다이어그램

### 1. 기본 상태 전이
```
PENDING → (평가)
  ├─ 합격 → PASS / REEXAM_PASS / REEXAM2_PASS
  └─ 불합격
      ├─ 시도 남음 → REEXAM
      ├─ Training 내 최대 도달 → FAIL / REEXAM_FAIL / REEXAM2_FAIL
      └─ 전체 최대 도달 → FINAL_FAIL
```

### 2. Cross-Training 시나리오
```
Training 1: PENDING → FAIL/REEXAM_FAIL
Training 2: PENDING → REEXAM_PASS/REEXAM_FAIL  
Training 3: PENDING → REEXAM2_PASS/FINAL_FAIL
```

---

## 📊 검증 항목

### 1. 기능 검증
- [ ] globalAttemptCount 정확한 계산
- [ ] Cross-Training 간 시도 횟수 누적
- [ ] TT.maxPassingCount 초과 시 FINAL_FAIL
- [ ] PQ.maxPassingCount 초과 시 다음 Training 이동
- [ ] reexamLevel 올바른 증가

### 2. 데이터 검증
- [ ] 4개 시나리오 테스트 데이터 정상 생성
- [ ] 각 시나리오별 상태 정확성
- [ ] ItemResult 실패 표시 정상 동작

### 3. 성능 검증
- [ ] 배치 쿼리 N+1 문제 해결
- [ ] 인덱스 활용도
- [ ] 대량 데이터 처리 성능

---

## 🚀 구현 순서

1. **Phase 1: Entity 및 DB 수정** (30분)
   - PracticalExamSubmission entity 수정
   - DB 마이그레이션 스크립트 작성
   - 인덱스 추가

2. **Phase 2: Repository 구현** (30분)
   - 쿼리 메서드 추가
   - JPQL 쿼리 작성

3. **Phase 3: Service 계층** (1시간)
   - AttemptCalculationService 구현
   - StatusTransitionService 구현
   - PracticalExamService 수정

4. **Phase 4: 테스트 데이터 생성** (45분)
   - PracticalExamReexamTestDataInitializer 구현
   - 4개 시나리오 데이터 생성

5. **Phase 5: API 수정** (30분)
   - Controller 메서드 수정
   - DTO 업데이트

6. **Phase 6: 검증** (45분)
   - API 테스트
   - 시나리오별 동작 확인

**총 예상 시간: 4시간**

---

## 📝 API 테스트 예시

### 1. 재심사 가능 확인 (Member A)
```
GET /api/v1/practical-exam/submissions?memberId={memberA}
Expected: globalAttemptCount=1, 재심사 가능
```

### 2. 최대 시도 초과 확인 (Member C)
```
POST /api/v1/practical-exam/submissions
Body: { "memberId": "{memberC}", ... }
Expected: 400 Bad Request - "최대 시도 횟수 초과"
```

### 3. 마지막 시도 생성 (Member D)
```
POST /api/v1/practical-exam/submissions
Body: { "memberId": "{memberD}", ... }
Expected: 201 Created - 마지막 시도 (globalAttemptCount=2)
```

---

## 🎯 예상 결과

구현 완료 후:
- 복잡한 재심사 로직 자동화
- Cross-Training 재심사 완벽 지원
- 명확한 시도 횟수 관리 및 추적
- 다양한 시나리오 테스트 가능
- 사용자 친화적인 재심사 상태 표시

---

## 📌 주의사항

1. **데이터 일관성**
   - globalAttemptCount는 항상 이전 submission들의 합계와 일치해야 함
   - reexamLevel은 실패한 Training 수와 일치해야 함

2. **성능 고려사항**
   - 대량 조회 시 배치 쿼리 사용 필수
   - 인덱스 활용 최적화 필요

3. **비즈니스 규칙**
   - FINAL_FAIL 상태는 되돌릴 수 없음
   - 새 Training 생성 시에도 이전 기록 유지

4. **테스트 데이터**
   - 운영 환경에서는 테스트 데이터 생성 비활성화
   - 개발/테스트 환경에서만 활성화