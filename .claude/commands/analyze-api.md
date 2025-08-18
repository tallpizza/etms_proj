# /analyze-api 커스텀 슬래쉬 커맨드

## 명령어 설명

화면 캡쳐, Figma 링크, 또는 화면 설명 텍스트를 입력받아 해당 화면을 구현하기 위해 필요한 API들을 분석하고 TypeScript 인터페이스로 정의합니다.

## 프롬프트

당신은 프론트엔드 화면을 분석하여 필요한 백엔드 API를 설계하는 시니어 개발자입니다.

주어진 화면 정보(이미지, Figma 링크, 또는 설명)를 보고 다음을 수행하세요:

1. **화면 분석**: 화면의 주요 구성 요소와 기능을 파악합니다
2. **필요한 API 도출**: 각 기능에 필요한 API 엔드포인트를 정의합니다
3. **데이터 구조 설계**: 각 API의 요청/응답 타입을 TypeScript로 정의합니다

### 출력 형식:

```typescript
// 1. 화면 분석 요약
// - 주요 기능: [기능 목록]
// - 사용자 상호작용: [인터랙션 목록]

// 2. API 목록
interface APIList {
  // GET /api/[리소스명]
  get[리소스명]: {
    description: "API 목적 설명";
    request: {
      query?: {
        [파라미터명]: [타입]; // 설명
      };
    };
    response: {
      [필드명]: [타입] | { /* 한글 설명 */ };
    };
  };

  // POST /api/[리소스명]
  create[리소스명]: {
    description: "API 목적 설명";
    request: {
      body: {
        [필드명]: [타입] | { /* 한글 설명 */ };
      };
    };
    response: {
      [필드명]: [타입];
    };
  };

  // PUT/PATCH /api/[리소스명]/[id]
  update[리소스명]: {
    description: "API 목적 설명";
    request: {
      params: {
        id: string;
      };
      body: {
        [필드명]?: [타입];
      };
    };
    response: {
      [필드명]: [타입];
    };
  };

  // DELETE /api/[리소스명]/[id]
  delete[리소스명]: {
    description: "API 목적 설명";
    request: {
      params: {
        id: string;
      };
    };
    response: {
      success: boolean;
    };
  };
}

// 3. 공통 타입 정의
interface [모델명] {
  [필드명]: [타입] | {
    /* 복잡한 경우 한글 설명 */
    [서브필드]: [타입];
  };
}

// 4. 상태 코드 및 에러 처리
interface APIError {
  code: number;
  message: string;
  details?: any;
}
```

### 분석 시 고려사항:

1. **리스트 화면**인 경우:

   - 페이지네이션 파라미터 (page, limit, offset)
   - 정렬 파라미터 (sortBy, orderBy)
   - 필터링 파라미터 (status, category, dateRange 등)
   - 검색 파라미터 (search, keyword)

2. **상세 화면**인 경우:

   - 단일 리소스 조회 API
   - 연관 데이터 조회 API (댓글, 태그, 관련 항목 등)
   - 수정/삭제 권한 체크 정보

3. **입력 폼**이 있는 경우:

   - 유효성 검증 규칙
   - 파일 업로드 여부
   - 드롭다운/선택 옵션을 위한 조회 API

4. **인증/권한**이 필요해 보이는 경우:

   - 로그인/로그아웃 API
   - 권한 체크 API
   - 토큰 갱신 API

5. **실시간 기능**이 있는 경우:
   - WebSocket 연결 정보
   - 이벤트 타입 정의
   - 구독/구독 해제 API

### 타입 작성 규칙:

- 정확한 필드명이 불명확한 경우: `{ 코드, 이름, 설명 }` 형태로 한글 설명
- 선택적 필드는 `?` 사용
- 배열은 `[]` 사용
- 날짜는 `string | Date` 타입 사용
- ID는 일반적으로 `string | number` 타입
- 상태값은 `enum` 또는 `union type` 사용

### 예시:

```typescript
// 게시판 목록 화면 분석 결과
interface BoardAPIs {
  // GET /api/posts
  getPostList: {
    description: "게시글 목록 조회";
    request: {
      query?: {
        page?: number; // 페이지 번호
        limit?: number; // 페이지당 항목 수
        category?: string; // 카테고리 필터
        search?: string; // 검색어
      };
    };
    response: {
      posts: Array<{
        id: string;
        title: string;
        author: { 이름; 프로필사진 };
        createdAt: string;
        viewCount: number;
        commentCount: number;
      }>;
      totalCount: number;
      currentPage: number;
    };
  };
}
```

입력된 화면 정보를 바탕으로 위 형식에 맞춰 필요한 API를 분석하고 TypeScript 인터페이스로 정의해주세요.
