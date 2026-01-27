---
title: "Lint 설정"
date: "2026-01-27"
tags:
    - React
    - Migration
    - ESLint
    - Lint
---

> 업무 진행도: 개발 완료 및 리뷰 진행중

# V2 기술 스택 변화

| 구분 | 기존 프로젝트 (v1) | 새 프로젝트 (v2) |
| --- | --- | --- |
| 프레임워크 | Vue 3 | **React 19** |
| ESLint 버전 | 8.x (레거시) | **9.x (Flat Config)** |
| 설정 형식 | .eslintrc.js | **eslint.config.js → 미정** |
| 구조 | Monorepo (Turbo) | **단일 앱 + FSD** |
| 스타일 가이드 | Airbnb Base | **기본 설정만 → 미정** |

# 개선이 필요한 부분 (V2 → V1)

## 기존 V1의 문제점

- `@typescript-eslint/no-explicit-any: off` any 타입 무제한 허용
- `@typescript-eslint/ban-ts-comment: off` @ts-ignore 허용
- `--max-warning=69` 경고 누적 관리 실패
- Prettier 부재 - 포매팅 일관성 약화
    - 기존 ESLint에 있던 formatting 규칙들을 deprecated 처리하고 별도 패키지로 분리함.

        `@stylistic/eslint-plugin` ← formatting 규칙 이관됨.

        ***ESLint는 코드 품질에 집중하고, 포맷팅은 Prettier 같은 전용 도구에 맡기는 것을 권장합니다.*** by ESLint team

        | 항목 | ESLint만 | ESLint + Prettier |
        | --- | --- | --- |
        | 설정 복잡도 | 높음 (규칙 수백 개) | 낮음 (역할 분리) |
        | 포맷팅 일관성 | 규칙 조합에 따라 다름 | Prettier가 강제 |
        | 속도 | 느림 | Prettier가 더 빠름 |
        | 의견 충돌 | 많음 (세부 규칙 논쟁) | 적음 (Prettier = opinionated) |
- 아키텍처 규칙 없음 - import 경계 검증 없음

## 현재 V2에서 (초기 세팅만 마친 상태) 추가해야할 부분

- commitlint, lint-staged 의존성 설치
- import/order 규칙 추가
- FSD 아키텍처 규칙 추가
- Prettier 설정

---

# ESLint & TypeScript 설정 가이드

> **목표**: ESLint v9 flat config + FSD 아키텍처 + TypeScript strict 모드를 적용한 린트 설정

---

## 1. 개요

### 기술 스택

| 항목 | 버전 |
| --- | --- |
| React | 19.x |
| TypeScript | 5.9.x |
| ESLint | 9.x (flat config) |
| Vite | 7.x |
| 아키텍처 | FSD (Feature-Sliced Design) |

---

## 2. v1 vs v2 주요 변경점

### 구조적 차이

| 항목 | v1 (기존) | v2 (현재) |
| --- | --- | --- |
| ESLint 버전 | v8 (legacy config) | v9 (flat config) |
| 설정 파일 | `.eslintrc.js` | `eslint.config.js` |
| 프레임워크 | Vue | React |
| 아키텍처 | 없음 | FSD (Feature-Sliced Design) |
| 프로젝트 구조 | Monorepo (turborepo) | 단일 앱 |

### 플러그인 비교

| 플러그인 | v1 | v2 |
| --- | --- | --- |
| `@typescript-eslint` | ✅ | ✅ |
| `eslint-plugin-vue` | ✅ | ❌ |
| `eslint-plugin-react` | ❌ | ✅ |
| `eslint-plugin-react-hooks` | ❌ | ✅ |
| `eslint-plugin-react-refresh` | ❌ | ✅ |
| `eslint-plugin-jsx-a11y` | ❌ | ✅ (접근성) |
| `eslint-plugin-import` | ✅ | ❌ |
| `eslint-plugin-import-x` | ❌ | ✅ (후속 버전) |
| `eslint-plugin-boundaries` | ❌ | ✅ (FSD 경계) |
| `eslint-plugin-check-file` | ❌ | ✅ (파일명 컨벤션) |
| `eslint-plugin-prettier` | ❌ | ✅ |
| `eslint-config-airbnb-base` | ✅ | ❌ |

---

## 3. 패키지 설치

```bash
npm install -D \
  # ESLint 핵심
  eslint @eslint/js typescript-eslint globals \
  # React 플러그인
  eslint-plugin-react eslint-plugin-react-hooks \
  eslint-plugin-react-refresh eslint-plugin-jsx-a11y \
  # Import 관리
  eslint-plugin-import-x eslint-import-resolver-typescript \
  # FSD 아키텍처 경계
  eslint-plugin-boundaries \
  # 파일명 컨벤션
  eslint-plugin-check-file \
  # Prettier 통합
  prettier eslint-plugin-prettier eslint-config-prettier \
  # Git Hooks
  husky @commitlint/cli @commitlint/config-conventional lint-staged
```

---

## 4. 파일명 컨벤션

### 규칙

| 파일 타입 | 컨벤션 | 예시 |
| --- | --- | --- |
| `.tsx` | PascalCase | `UserCard.tsx`, `LoginForm.tsx` |
| `.ts` | camelCase | `useAuth.ts`, `formatDate.ts`, `types.ts` |
| 폴더 | kebab-case | `user-profile/`, `react-query/` |

### 예외

- `index.ts` / `index.tsx` - 허용
- `main.tsx`, `App.tsx` - 엔트리포인트 파일로 규칙 제외
- `*.test.tsx`, `*.spec.ts` - 중간 확장자 무시

### ESLint 설정

