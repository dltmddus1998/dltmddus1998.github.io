---
title: "Lint 설정"
date: "2026-01-27"
bookmark: true
tags:
    - React
    - Migration
    - ESLint
    - Lint
---

# CloudOps Console v2 - ESLint 설정 구축

## TL;DR

> **핵심 변경사항**: ESLint 9 flat config 도입, airbnb 제거, FSD 아키텍처 경계 규칙 적용

- ✅ ESLint 9.x + flat config 기반 설계
- ✅ `eslint-plugin-import-x`로 마이그레이션
- ✅ `eslint-plugin-boundaries`로 FSD 레이어 의존성 강제
- ✅ 파일명 컨벤션 자동화

---

## 목차

1. [배경](#배경)
2. [주요 변경 사항](#주요-변경-사항)
3. [ESM 환경 이슈](#esm-환경-이슈)
4. [기타 설정](#기타-설정)
5. [v1 대비 개선점](#v1-대비-개선점)
6. [향후 추가 예정](#향후-추가-예정)

---

## 배경

v1에서 ESLint 9.x와 `eslint-config-airbnb` 호환성 문제로 ESLint 8.x로 다운그레이드했던 경험이 있었다. 이번 v2에서는 처음부터 **ESLint 9 flat config 기반**으로 설계해서 이런 문제를 원천 차단하고 싶었다.

---

## 주요 변경 사항

### 1. ESLint 9 Flat Config 도입

기존 `.eslintrc.js` 방식 대신 `eslint.config.js` flat config 방식을 채택했다.

```js
// eslint.config.js
export default defineConfig([
  globalIgnores(['dist', 'node_modules']),
  {
    files: ['**/*.{ts,tsx}'],
    extends: [js.configs.recommended, tseslint.configs.strictTypeChecked],
  },
])
```

> **💡 flat config를 선택한 이유**
> - 배열 순서가 곧 우선순위라 **직관적**
> - `import`로 의존성이 **명시적**으로 보임
> - `overrides` 대신 배열에 객체 추가하면 돼서 **단순함**
> - **ESM 네이티브** 지원

---

### 2. airbnb 플러그인 제거

v1에서는 `eslint-config-airbnb-typescript`를 썼는데, 이게 **2025년 5월에 archive**되면서 더 이상 유지보수가 안 된다. v2에서는 필요한 규칙들을 직접 설정하는 방식으로 갔다.

> ⚠️ **주의**: [eslint-config-airbnb-typescript](https://github.com/iamturns/eslint-config-airbnb-typescript)는 더 이상 유지보수되지 않음

```js
// 직접 설정한 TypeScript 규칙
rules: {
  '@typescript-eslint/consistent-type-imports': ['error', { prefer: 'type-imports' }],
  '@typescript-eslint/naming-convention': [
    'error',
    { selector: 'interface', format: ['PascalCase'] },
  ],
}
```

---

### 3. eslint-plugin-import → eslint-plugin-import-x

[`eslint-plugin-import`](https://github.com/import-js/eslint-plugin-import)가 ESLint 9 flat config를 제대로 지원하지 않아서 fork 버전인 [`eslint-plugin-import-x`](https://github.com/un-ts/eslint-plugin-import-x)로 교체했다.

| 비교 항목 | `eslint-plugin-import` | `eslint-plugin-import-x` |
|----------|------------------------|--------------------------|
| ESLint 9 지원 | ❌ 미지원 | ✅ 지원 |
| Flat Config | ❌ 미지원 | ✅ 지원 |
| 성능 | 보통 | ⚡ 개선됨 |

---

### 4. FSD 아키텍처 경계 규칙

[`eslint-plugin-boundaries`](https://github.com/javierbrea/eslint-plugin-boundaries)로 FSD 레이어 간 의존성 규칙을 강제했다.

<details>
<summary><strong>📦 boundaries 규칙 전체 코드 보기</strong></summary>

```js
'boundaries/element-types': ['error', {
  default: 'disallow',
  rules: [
    { from: 'app', allow: ['pages', 'widgets', 'features', 'entities', 'shared'] },
    { from: 'pages', allow: ['widgets', 'features', 'entities', 'shared'] },
    { from: 'widgets', allow: ['features', 'entities', 'shared'] },
    { from: 'features', allow: ['entities', 'shared'] },
    { from: 'entities', allow: ['shared'] },
    { from: 'shared', allow: ['shared'] },
  ],
}]
```

</details>

이 규칙의 핵심은 **상위 레이어 → 하위 레이어**로만 import 가능하게 제한하는 것이다.

```
app → pages → widgets → features → entities → shared
 ↓      ↓        ↓          ↓          ↓
 ✅     ✅       ✅         ✅         ✅
 ←      ←        ←          ←          ←
 ❌     ❌       ❌         ❌         ❌
```

---

### 5. Public API 패턴 강제

```js
'import-x/no-internal-modules': ['error', {
  allow: ['**/index.{ts,tsx,js}', '**/node_modules/**', '**/*.{svg,css}'],
}]
```

> **🎯 목적**: 각 모듈의 `index.ts`를 통해서만 import하도록 강제해서 **내부 구현을 캡슐화**

```ts
// ✅ Good - Public API 사용
import { Button } from '@/shared/ui'

// ❌ Bad - 내부 모듈 직접 접근
import { Button } from '@/shared/ui/Button/Button'
```

---

### 6. 파일명 컨벤션

[`eslint-plugin-check-file`](https://github.com/DukeLuo/eslint-plugin-check-file)로 파일명 규칙을 강제했다.

| 확장자 | 컨벤션 | 예시 |
|:------:|:------:|:----:|
| `.tsx` | **PascalCase** | `UserProfile.tsx` |
| `.ts` | **camelCase** | `useAuth.ts` |
| 폴더 | **kebab-case** | `user-profile/` |

<details>
<summary><strong>📦 설정 코드 보기</strong></summary>

```js
'check-file/filename-naming-convention': ['error', {
  '**/*.tsx': 'PASCAL_CASE',
  '**/*.ts': 'CAMEL_CASE',
}],
'check-file/folder-naming-convention': ['error', {
  'src/**': 'KEBAB_CASE',
}]
```

</details>

---

### 7. Import 정렬

FSD 구조를 반영한 import 정렬 규칙을 적용했다.

<details>
<summary><strong>📦 import 정렬 규칙 전체 코드 보기</strong></summary>

```js
'import-x/order': ['error', {
  groups: ['builtin', 'external', 'internal', ['parent', 'sibling', 'index'], 'type'],
  pathGroups: [
    { pattern: 'react', group: 'builtin', position: 'before' },
    { pattern: '@/app/**', group: 'internal', position: 'before' },
    { pattern: '@/pages/**', group: 'internal', position: 'before' },
    { pattern: '@/widgets/**', group: 'internal', position: 'before' },
    { pattern: '@/features/**', group: 'internal', position: 'before' },
    { pattern: '@/entities/**', group: 'internal', position: 'before' },
    { pattern: '@/shared/**', group: 'internal', position: 'before' },
  ],
  'newlines-between': 'always',
}]
```

</details>

**정렬 결과 예시:**

```ts
// 1. builtin
import React from 'react'

// 2. external
import { QueryClient } from '@tanstack/react-query'

// 3. internal (FSD 순서)
import { AppProvider } from '@/app/providers'
import { HomePage } from '@/pages/home'
import { useAuth } from '@/features/auth'
import { Button } from '@/shared/ui'

// 4. types
import type { User } from '@/entities/user'
```

---

## ESM 환경 이슈

`package.json`에 `"type": "module"`이 설정되어 있어서 ESM으로 동작한다. 이 경우 **`__dirname`을 사용할 수 없다**.

> ⚠️ **문제**: v1에서는 CJS라 `__dirname`을 쓸 수 있었지만, v2는 ESM이라 다른 방식 필요

**수정 전** ❌
```js
'@': path.resolve(__dirname, './src')
```

**수정 후** ✅
```js
import { fileURLToPath, URL } from 'node:url'

'@': fileURLToPath(new URL('./src', import.meta.url))
```

| 모드 | `"type": "module"` | `__dirname` 사용 |
|:----:|:-----------------:|:---------------:|
| **CJS** | 없음 | ✅ 가능 |
| **ESM** | 있음 | ❌ 불가 |

> **💡 참고**: ESM이 현재 표준이고, 트리 쉐이킹 같은 최적화에도 유리하니까 `__dirname` 쓰려고 CJS로 돌아가는 건 비추천

---

## 기타 설정

### Husky + lint-staged

커밋 전 자동으로 lint와 포맷팅을 실행한다.

```sh
# .husky/pre-commit
#!/bin/sh
npx lint-staged
```

```json
// package.json
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"]
  }
}
```

### commitlint

[Conventional Commits](https://www.conventionalcommits.org/) 스펙을 강제한다.

```js
// commitlint.config.cjs
module.exports = {
  extends: ['@commitlint/config-conventional'],
}
```

**커밋 메시지 예시:**
```
feat: 사용자 인증 기능 추가
fix: 로그인 버튼 클릭 이벤트 수정
docs: README 업데이트
```

---

## v1 대비 개선점

| 항목 | v1 | v2 |
|:-----|:---|:---|
| **ESLint 버전** | 8.x *(호환성 문제로 다운그레이드)* | 9.x flat config |
| **설정 방식** | `.eslintrc.js` + airbnb | `eslint.config.js` 직접 설정 |
| **import 플러그인** | eslint-plugin-import | **eslint-plugin-import-x** |
| **아키텍처 강제** | ❌ 없음 | ✅ FSD boundaries 규칙 |
| **Public API** | ❌ 없음 | ✅ no-internal-modules |
| **파일명 컨벤션** | ❌ 없음 | ✅ check-file |

---

## 향후 추가 예정

테스트 환경 구축할 때 아래 플러그인들 추가할 예정:

- [`eslint-plugin-testing-library`](https://github.com/testing-library/eslint-plugin-testing-library) - Testing Library 베스트 프랙티스
- [`eslint-plugin-vitest`](https://github.com/vitest-dev/eslint-plugin-vitest) - Vitest 규칙
- [`eslint-plugin-storybook`](https://github.com/storybookjs/eslint-plugin-storybook) - Storybook 베스트 프랙티스

---

## 참고 자료

- [ESLint Flat Config 공식 문서](https://eslint.org/docs/latest/use/configure/configuration-files)
- [Feature-Sliced Design](https://feature-sliced.design/)
- [typescript-eslint](https://typescript-eslint.io/)