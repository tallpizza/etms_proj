# Run Task

Guidelines for managing task lists in markdown files to track progress on completing a PRD

## Task Hierarchy Definition

- **Parent Task**: 메인 번호 태스크 (예: 1.0, 2.0, 3.0)
- **Sub-task**: 하위 번호 태스크 (예: 1.1, 1.2, 1.3)
- **Parallel Tasks**: 🔄 표시된 Parent task는 병렬 처리 가능

## Task Implementation

### Sequential Execution (기본)

- **One Parent task at a time:** 하나의 Parent task(예: 1.0)와 그 모든 sub-task들을 완료한 후, 다음 Parent task 시작 전에 사용자 확인을 받아야 함
- **Parent task 내에서는 연속 작업:** Parent task 내의 모든 sub-task들은 연속적으로 진행하되, 각 sub-task 완료 시마다 진행상황을 업데이트

### Parallel Execution (🔄 표시된 작업)

- **병렬 처리 가능 Parent task:** 🔄 아이콘이 있는 Parent task는 내부 sub-task들을 병렬로 처리
- **Subagent task 실행:**
  - 독립적인 sub-task들을 식별하여 동시 실행
  - 의존성이 명시된 sub-task는 선행 작업 완료 후 실행
  - 예: `3.3 [작업 설명] *(3.1, 3.2 완료 후 실행)*`

### Completion Protocol

1. **Sub-task 완료 처리:**
   - Sequential: 각 sub-task 완료 시 즉시 `[ ]`를 `[x]`로 변경
   - Parallel: 병렬 실행된 sub-task들이 모두 완료되면 일괄 `[x]`로 변경
2. **Parent task 완료:**
   - 모든 sub-task가 `[x]`가 되면 Parent task도 `[x]`로 변경
   - Parent task 완료 후 다음 Parent task 시작 전에 사용자 승인 요청

## Task List Maintenance

1. **Update the task list as you work:**

   - Mark tasks and subtasks as completed (`[x]`) per the protocol above
   - Add new tasks as they emerge with appropriate dependencies
   - Identify newly discovered parallel opportunities with 🔄

2. **Maintain the "Relevant Files" section:**
   - List every file created or modified
   - Note file conflicts for parallel tasks
   - Give each file a one-line description of its purpose

## AI Instructions

When working with task lists, the AI must:

### 1. Task Execution Strategy

- **Sequential Parent tasks (🔄 없음)**: 순차적으로 모든 sub-task 실행
- **Parallel Parent tasks (🔄 있음)**:
  - 독립적인 sub-task들을 subagent로 동시 실행
  - 의존성 체인 확인 후 순차 작업 처리

### 2. Subagent Task Management

- **병렬 실행 관리:**
  - 🔄 표시된 Parent task의 독립적인 sub-task들을 subagent로 동시 실행
  - 완료된 작업만 `[x]`로 표시
  - 의존성이 있는 작업은 선행 작업 완료 후 자동 시작

### 3. Progress Tracking

- 완료된 sub-task만 `[x]`로 표시
- 병렬 처리 중인 작업은 완료 시점에만 업데이트
- 충돌 또는 문제 발생 시 즉시 보고

### 4. Parent Task Completion

- Parent task의 모든 sub-task 완료 후:
  - Parent task를 `[x]`로 표시
  - "Relevant Files" 섹션 업데이트
  - 병렬 처리 효율성 보고 (시간 단축률 등)
  - 사용자에게 다음 Parent task 진행 승인 요청

### 5. User Approval

- 사용자가 "yes" 또는 "y"로 승인한 후에만 다음 Parent task 시작
- 병렬 처리 가능한 다음 작업이 있을 경우 알림

### 6. Dynamic Task Management

- 새로 발견된 태스크는 적절한 Parent task 하위에 추가
- 병렬 처리 가능한 새 작업 발견 시 🔄 표시 추가 제안

## Execution Examples

### Sequential Execution

```
현재 작업: Task 1.0 데이터 모델 설계
- [x] 1.1 엔티티 정의
- [x] 1.2 관계 설정
- [ ] 1.3 인덱스 설계
- [ ] 1.4 유효성 검사 규칙
```

### Parallel Execution

```
현재 작업: Task 3.0 API 개발 🔄
- [x] 3.1 사용자 API
- [x] 3.2 상품 API
- [ ] 3.3 통합 테스트 (3.1, 3.2 완료 후)
- [ ] 3.4 API 문서화
```
