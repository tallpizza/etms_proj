# AGENTS.md

역할: 이 레포지토리에서 코드를 다루는 작업 에이전트(Coding Agent) 운영 가이드.

목표: 파일과 폴더 구조를 기반으로 일관된 작업 방식, 안전한 변경, 검증 가능한 산출물을 보장한다.

## 레포 개요 (Monorepo)

- `etms/` — 프론트엔드 Next.js 15 앱 (TypeScript, TailwindCSS, shadcn/ui)
- `etms_backend/` — 백엔드 Spring Boot 3.5.0 (Java 21, JPA/QueryDSL, MySQL/H2)
- `prd/` — 제품 요구/설계 문서(PRD)
- `tasks/` — PRD 기반 작업 산출물(세부 작업 목록 등)
- `.claude/commands/` — 에이전트용 표준 작업 템플릿/명령 가이드
- `prompt_history/` `logs/` — 히스토리 및 로깅

서브모듈: 이 레포는 Git submodules로 구성된다.

- `etms/` → 프론트엔드 하위 저장소
- `etms_backend/` → 백엔드 하위 저장소

참고 문서:

- 레포 전반: `CLAUDE.md`
- 프론트엔드: `etms/CLAUDE.md`
- 백엔드: `etms_backend/CLAUDE.md`
- 명령 템플릿: `.claude/commands/*`

## 작업 원칙 (MANDATORY)

- 정확성 우선: 추측 금지, 불확실하면 질문하고 가정은 분리 표기.
- 최소 변경: 기존 패턴·스타일을 따르고 영향 범위를 좁힌다.
- 단계적 진행: 계획 → 구현 → 검증 → 요약. 큰 변경은 세분화.
- 가드레일 준수: 각 서브모듈의 `CLAUDE.md` 규칙 위반 시 즉시 중단·수정.
- 재현 가능성: 실행/검증 방법을 함께 남긴다.

## 기본 워크플로우

1. 구조 파악

- 최상위와 대상 서브모듈의 디렉터리/파일 구조를 먼저 살핀다.
- 예: `rg --files --hidden -g '!.git' | head -n 200`로 빠르게 스캔.

2. 요구 분석

- 변경 요구를 PRD(`prd/`)와 기존 코드/문서와 대조하여 명세화.
- 모호점은 닫힌형 질문으로 확인 요청.

3. 계획 수립

- 구현 단위를 2~5단계로 쪼개고, 병렬 가능/의존 관계를 표시.
- 산출물(생성/수정 파일)과 검증 방법을 명시.

4. 구현

- 프론트는 `etms/` 내 feature 기반 구조를 따르고 디자인 시스템 준수.
- 백엔드는 도메인 패키지/QueryDSL/REST 규칙을 따른다.
- 파일 수정은 최소 범위로, 커밋은 목적 단위로 분리(사용자 지시에 따름).

5. 검증

- 프론트: `etms/scripts/check-rules.sh`로 사전 점검, 필요 시 `npm run lint`.
- 백엔드: `./gradlew build` 또는 `./gradlew bootRun --args='--spring.profiles.active=local'`로 확인.
- 로컬 의존(도커/DB) 필요 시 `etms_backend/docker-compose.local.yml` 참고.

6. 결과 정리

- 변경 요약, 영향도(파일/모듈), 실행/롤백 방법, 다음 단계 제안.

## 폴더 구조 요약(핵심)

- `etms/src/`

  - `components/` UI·Compound 컴포넌트(DataTable 등)
  - `hooks/` `lib/` `types/` `constants/` 공통 모듈
  - `app/` App Router 레이아웃·페이지
  - `scripts/` 규칙 점검 스크립트(`check-rules.sh`, `post-work-check.sh`)

- `etms_backend/src/main/java/com/eastarjet/etms/`

  - `common/` `config/` 공통/설정
  - `training/` `curriculum/` `member/` `license/` 등 도메인 패키지
  - REST 컨트롤러 → 서비스 → 리포지토리 계층, QueryDSL 기반 조회

- `.claude/commands/` — 작업 분해·검증용 템플릿(`task.md`, `check.md`, `sc:*` 등)

## 구현 규칙(요약)

프론트엔드(상세는 `etms/CLAUDE.md`):

- 색상 하드코딩 금지 → 시스템/시맨틱 토큰 사용.
- 타이포는 `Typo` 컴포넌트 필수, 버튼은 `Button` 변형 사용.
- 큰 컴포넌트(>500LOC) 선(先)분리, 중복 로직은 훅/유틸로 흡수.
- feature 디렉터리 구조를 지키고 API 클라이언트/React Query 패턴 준수.

백엔드(상세는 `etms_backend/CLAUDE.md`):

- REST 설계 일관성, Nested Resource 활용, 페이징은 DB 레벨.
- 엔티티는 BaseEntity/UUID·감사 필드, 순환참조 방지 설정.
- 조회는 QueryDSL 우선, N+1·연결/리소스 누수 방지.

## 검증 체크리스트

사전(Pre-work):

- 프론트 규칙 위반 스캔: `etms/scripts/check-rules.sh`
- 큰 파일/중복/금지 패턴 존재 여부 점검.

사후(Post-work):

- 프론트: `npm run lint`(또는 IDE 진단)·필요 시 `post-work-check.sh`.
- 백엔드: `./gradlew build` 결과 확인. 로컬 프로필 실행 검증.

## 실행 예시

프론트(개발):

```bash
cd etms
npm install
npm run dev
```

백엔드(로컬 실행):

```bash
cd etms_backend
export $(cat .env | xargs)
./gradlew bootRun --args='--spring.profiles.active=local'
```

DB 초기화(로컬):

```bash
cd etms_backend
docker-compose -f docker-compose.local.yml up -d
```

## 도구 사용 팁

- 파일/텍스트 검색은 `rg` 사용 권장. 출력은 200~250라인 단위로 확인.
- Serena MCP 사용 시 작업 루트는 레포 최상위여야 한다.
  - `relative_path`: 백엔드 `"etms_backend"`, 프론트 `"etms"`를 기준으로 지정.

## 참고 문서

- 레포 전반 가이드: `CLAUDE.md`
- 프론트 가이드: `etms/CLAUDE.md`
- 백엔드 가이드: `etms_backend/CLAUDE.md`
- 명령/템플릿: `.claude/commands/`
