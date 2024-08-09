---
title: "WebStorm 사용 후기"
tags: 
  - IDE
  - WebStorm
  - vue
date: "2024-08-08"
thumbnail: "/assets/img/thumbnail/nightgardenflower.jpg"
bookmark: true
---

# WebStorm 사용 후기

> 우선 WebStorm은 유료 IDE 중 하나인데, 그동안 Visual Studio Code로만 개발해왔던 나에겐 생소한 아이였다.
> 

> 회사에서 WebStorm을 쓰게돼서 처음으로 깔아봤는데 진짜 까다로운 아이다. 

> 우선, Visual Studio Code에서는 prettier와 eslint를 설정하려면 전역 설정 파일인 setting.json에서 전역으로 설정해도 되고, 특정 작업 영역에서만 적용시키려면 해당 워크스페이스에 .eslintrc.js와 .prettierrc 파일을 통해 쉽게 설정이 가능하다.

> 근데, WebStorm은 설정파일에서 아래와 같이 따로 설정해줘야 위에서 말한 .eslintrc.js 파일이 적용될 수 있다. (우리의 경우, eslint파일 내에 prettier 설정도 함께해주었다. )

![image](https://github.com/user-attachments/assets/0be61c72-0347-4ae8-9758-8af0932dc428)

> eslint와 prettier 설정하는데만 많은 시간이 들었는데, 이건 WebStorm 때문은 아니고 그냥 eslint 설정이 제대로 안돼서 계속 오류가 났다. extends 부분에 제대로 된 플러그인들을 입력해줘야 하는데 그렇지 못해서 그랬던 것 같다. 최종적으로 모든 파일에 에러 없이 eslint가 적용된 설정파일은 아래와 같다. (참고로 vue2 관련 eslint 설정파일이다)

```jsx
module.exports = {
  root: true,
  env: {
    node: true,
  },
  extends: [
    'eslint:recommended',
    '@vue/eslint-config-typescript',
    '@vue/eslint-config-prettier',
    'plugin:vue/recommended',

  ],
  rules: {
    'prettier/prettier': [
      'error',
      {
        singleQuote: true,
        semi: true,
        useTabs: false,
        tabWidth: 2,
        trailingComma: 'all',
        printWidth: 80,
        bracketSpacing: true,
        arrowParens: 'avoid',
      },
    ],
    'vue/no-unused-vars': 'warn',
    'no-console': 'warn',
    'vue/multi-word-component-names': 'off',
    "no-unused-vars": "warn"
  },
};
```
