## Vue를 이용하여 SSR 방식 구현하기

Reference: https://ko.vuejs.org/guide/scaling-up/ssr.html (Vue 공식문서)

### SSR이란?

우선, Vue.js는 클라이언트 측 앱을 빌드하기 위한 프레임워크이다. 기본적으로 브라우저에서 DOM으로 출력되는 Vue 컴포넌트를 생성하고 조작한다. 그러나 동일한 컴포넌트를 서버의 HTML 문자열로 렌더링하여 정적 마크업을 클라이언트 브라우저로 보내는 완전한 대화영 앱으로 변환 (hydrate)하는 것도 가능하다.

### SSR의 장점

- 컨텐츠에 도달하는 시간 단축
- 통합 유지모수 모델
- 더 나은 SEO

### SSR의 단점

- 개발 제약 사항
- 더 복잡한 빌드 설정 및 배포 요구 사항
- 더 많은 서버 측 부하

> 위와 같은 특징들이 있으므로 SSR을 활용하기 전 실제로 필요한지에 대해 고려해야 한다.
→ 예를 들어, 초기 로드시 추가로 수백 밀리초가 중요하지 않은 내부 대시보드를 구축하는 경우 내부 대시보드를 구축하는 경우 SSR은 과도하다. 
→ 그러나 컨텐츠에 도달하는 시간이 절대적으로 중요한 경우, SSR은 가능한 최상의 초기 로드 성능을 달성하는 데 도움이 될 수 있다.
> 

### 실습

`renderToString()` - Vue 앱 인스턴스를 사용하여 앱의 렌더링된 HTML로 해결되는 Promise를 반환한다.

(Node.js Stream API 또는 Web Streams API를 사용하여 스트리밍 렌더링도 가능 - https://ko.vuejs.org/api/ssr.html)

여기서는 앱이 전체 페이지 HTML로 마크업된 Vue SSR 코드를 Server Request Handler로 이동시킬 수 있다. (express 사용)

```jsx
import express from 'express'
import { createSSRApp } from 'vue'
import { renderToString } from 'vue/server-renderer'

const server = express()

server.get('/', (req, res) => {
  const app = createSSRApp({
    data: () => ({ count: 1 }),
    template: `<button @click="count++">{{ count }}</button>`
  })

  renderToString(app).then((html) => {
    res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <title>Vue SSR 예제</title>
      </head>
      <body>
        <div id="app">${html}</div>
      </body>
    </html>
    `)
  })
})

server.listen(3000, () => {
  console.log('ready')
})
```

위 코드로는 버튼 클릭시 숫자가 변경되지 않는데,

그 이유는 브라우저에서 Vue를 로드하지 않기 때문에 HTML은 클라이언트에서 완전히 정적이기 때문이다.

### 클라이언트 하이드레이트

위 부분을 개선하기 위해 다음과 같이 구현할 수 있다.

> 클라이언트 측 앱을 대화형으로 만들기 위해 Vue는 하이드레이트 단계를 수행해야 한다.
→ 하이드레이트하는 동안 서버에서 실행된 것과 동일한 Vue 앱을 만들고 제어해야 하는 DOM 노드에 각 컴포넌트를 일치시키고 DOM 이벤트 핸들러를 연결한다.
> 

이 때, 앱을 하이드레이트 모드로 마운트하려면 `createApp()`이 아닌 `createSSRApp()`을 사용한다.

```jsx
import { createSSRApp } from 'vue';

export function createApp() {
  return createSSRApp({
    data: () => ({ count: 1 }),
    template: `<button @click="count++" style="width: 58px; height: 30px; border: none; background-color: black; border-radius: 8px; color: #ffffff;">{{ count }}</button>`,
  });
}
```

> app.js의 파일과 그 의존성은 서버와 클라이언트 간에 공유된다. (→ 범용 코드 __ universasl code)
✓ 클라이언트 항목은 범용 코드를 가져오고 앱을 만들고 마운트를 수행한다 (아래의 코드)
> 

```jsx
import { createApp } from './app.js';