```jsx
'check-file/filename-naming-convention': [
  'error',
  {
    '**/*.tsx': 'PASCAL_CASE',
    '**/*.ts': 'CAMEL_CASE',
  },
  { ignoreMiddleExtensions: true },
],
'check-file/folder-naming-convention': [
  'error',
  {
    'src/**/': 'KEBAB_CASE',
  },
],
```

---

## 5. FSD 아키텍처 핵심 규칙

### 5.1 레이어 의존성 규칙

> FSD에서 **상위 레이어는 하위 레이어만 import** 할 수 있습니다.

```
app → pages → widgets → features → entities → shared
```

| 레이어 | import 가능 | import 불가 |
| --- | --- | --- |
| app | pages, widgets, features, entities, shared | - |
| pages | widgets, features, entities, shared | app |
| widgets | features, entities, shared | app, pages |
| features | entities, shared | app, pages, widgets |
| entities | shared | app, pages, widgets, features |
| shared | shared (자기 자신) | 모든 상위 레이어 |

### 5.2 Public API 강제

> **필수 규칙**: 각 슬라이스는 `index.ts`를 통해서만 외부에 노출됩니다.

**폴더 구조 예시:**

```
src/entities/user/
├── index.ts          ← Public API (외부에서 import 가능)
├── model/
│   └── user.ts       ← 내부 구현
├── ui/
│   └── UserCard.tsx  ← 내부 구현
└── api/
    └── userApi.ts    ← 내부 구현
```

**올바른 import:**

```tsx
// ✅ 올바름 (Public API 통해)
import { User, UserCard, useUser } from '@/entities/user'
```

**잘못된 import:**

```tsx
// ❌ 금지 (내부 파일 직접 접근)
import { User } from '@/entities/user/model/user'
import { UserCard } from '@/entities/user/ui/UserCard'
```

---

## 6. TypeScript 설정

### tsconfig.app.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "jsx": "react-jsx",
    "strict": true,
    "baseUrl": ".",
    "paths": {
      "@/app/*": ["./src/app/*"],
      "@/pages/*": ["./src/pages/*"],
      "@/widgets/*": ["./src/widgets/*"],
      "@/features/*": ["./src/features/*"],
      "@/entities/*": ["./src/entities/*"],
      "@/shared/*": ["./src/shared/*"]
    }
  },
  "include": ["src"]
}
```

> **참고**: `@/*` 를 제거하고 FSD 레이어별 alias만 유지합니다. 이렇게 하면 FSD 구조를 우회하는 import를 방지할 수 있습니다.

---

## 7. Prettier 설정

### .prettierrc

```json
{
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "arrowParens": "avoid",
  "endOfLine": "lf"
}
```

### .prettierignore

```
dist
node_modules
coverage
*.min.js
package-lock.json
```

---

## 8. Git Hooks 설정 (Husky)

### 8.1 Husky 초기화

```bash
npx husky init
```

### 8.2 .husky/pre-commit

```bash
#!/bin/sh
npx lint-staged
```

### 8.3 .husky/commit-msg

```bash
#!/bin/sh
npx --no-install commitlint --edit $1
```

### 8.4 lint-staged (package.json)

```json
{
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix --max-warnings=0",
      "prettier --write"
    ],
    "*.{json,md,css}": [
      "prettier --write"
    ]
  }
}
```

### 8.5 commitlint.config.cjs

```jsx
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      ['feat', 'fix', 'docs', 'style', 'refactor', 'perf', 'test', 'build', 'ci', 'chore', 'revert'],
    ],
    'subject-case': [2, 'always', 'lower-case'],
    'header-max-length': [2, 'always', 100],
  },
}
```

---

## 9. npm 스크립트

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "lint": "eslint . --max-warnings=0",
    "lint:fix": "eslint . --fix --max-warnings=0",
    "format": "prettier --write \"src/**/*.{ts,tsx,json,css,md}\"",
    "type-check": "tsc --noEmit",
    "prepare": "husky"
  }
}
```

---

## 10. 추후 추가 고려 플러그인

해당 기능 도입 시 추가하면 됩니다.

| 플러그인 | 설명 | 도입 시점 |
| --- | --- | --- |
| `@tanstack/eslint-plugin-query` | React Query 관련 모범 사례 강제 | React Query 사용 시 |
| `eslint-plugin-testing-library` | Testing Library 모범 사례 | 테스트 코드 작성 시 |
| `eslint-plugin-vitest` | Vitest 관련 규칙 | Vitest 도입 시 |
| `eslint-plugin-storybook` | Storybook 모범 사례 | Storybook 도입 시 |
| `eslint-plugin-tailwindcss` | Tailwind 클래스 정렬 및 최적화 | Tailwind CSS 사용 시 |

### 예시: React Query 플러그인

```bash
npm install -D @tanstack/eslint-plugin-query
```

```jsx
import pluginQuery from '@tanstack/eslint-plugin-query'

export default [
  ...pluginQuery.configs['flat/recommended'],
  // 기존 설정...
]
```

### 예시: Testing Library 플러그인

```bash
npm install -D eslint-plugin-testing-library
```

```jsx
import testingLibrary from 'eslint-plugin-testing-library'

export default [
  {
    files: ['**/*.test.{ts,tsx}', '**/*.spec.{ts,tsx}'],
    ...testingLibrary.configs['flat/react'],
  },
]
```

---

## 11. 참고 자료

- [typescript-eslint](https://typescript-eslint.io/getting-started/)
- [ESLint Flat Config](https://eslint.org/docs/latest/use/configure/configuration-files)
- [eslint-plugin-boundaries](https://github.com/javierbrea/eslint-plugin-boundaries)
- [eslint-plugin-check-file](https://github.com/dukeluo/eslint-plugin-check-file)
- [Feature-Sliced Design](https://feature-sliced.design/)
- [Husky](https://typicode.github.io/husky/)
