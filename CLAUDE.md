# CLAUDE.md

역할: 사실 검증·오류 탐지 우선의 Claude Code.

규칙:

- 친절함보다 진실성·정확성. 불필요한 완곡어 금지(무례 금지).
- 사용자의 주장·전제를 먼저 점검하고 거짓·과장·모순·논리 오류를 즉시, 단호하고 간결하게 지적.
- 근거 없으면 "근거 부족"이라 명시하고 추측 금지.
- 수치·코드·절차는 계산/검증 후만 제시; 가능하면 반례·테스트 포함.
- 가정·제약·불확실성·출처를 분리해 명시. 안전 정책 위반은 짧게 거부하고 대안 제시.

형식: ①오류/리스크 → ②정정·근거 → ③다음 조치(예: 테스트, 코드 수정). 길게 말하지 말 것.

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ETMS Project (Training Management System) is a monorepo containing both frontend and backend applications for EastarJet's airline training management system (이스타항공 훈련관리시스템). The project uses Git submodules to manage two separate repositories:

- **Frontend**: Next.js 15 application (`etms/`)
- **Backend**: Spring Boot 3.5.0 application (`etms_backend/`)

## Repository Structure

This is a Git submodules setup where:

- `etms/` → `git@github.com:eastarjet-etms/etms_frontend.git`
- `etms_backend/` → `git@github.com:eastarjet-etms/etms_backend.git`

## Common Development Commands

### Git Submodules Management

```bash
# Initialize and update submodules
git submodule init
git submodule update

# Update submodules to latest commits
git submodule update --remote

# Clone with submodules
git clone --recurse-submodules <repo-url>

# Work in submodule directory
cd etms/          # Frontend development
cd etms_backend/  # Backend development
```

### Frontend Commands (etms/)

```bash
cd etms/

# Development
npm install
npm run dev        # Start dev server at http://localhost:3000

# Build and production
npm run build
npm start
npm run lint
```

### Backend Commands (etms_backend/)

```bash
cd etms_backend/

# Development
./gradlew build
./gradlew bootRun --args='--spring.profiles.active=local'

# Tools
./gradlew compileJava  # Generate QueryDSL classes
```

#### Database Management (데이터베이스 관리)

**로컬 개발 환경**:

- **데이터 저장**: Docker 볼륨 (`etms_backend_mysql_data`)
- **데이터 영속화**: 애플리케이션 재시작 시 데이터 유지
- **초기화 방식**: 조건부 초기화 (DB가 비어있을 때만 실행)

**실행 프로세스**:

```bash
# 첫 실행 (DB 초기화 필요시)
docker-compose -f docker-compose.local.yml up -d
export $(cat .env | xargs)
./gradlew bootRun --args='--spring.profiles.active=local' | grep -v "^\s*at "
# → 자동으로 테이블 생성 + 초기 데이터 로드

# 재시작 (데이터 유지)
export $(cat .env | xargs)
./gradlew bootRun --args='--spring.profiles.active=local' | grep -v "^\s*at "
# → 기존 데이터 유지

# 데이터 완전 초기화
docker-compose -f docker-compose.local.yml down -v  # 볼륨 삭제
docker-compose -f docker-compose.local.yml up -d    # 새 볼륨으로 재시작
export $(cat .env | xargs)
./gradlew bootRun --args='--spring.profiles.active=local' | grep -v "^\s*at "
```

**상태 확인**:

```bash
# 컨테이너 상태
docker-compose -f docker-compose.local.yml ps

# MySQL 접속
docker exec -it etms_backend-mysql-1 mysql -u etms -petms etms

# 볼륨 확인
docker volume ls | grep mysql
```

## Architecture Overview

### Frontend Architecture (Next.js 15)

- **Tech Stack**: Next.js 15, TailwindCSS v4, shadcn/ui, Zustand, React Query
- **Patterns**: App Router, Compound Components, Feature-based organization
- **API Integration**: Axios client with backend at `https://etmsapi.eland.co.kr`

### Backend Architecture (Spring Boot 3.5.0)

- **Tech Stack**: Java 21, Spring Boot, JPA/Hibernate, QueryDSL, MySQL/H2
- **Authentication**: Keycloak integration with JWT tokens
- **Storage**: AWS S3 for file management
- **Documentation**: OpenAPI/Swagger at `/swagger-ui/index.html`

## High-Level System Architecture

### Domain Structure

The system is organized around core airline training domains:

**Training Management Flow**:

1. **TrainingTemplate** → defines curriculum structure
2. **Training** → actual training instances with auto-generated **Subjects**
3. **DailySubject** → scheduled training sessions with instructor assignments
4. **Attendance** → trainee participation tracking

**Core Entities**:

- **Member** → users (trainees, instructors, administrators)
- **License** → pilot/cabin crew certifications and qualifications
- **Classroom** → physical training locations and equipment
- **Material** → training content and documentation

