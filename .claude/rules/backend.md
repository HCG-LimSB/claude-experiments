# 백엔드 규칙

## 스택

- **런타임**: Next.js API Routes / Server Actions (Vercel Edge 또는 Node.js Runtime)
- **DB**: Neon Serverless PostgreSQL
- **ORM**: Drizzle ORM
- **배포**: Vercel

## 디렉토리 구조

```
src/
  db/
    schema/       # Drizzle 테이블 스키마 정의
    migrations/   # Drizzle 마이그레이션 파일
    index.ts      # DB 클라이언트 초기화 및 export
  server/
    actions/      # Next.js Server Actions
    api/          # API Route Handlers (route.ts)
    repositories/ # DB 쿼리 함수 모음 (Drizzle 쿼리)
    services/     # 비즈니스 로직
  types/          # 공유 타입 (프론트/백엔드 공통)
  schemas/        # 유효성 검사 스키마 (zod)
```

## DB / Drizzle 규칙

- 모든 테이블 스키마는 `db/schema/` 에 도메인별로 파일을 분리하여 정의한다.
  - 예: `db/schema/user.ts`, `db/schema/post.ts`
- 스키마 변경은 반드시 Drizzle 마이그레이션으로 관리한다 — 직접 DB 수정 금지.
- DB 클라이언트는 `db/index.ts` 에서만 초기화하고, 다른 곳에서 직접 초기화 금지.
- 쿼리는 반드시 `server/repositories/` 에 도메인별로 모아서 작성한다.
- 컴포넌트, Server Action, API Route에서 Drizzle 쿼리를 직접 작성 금지.

## 레이어 구조 및 책임

```
Client → Server Action / API Route → Service → Repository → DB
```

- **API Route / Server Action**: 요청 수신, 입력값 검증, Service 호출, 응답 반환만 담당.
- **Service**: 비즈니스 로직 담당. DB를 직접 알지 않고 Repository를 통해서만 접근.
- **Repository**: Drizzle 쿼리만 담당. 비즈니스 로직 작성 금지.

## Server Actions 규칙

- Server Action은 `"use server"` 디렉티브를 반드시 파일 최상단에 선언한다.
- 입력값은 반드시 zod 스키마로 검증한 후 사용한다.
- 인증/권한 확인은 Server Action 진입 시점에서 먼저 처리한다.
- Server Action에서 직접 DB 쿼리 작성 금지 — 반드시 Service를 거친다.

## API Route 규칙

- 파일명은 Next.js 컨벤션을 따른다 — `route.ts`.
- HTTP 메서드는 명시적으로 export한다 (`GET`, `POST`, `PUT`, `DELETE`).
- 응답은 항상 `NextResponse.json()`으로 통일한다.
- 입력값은 반드시 zod 스키마로 검증한 후 사용한다.
- 인증/권한 확인은 Route Handler 진입 시점에서 먼저 처리한다.

## 타입 규칙

- Drizzle 스키마에서 타입을 추론하여 사용한다 — 별도 중복 선언 금지.
  ```ts
  // Good
  type User = typeof users.$inferSelect;
  type NewUser = typeof users.$inferInsert;
  ```
- 프론트/백엔드 공통으로 사용하는 타입은 `types/` 에 선언한다.

## 유효성 검사

- 모든 외부 입력(API, Server Action, Form)은 반드시 zod로 검증한다.
- zod 스키마는 `schemas/` 에 도메인별로 정의하고 재사용한다.
- 프론트엔드 폼 검증과 백엔드 입력 검증은 같은 스키마를 공유한다.

## 에러 처리

- Repository, Service에서 발생하는 에러는 명시적으로 throw한다.
- API Route / Server Action에서 try-catch로 에러를 잡고 적절한 응답을 반환한다.
- 클라이언트에 DB 에러 메시지를 그대로 노출하지 않는다.
- 에러 응답은 `{ error: string }` 형태로 통일한다.

## 환경변수

- DB 연결 정보 등 민감한 값은 반드시 환경변수로 관리한다.
- 환경변수는 `configs/` 에서 중앙 관리하고, 직접 `process.env` 참조 금지.
- Vercel 환경변수와 로컬 `.env` 를 구분하여 관리한다 (`.env.local`, `.env.production`).
- `NEXT_PUBLIC_` prefix는 클라이언트에 노출되어도 되는 값에만 사용한다 — DB 정보 등 민감한 값에 절대 사용 금지.

## 보안

- 모든 Server Action과 API Route는 인증 여부를 먼저 확인한다.
- 사용자 입력값을 SQL에 직접 삽입 금지 — 반드시 Drizzle ORM을 통해 쿼리한다.
- 민감한 데이터(비밀번호 등)는 절대 응답에 포함하지 않는다.

## 성능

- Neon Serverless 특성상 커넥션 풀링을 고려하여 DB 클라이언트를 싱글톤으로 관리한다.
- N+1 쿼리가 발생하지 않도록 쿼리 설계 시 join 또는 batch를 활용한다.
- 무거운 쿼리는 Vercel Edge Runtime 대신 Node.js Runtime에서 실행한다.