createApp().mount('#app');
```

> 그리고 서버는 request handler에서 동일한 앱 생성 로직을 사용한다.
✓ `server.use(express.static(’.’))`를 추가하여 클라이언트 파일을 제공한다.
✓ `<script type="module" src="/client.js"></script>` 를 추가하여 클라이언트 항목을 로드한다.
> 

```jsx
import express from 'express';
import { renderToString } from 'vue/server-renderer';
import { createApp } from './app.js';

const server = express();

server.get('/', (req, res) => {
  const app = createApp();

  renderToString(app).then((html) => {
    res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <title>Vue SSR Example</title>
        <script type="importmap">
          {
            "imports": {
              "vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js"
            }
          }
        </script>
        <script type="module" src="/client.js"></script>
      </head>
      <body>
        <div id="app">${html}</div>
      </body>
    </html>
    `);
  });
});

server.use(express.static('.'));

server.listen(4000, () => {
  console.log('Go!! 🚀');
});
```

node server.js를 통해 실행시킨 후 localhost:4000에 접속하면 제대로 렌더링됨을 알 수 있다.

> 이제 vue ssr을 활용할 때 vue의 속성들(ex. v-for, v-if etc…)을 제대로 활용할 수 있는지 확인해보자.
> 

```jsx
import { createSSRApp, ref } from 'vue';
import { data } from './data/index.js';

export function createApp() {
  return createSSRApp({
    setup() {
      const count = ref(1);

      return {
        count,
        data,
      };
    },
    template: `
      <div>
        <button @click="count++" style="width: 58px; height: 30px; border: none; background-color: black; border-radius: 8px; color: #ffffff;">
          {{ count }}
        </button>
        <ul>
          <li v-for="(d, i) in data" :key="item">{{ d['id'] }}</li>
        </ul>
      </div>
    `,
  });
}
```

## Vue를 이용하여 SSR 방식 구현하기

Reference: https://ko.vuejs.org/guide/scaling-up/ssr.html (Vue 공식문서)

### SSR이란?

우선, Vue.js는 클라이언트 측 앱을 빌드하기 위한 프레임워크이다. 기본적으로 브라우저에서 DOM으로 출력되는 Vue 컴포넌트를 생성하고 조작한다. 그러나 동일한 컴포넌트를 서버의 HTML 문자열로 렌더링하여 정적 마크업을 클라이언트 브라우저로 보내는 완전한 대화영 앱으로 변환 (hydrate)하는 것도 가능하다.

### SSR의 장점

- 컨텐츠에 도달하는 시간 단축
- 통합 유지모수 모델
- 더 나은 SEO

### SSR의 단점

- 개발 제약 사항
- 더 복잡한 빌드 설정 및 배포 요구 사항
- 더 많은 서버 측 부하

> 위와 같은 특징들이 있으므로 SSR을 활용하기 전 실제로 필요한지에 대해 고려해야 한다.
→ 예를 들어, 초기 로드시 추가로 수백 밀리초가 중요하지 않은 내부 대시보드를 구축하는 경우 내부 대시보드를 구축하는 경우 SSR은 과도하다. 
→ 그러나 컨텐츠에 도달하는 시간이 절대적으로 중요한 경우, SSR은 가능한 최상의 초기 로드 성능을 달성하는 데 도움이 될 수 있다.
> 

### 실습

`renderToString()` - Vue 앱 인스턴스를 사용하여 앱의 렌더링된 HTML로 해결되는 Promise를 반환한다.

(Node.js Stream API 또는 Web Streams API를 사용하여 스트리밍 렌더링도 가능 - https://ko.vuejs.org/api/ssr.html)

여기서는 앱이 전체 페이지 HTML로 마크업된 Vue SSR 코드를 Server Request Handler로 이동시킬 수 있다. (express 사용)

```jsx
import express from 'express'
import { createSSRApp } from 'vue'
import { renderToString } from 'vue/server-renderer'

const server = express()

server.get('/', (req, res) => {
  const app = createSSRApp({
    data: () => ({ count: 1 }),
    template: `<button @click="count++">{{ count }}</button>`
  })

  renderToString(app).then((html) => {
    res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <title>Vue SSR 예제</title>
      </head>
      <body>
        <div id="app">${html}</div>
      </body>
    </html>
    `)
  })
})