### API Integration Pattern

**Backend API Structure**:

```
/api/v1/trainings/{trainingId}/
├── attendances        # Training attendance records
├── trainees          # Assigned trainees
├── daily-subjects    # Scheduled sessions
└── subjects          # Auto-generated curriculum subjects
```

**Frontend State Management**:

- **Server State**: React Query for API data caching
- **Client State**: Zustand for complex UI state (especially DataTable)
- **Forms**: React Hook Form with Zod validation

### Security Architecture

- **Authentication**: Keycloak OAuth2 + JWT tokens
- **Frontend**: Token stored in localStorage with automatic refresh
- **Backend**: Spring Security with OAuth2 Resource Server
- **Authorization**: Role-based access control (RBAC)

## Development Guidelines

### Frontend Development Rules

**Design System Compliance (MANDATORY)**:

- Never use hardcoded colors: `text-[#color]` → use `text-gray-900`
- Always use Typo component: `<h1>` → `<Typo variant="h1">`
- Use Button variants: custom buttons → `<Button variant="solid" color="black">`

**Component Architecture**:

- Compound components for complex UI (DataTable pattern)
- Feature-based file organization under `src/features/`
- Co-locate components with pages when feature-specific
- Extract custom hooks for complex business logic

### Backend Development Rules

**API Design Patterns (MANDATORY)**:

- Use Nested Resource structure: `/trainings/{id}/attendances`
- Consolidate similar endpoints with query parameters
- DB-level pagination only (no memory-based pagination)
- QueryDSL for complex queries and filtering

**Entity Design**:

- All entities extend BaseEntity with UUID primary keys
- Use `@JsonIgnoreProperties` to prevent circular references
- Auto-generation patterns (Training → Subject creation)

### Development Workflow

1. **Frontend Development**: Work in `etms/` directory
2. **Backend Development**: Work in `etms_backend/` directory
3. **Cross-cutting Changes**: May require commits to both submodules
4. **Integration**: Frontend connects to backend via configured API endpoints

## Common Integration Points

### Authentication Flow

1. Frontend login → Keycloak authentication
2. JWT token stored and managed by frontend auth system
3. Backend validates JWT on each API request
4. Frontend automatically refreshes expired tokens

### Data Flow Patterns

- **Training Creation**: Frontend multi-step form → Backend auto-generates Subjects
- **Schedule Management**: Calendar drag-drop → Real-time API updates
- **File Uploads**: Frontend → Backend → S3 storage

### Shared Types and Contracts

- Backend OpenAPI spec available at `backend-api.json` in frontend
- Common data structures: PaginatedResponse, Member, Training entities
- Date/time arrays from backend converted by frontend utilities

## Performance Considerations

### Frontend Optimizations

- React.memo for expensive components
- Zustand selective subscriptions for DataTable state
- Dynamic imports for code splitting
- Debounced search and filtering

### Backend Optimizations

- QueryDSL for efficient database queries
- Fetch joins to prevent N+1 problems
- Database-level pagination and filtering
- Connection pooling and query optimization

## Development Environment Setup

### Prerequisites

- Node.js 18+ (frontend)
- JDK 21+ (backend)
- Git with SSH access to submodule repositories

### Local Development

1. Clone with submodules: `git clone --recurse-submodules`
2. Frontend: `cd etms && npm install && npm run dev`
3. Backend: `cd etms_backend && ./gradlew bootRun --args='--spring.profiles.active=local'`
4. Access frontend at http://localhost:3000
5. Access backend API docs at http://localhost:8080/swagger-ui/index.html

### External Dependencies

- **Backend API**: https://etmsapi.eland.co.kr (production)
- **Keycloak**: Configured for authentication
- **AWS S3**: File storage integration
- **Figma Design**: [Design System Reference](https://www.figma.com/design/01WxUktul94bsQCVTCD4Ig/)

## Important Notes

- Each submodule has its own CLAUDE.md with detailed development guidelines
- Follow established patterns from existing codebase
- Frontend emphasizes design system compliance and component reusability
- Backend focuses on API consistency and database performance
- Both applications are production-ready with comprehensive error handling and logging

## Serena MCP Usage Guidelines

**IMPORTANT: Serena MCP must always be executed from the project root directory**

- Always verify current directory with `pwd` before using serena tools
- Project root should be: `/Users/tallpizza/source/codestep/etms_proj`
- If in a subdirectory (e.g., etms_backend), use `cd ..` to return to root
- When using `relative_path` parameter:
  - For backend code: use `"etms_backend"`
  - For frontend code: use `"etms"`
  - Never use paths like `src/main/java` directly

#백엔드가 실행되고 있을때 컴파일만 실행하면 백엔드는 자동 재시작이 된다.
