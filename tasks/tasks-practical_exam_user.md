# 사용자용 실습심사 페이지 작업 목록

> PRD 문서: `/prd/practical_exam_user_PRD_20250125.md`
> 생성일: 2025-01-25

## 📁 관련 파일

### 신규 생성

- `etms/src/features/practical-exam/api/trainee.ts` - 훈련생용 실습심사 API 클라이언트
- `etms/src/app/(user)/practical-exam/page.tsx` - 실습심사 목록 페이지
- `etms/src/app/(user)/practical-exam/[examId]/page.tsx` - 실습심사 상세 페이지
- `etms/src/features/practical-exam/components/trainee/PracticalExamCard.tsx` - 실습심사 카드 컴포넌트
- `etms/src/features/practical-exam/hooks/useTraineePracticalExams.ts` - React Query 훅

### 수정 필요

- `etms/src/app/(user)/home/page.tsx` - MENU_ITEMS에 실습심사 메뉴 추가
- `etms/src/features/practical-exam/api/index.ts` - trainee API export 추가

## ✅ 작업 목록

- [x] **1.0 실습심사 API 클라이언트 구축**

  - [x] 1.1 API 타입 정의 (TraineePracticalExam, TraineePracticalExamDetail 인터페이스)
  - [x] 1.2 목록 조회 API 메서드 구현 (getTraineePracticalExamList)
  - [x] 1.3 상세 조회 API 메서드 구현 (getTraineePracticalExamDetail)
  - [x] 1.4 API 클라이언트 객체 생성 및 export
  - [x] 1.5 features/practical-exam/api/index.ts에 export 추가

- [ ] **2.0 실습심사 목록 페이지 구현** 🔄

  > **병렬 처리 가능**: 2.1, 2.2는 동시 실행 가능

  - [ ] 2.1 PracticalExamCard 컴포넌트 생성 (TheoryExamCard 기반)
  - [ ] 2.2 React Query 커스텀 훅 생성 (useTraineePracticalExams)
  - [ ] 2.3 목록 페이지 컴포넌트 구현 _(2.1, 2.2 완료 후)_
  - [ ] 2.4 로딩 및 에러 상태 처리 구현
  - [ ] 2.5 페이지네이션 로직 추가
  - [ ] 2.6 빈 상태 UI 구현

- [ ] **3.0 실습심사 상세 페이지 구현**

  - [ ] 3.1 상세 조회 React Query 훅 생성 (useTraineePracticalExamDetail)
  - [ ] 3.2 읽기 전용 모드 PracticalExamTable 래퍼 컴포넌트 생성
  - [ ] 3.3 상세 페이지 컴포넌트 구현
  - [ ] 3.4 뒤로가기 네비게이션 구현
  - [ ] 3.5 로딩 및 에러 상태 처리

- [ ] **4.0 홈 페이지 메뉴 통합**

  - [ ] 4.1 실습심사 메뉴 아이콘 SVG 생성 또는 선택
  - [ ] 4.2 MENU_ITEMS 배열에 실습심사 항목 추가
  - [ ] 4.3 메뉴 색상 및 스타일 설정 (보라색 계열)
  - [ ] 4.4 라우팅 경로 확인 및 테스트

- [ ] **5.0 컴포넌트 재사용 및 최적화** 🔄

  > **병렬 처리 가능**: 모든 하위 작업 동시 실행 가능

  - [ ] 5.1 공통 타입 정의 파일 정리 (types/practical-exam.ts)
  - [ ] 5.2 공통 유틸리티 함수 추출 (날짜 포맷팅, 상태 변환)
  - [ ] 5.3 컴포넌트 성능 최적화 (React.memo 적용)
  - [ ] 5.4 코드 중복 제거 및 리팩토링

## 🎯 완료 기준

- 모든 작업 항목 체크
- TypeScript 타입 에러 없음
- ESLint 에러 없음
- 실습심사 목록 조회 정상 작동
- 실습심사 상세 보기 정상 작동
- 홈 페이지에서 실습심사 메뉴 접근 가능
- 페이지 로딩 성능 기준 충족 (목록 2초, 상세 1초 이내)

## 🔄 병렬 처리 전략

### 병렬 그룹 1 (2.0 작업)
- **동시 실행**: 2.1 (PracticalExamCard), 2.2 (React Query 훅)
- **이유**: 서로 다른 파일이며 독립적인 컴포넌트
- **주의**: 2.3은 반드시 2.1, 2.2 완료 후 실행

### 병렬 그룹 2 (5.0 작업)
- **동시 실행**: 5.1, 5.2, 5.3, 5.4 모두
- **이유**: 각각 다른 측면의 최적화 작업으로 충돌 없음
- **주의**: 다른 작업들이 모두 완료된 후 실행

## 📝 실행 순서 권장사항

1. **1.0 API 클라이언트** (필수 선행)
2. **2.0 목록 페이지** + **4.0 홈 메뉴** (병렬 가능)
3. **3.0 상세 페이지**
4. **5.0 최적화** (마지막)

## ⚠️ 주의사항

- API 엔드포인트는 백엔드 팀과 확인 필요
- 실습심사 테이블 컴포넌트는 읽기 전용 prop 필요
- 이론심사 컴포넌트 재사용 시 네이밍 충돌 주의
- 홈 페이지 메뉴 추가 시 기존 레이아웃 영향 확인