server.listen(3000, () => {
  console.log('ready')
})
```

위 코드로는 버튼 클릭시 숫자가 변경되지 않는데,

그 이유는 브라우저에서 Vue를 로드하지 않기 때문에 HTML은 클라이언트에서 완전히 정적이기 때문이다.

### 클라이언트 하이드레이트

위 부분을 개선하기 위해 다음과 같이 구현할 수 있다.

> 클라이언트 측 앱을 대화형으로 만들기 위해 Vue는 하이드레이트 단계를 수행해야 한다.
→ 하이드레이트하는 동안 서버에서 실행된 것과 동일한 Vue 앱을 만들고 제어해야 하는 DOM 노드에 각 컴포넌트를 일치시키고 DOM 이벤트 핸들러를 연결한다.
> 

이 때, 앱을 하이드레이트 모드로 마운트하려면 `createApp()`이 아닌 `createSSRApp()`을 사용한다.

```jsx
import { createSSRApp } from 'vue';

export function createApp() {
  return createSSRApp({
    data: () => ({ count: 1 }),
    template: `<button @click="count++" style="width: 58px; height: 30px; border: none; background-color: black; border-radius: 8px; color: #ffffff;">{{ count }}</button>`,
  });
}
```

> app.js의 파일과 그 의존성은 서버와 클라이언트 간에 공유된다. (→ 범용 코드 __ universasl code)
✓ 클라이언트 항목은 범용 코드를 가져오고 앱을 만들고 마운트를 수행한다 (아래의 코드)
> 

```jsx
import { createApp } from './app.js';

createApp().mount('#app');
```

> 그리고 서버는 request handler에서 동일한 앱 생성 로직을 사용한다.
✓ `server.use(express.static(’.’))`를 추가하여 클라이언트 파일을 제공한다.
✓ `<script type="module" src="/client.js"></script>` 를 추가하여 클라이언트 항목을 로드한다.
> 

```jsx
import express from 'express';
import { renderToString } from 'vue/server-renderer';
import { createApp } from './app.js';

const server = express();

server.get('/', (req, res) => {
  const app = createApp();

  renderToString(app).then((html) => {
    res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <title>Vue SSR Example</title>
        <script type="importmap">
          {
            "imports": {
              "vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js"
            }
          }
        </script>
        <script type="module" src="/client.js"></script>
      </head>
      <body>
        <div id="app">${html}</div>
      </body>
    </html>
    `);
  });
});

server.use(express.static('.'));

server.listen(4000, () => {
  console.log('Go!! 🚀');
});
```

node server.js를 통해 실행시킨 후 localhost:4000에 접속하면 제대로 렌더링됨을 알 수 있다.

> 이제 vue ssr을 활용할 때 vue의 속성들(ex. v-for, v-if etc…)을 제대로 활용할 수 있는지 확인해보자.
> 

```jsx
import { createSSRApp, ref } from 'vue';
import { data } from './data/index.js';

export function createApp() {
  return createSSRApp({
    setup() {
      const count = ref(1);

      return {
        count,
        data,
      };
    },
    template: `
      <div>
        <button @click="count++" style="width: 58px; height: 30px; border: none; background-color: black; border-radius: 8px; color: #ffffff;">
          {{ count }}
        </button>
        <ul>
          <li v-for="(d, i) in data" :key="item">{{ d['id'] }}</li>
        </ul>
      </div>
    `,
  });
}
```

![image](https://github.com/dltmddus1998/dltmddus1998.github.io/assets/73332608/7be9aa68-eec1-4017-8349-ccc86e805175)

💡 **별다른 문제없이 잘 실행된다!**
