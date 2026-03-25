# 프론트엔드 규칙

## 디자인 패턴: 아토믹 디자인

모든 프론트엔드 컴포넌트는 아토믹 디자인 패턴을 따른다.

### 디렉토리 구조

```
src/
  components/
    atoms/       # 가장 작은 단위 — Button, Input, Label, Icon 등
    molecules/   # atoms 조합 — SearchBar, FormField, Card 등
    organisms/   # molecules/atoms 조합 — Header, Sidebar, ProductList 등
    templates/   # 레이아웃 구조 — 실제 데이터 없이 구조만 정의
    pages/       # templates + 실제 데이터 — 최종 화면
```

### 규칙

- **atoms**: 외부 의존성 없이 단독으로 동작하는 최소 단위. props만으로 제어.
- **molecules**: atoms 2개 이상 조합. 단일 책임 원칙 유지.
- **organisms**: 독립적인 섹션. UI 상태(열림/닫힘 등)는 가질 수 있으나 비즈니스 로직은 hooks/로 분리.
- **templates**: 레이아웃과 구조 정의. 데이터는 props로만 받음.
- **pages**: 라우팅 단위. 데이터 패칭 및 상태 관리는 여기서.

### 컴포넌트 작성 시

- 새 컴포넌트를 만들기 전에 어느 계층에 속하는지 먼저 결정한다.
- 상위 계층 컴포넌트는 하위 계층만 import할 수 있다 (pages → templates → organisms → molecules → atoms).
- 같은 계층 간 import는 피한다.

### 레이어 내부 폴더 구조

각 레이어 내부는 UI 역할 또는 도메인에 따라 카테고리 폴더로 그룹화한다.

```
atoms/
  form/        # Input, Checkbox, Select 등
  feedback/    # Spinner, Toast, Badge 등
  typography/  # Heading, Text, Label 등
molecules/
  auth/        # LoginForm, SignupForm 등
  navigation/  # Breadcrumb, Tabs 등
```

- 카테고리는 고정이 아닌 참고용이며, 상황에 따라 자유롭게 추가한다.
- 기존 카테고리에 맞지 않는 컴포넌트는 새 카테고리 폴더를 생성한다.
- 카테고리 폴더 내에 추가 depth는 만들지 않는다.

## 프로젝트 구조

```
src/
  components/   # UI만 — 비즈니스 로직 없음
  hooks/        # 비즈니스 로직을 담은 커스텀 훅
  services/     # API 호출 및 외부 연동
  types/        # 모든 TypeScript 타입/인터페이스 선언
  utils/        # 순수 헬퍼 함수
  configs/      # 앱 설정 (env, feature flags, third-party 초기화)
  constants/    # 하드코딩 값, enum, 정적 룩업 데이터 (로직 없음)
  stores/       # 전역 상태 관리 (zustand, jotai, redux 등)
  contexts/     # React Context provider/consumer
  lib/          # 서드파티 라이브러리 래퍼 및 어댑터
  assets/       # 정적 파일 (이미지, 폰트, 아이콘, SVG)
  styles/       # 글로벌 스타일, 테마, 디자인 토큰, CSS 변수
  schemas/      # 유효성 검사 스키마 (zod, yup 등)
  mocks/        # 목 데이터 및 MSW 핸들러 (개발/테스트용)
  i18n/         # 다국어 설정 및 번역 파일
  middlewares/  # 요청/응답 인터셉터, 인증 가드
```

- 타입은 반드시 `types/` 또는 `*.types.d.ts` 파일에 선언 — 컴포넌트나 로직 파일에 인라인 선언 금지.

## 컴포넌트 규칙

- 컴포넌트는 순수 UI만 — 직접 API 호출, 비즈니스 로직 작성 금지.
- 비즈니스 로직은 커스텀 훅(`hooks/`) 또는 서비스(`services/`)로 분리.
- Props 타입은 반드시 `*.types.d.ts` 파일에 선언한다.

## 라이브러리/프레임워크 사용

- 기능을 사용하기 전에 공식 문서를 먼저 확인한다.
- 공식 권장 패턴을 따르며, 문서화된 API에서 벗어나는 우회 방법은 피한다.

## 네이밍 컨벤션

- 컴포넌트 파일명은 PascalCase — `Button.tsx`, `SearchBar.tsx`
- 폴더명은 kebab-case — `atoms/search-bar/SearchBar.tsx`
- 훅 파일명은 camelCase + `use` prefix — `useModal.ts`
- 유틸/헬퍼 파일명은 camelCase — `formatDate.ts`

## Export 방식

### 컴포넌트 파일

각 컴포넌트는 `export default`로 내보낸다.

```tsx
// Button.tsx
const Button = ({ children }: ButtonProps) => {
  return <button>{children}</button>;
};

export default Button;
```

### index 파일 (barrel export)

각 계층 폴더에는 `index.ts`를 두어 내부 컴포넌트를 모아서 named export한다.

```ts
// atoms/index.ts
export { default as Button } from "./Button";
export { default as Input } from "./Input";
export { default as Label } from "./Label";
```

### import 시

컴포넌트를 사용할 때는 개별 파일이 아닌 계층 index에서 import한다.

```tsx
// Good
import { Button, Input } from "@/components/atoms";

// Bad
import Button from "@/components/atoms/Button";
```

## 코드 스타일

- 배열 순회는 `for`/`while` 대신 `map`, `filter`, `reduce`, `forEach` 등 배열 메서드를 사용한다.
- 단, 성능이 critical하거나 `break`/`continue`가 필요한 경우는 예외로 한다.
- `reduce`는 가독성이 명확할 때만 사용하고, 복잡해지면 `forEach`로 대체한다.
- 조기 반환(early return)을 적극 활용해 중첩을 줄인다.
- 삼항 연산자는 단순한 경우에만 — 중첩 삼항 금지.

## 상태 관리

- 서버 상태와 클라이언트 상태를 분리한다.
  - 서버 상태: React Query / SWR 등 사용.
  - 클라이언트 전역 상태: `stores/` 사용.
- `useState`는 컴포넌트 로컬 UI 상태에만 사용한다.

## 에러 처리

- API 호출 시 반드시 에러 처리를 포함한다.
- Error Boundary를 적절한 단위로 설정한다.

## 성능

- `useEffect` 의존성 배열을 정확히 명시한다 — 빈 배열 남용 금지.
- `useCallback`, `useMemo`는 실제 성능 문제가 있을 때만 사용한다 — 남용 금지.

## 접근성

- 이미지에는 반드시 `alt` 속성을 명시한다.
- 버튼/링크에는 의미있는 텍스트 또는 `aria-label`을 제공한다.
- 키보드 접근이 가능하도록 `tabIndex`, `onKeyDown` 등을 고려한다.

## 테스트

- 컴포넌트 테스트는 구현이 아닌 동작 기준으로 작성한다.
- 비즈니스 로직(hooks, utils)은 단위 테스트를 작성한다.
- 테스트 파일은 대상 파일과 같은 위치에 `*.test.ts(x)` 로 둔다.

## 환경변수

- 환경변수는 반드시 `configs/`에서 중앙 관리한다.
- 컴포넌트나 서비스에서 `process.env`를 직접 참조하지 않는다.
- 환경변수 키는 `NEXT_PUBLIC_` prefix 규칙을 준수한다.

## 주석

- "무엇"이 아닌 "왜"를 설명하는 주석만 작성한다.
- 당연한 코드에 주석 금지 — 코드 자체가 읽히도록 작성한다.
- TODO 주석은 반드시 이슈 번호를 함께 명시한다 (`// TODO: #123`).
