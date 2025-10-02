# @gravity-ui/app-layout &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/app-layout)](https://www.npmjs.com/package/@gravity-ui/app-layout) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/app-layout/.github/workflows/ci.yml?label=CI&logo=github)](https://github.com/gravity-ui/app-layout/actions/workflows/ci.yml?query=branch:main)

## 설치

```shell
npm install --save-dev @gravity-ui/app-layout
```

## 사용법

`express`와 함께:

```js
import express from 'express';
import {createRenderFunction} from '@gravity-ui/app-layout';

const app = express();

const renderLayout = createRenderFunction();

app.get('/', function (req, res) {
  res.send(
    renderLayout({
      // RenderParams
      title: 'Home page',
      bodyContent: {
        root: 'Hello world!',
      },
    }),
  );
});

app.listen(3000);
```

여기서

```typescript
interface RenderParams<Data, Plugins> {
  // 페이지에 window.__DATA__로 설정될 JSON 호환 데이터
  data?: Data;
  // 파비콘
  icon?: Icon;
  // 적절한 태그에 설정될 nonce
  nonce?: string;

  // 공통 옵션
  // 페이지 제목
  title: string;
  // 페이지 언어, html 태그에 설정됨
  lang?: string;
  isMobile?: boolean;

  // html 속성
  htmlAttributes?: string;
  // header 태그 콘텐츠
  // 메타 태그
  meta?: Meta[];
  // 링크 태그
  links?: Link[];
  // 스크립트 태그
  scripts?: Script[];
  // 스타일 태그
  styleSheets?: Stylesheet[];
  // 인라인 코드가 포함된 스크립트 태그
  inlineScripts?: string[];
  // 인라인 스타일이 포함된 스타일 태그
  inlineStyleSheets?: string[];

  // body 태그 콘텐츠
  bodyContent?: {
    // body 태그의 클래스 이름
    className?: string;
    // body 속성
    attributes?: string;
    // id가 root인 div 태그 이전의 body 콘텐츠
    beforeRoot?: string;
    // id가 root인 div 태그의 innerHtml 콘텐츠
    root?: string;
    // id가 root인 div 태그 이후의 body 콘텐츠
    afterRoot?: string;
  };
  // 플러그인 옵션
  pluginsOptions?: Partial<PluginsOptions<Plugins>>;
}
```

### 메타

`meta` 태그를 설명합니다:

```typescript
interface Meta {
  name: string;
  content: string;
}
```

예시:

```js
const meta = [
  {name: 'description', content: 'some text'},
  {name: 'robots', content: 'noindex'},
  {name: 'og:title', content: 'Some title'},
];
```

다음과 같이 렌더링됩니다:

```html
<meta name="description" content="some text" />
<meta name="robots" content="noindex" />
<meta property="og:title" content="Some title" />
```

### 아이콘

페이지 파비콘을 설명합니다:

```typescript
interface Icon {
  type?: string;
  sizes?: string;
  href?: string;
}
```

기본값은 다음과 같습니다:

```js
const icon = {
  type: 'image/png',
  sizes: '16x16',
  href: '/favicon.png',
};
```

### 링크

`link` 태그를 설명합니다:

```typescript
interface Link {
  as?: string;
  href: string;
  rel?: string;
  type?: string;
  sizes?: string;
  title?: HTMLLinkElement['title'];
  crossOrigin?: '' | 'anonymous' | 'use-credentials';
}
```

예시:

```js
const link = {
  href: 'myFont.woff2',
  rel: 'preload',
  as: 'font',
  type: 'font/woff2',
  crossOrigin: 'anonymous',
};
```

다음과 같이 렌더링됩니다:

```html
<link href="myFont.woff2" rel="preload" as="font" type="font/woff2" crossorigin="anonymous" />
```

### 스크립트

프리로드가 포함된 스크립트 링크를 설명합니다:

```typescript
interface Script {
  src: string;
  defer?: boolean;
  async?: boolean;
  crossOrigin?: '' | 'anonymous' | 'use-credentials';
  type?: 'importmap' | 'module' | string;
}
```

예시:

```js
const script = {
  src: 'url/to/script',
  defer: true,
  async: false,
  crossOrigin: 'anonymous',
};
```

다음과 같이 렌더링됩니다:

```html
<link href="url/to/script" rel="preload" as="script" crossorigin="anonymous" />

<script src="url/to/script" defer="true" async="false" crossorigin="anonymous" nonce="..."></script>
```

#### 스타일시트

스타일 링크를 설명합니다:

```typescript
interface Stylesheet {
  href: string;
}
```

예시:

```js
const styleSheet = {
  href: 'url/to/stylesheet',
};
```

다음과 같이 렌더링됩니다:

```html
<link href="url/to/stylesheet" rel="stylesheet" />
```

## 플러그인

렌더 함수는 플러그인으로 확장할 수 있습니다. 플러그인은 사용자 정의 렌더 콘텐츠를 재작성할 수 있습니다.
플러그인은 `name`과 `apply` 속성을 가진 객체입니다:

```typescript
interface Plugin<Options = any, Name = string> {
  name: Name;
  apply: (params: {
    options: Options | undefined; // `renderLayout` 함수의 `pluginsOptions` 매개변수를 통해 전달됨.
    commonOptions: CommonOptions;
    renderContent: RenderContent;
    /** @deprecated use `renderContent.helpers` instead */
    utils: RenderHelpers;
  }) => void;
}

interface CommonOptions {
  name: string;
  title: string;
  lang?: string;
  isMobile?: boolean;
}

export interface HeadContent {
  scripts: Script[];
  helpers: RenderHelpers;
  links: Link[];
  meta: Meta[];
  styleSheets: Stylesheet[];
  inlineStyleSheets: string[];
  inlineScripts: string[];
  title: string;
}

export interface BodyContent {
  attributes: Attributes;
  beforeRoot: string[];
  root?: string;
  afterRoot: string[];
}

export interface RenderContent extends HeadContent {
  htmlAttributes: Attributes;
  bodyContent: BodyContent;
}

export interface RenderHelpers {
  renderScript(script: Script): string;
  renderInlineScript(content: string): string;
  renderStyle(style: Stylesheet): string;
  renderInlineStyle(content: string): string;
  renderMeta(meta: Meta): string;
  renderLink(link: Link): string;
  attrs(obj: Attributes): string;
}
```

이 패키지에는 몇 가지 플러그인이 포함되어 있습니다:

### Google Analytics

페이지에 Google Analytics 카운터를 추가합니다.

사용법:

```js
import {createRenderFunction, createGoogleAnalyticsPlugin} from '@gravity-ui/app-layout';

const renderLayout = createRenderFunction([createGoogleAnalyticsPlugin()]);

```

```js
app.get((req, res) => {
  res.send(
    renderLayout({
      title: 'Home page',
      pluginsOptions: {
        googleAnalytics: {
          useBeaconTransport: true, // enables use of navigator.sendBeacon
          counter: {
            id: 'some id',
          },
        },
      },
    }),
  );
});
```

플러그인 옵션:

```typescript
interface GoogleAnalyticsCounter {
  id: string;
}

interface GoogleAnalyticsOptions {
  useBeaconTransport?: boolean;
  counter: GoogleAnalyticsCounter;
}
```

### Yandex Metrika

페이지에 Yandex 메트릭 카운터를 추가합니다.

사용법:

```js
import {createRenderFunction, createYandexMetrikaPlugin} from '@gravity-ui/app-layout';

const renderLayout = createRenderFunction([createYandexMetrikaPlugin()]);

app.get((req, res) => {
  res.send(
    renderLayout({
      title: 'Home page',
      pluginsOptions: {
        yandexMetrika: {
          counter: {
            id: 123123123,
            defer: true,
            clickmap: true,
            trackLinks: true,
            accurateTrackBounce: true,
          },
        },
      },
    }),
  );
});
```

플러그인 옵션:

```typescript
export type UserParams = {
  [x: string]: boolean | string | number | null | UserParams;
};

export interface MetrikaCounter {
  id: number;
  defer: boolean;
  clickmap: boolean;
  trackLinks: boolean;
  accurateTrackBounce: boolean | number;
  webvisor?: boolean;
  nonce?: string;
  encryptedExperiments?: string;
  triggerEvent?: boolean;
  trackHash?: boolean;
  ecommerce?: boolean | string;
  type?: number;
  userParams?: UserParams;
}

export type MetrikaOptions = {
  src?: string;
  counter: MetrikaCounter | MetrikaCounter[];
};
```

### Layout

Webpack 에셋 매니페스트 파일에서 스크립트와 스타일을 추가합니다.

사용법:

```js
import {createRenderFunction, createLayoutPlugin} from '@gravity-ui/app-layout';

const renderLayout = createRenderFunction([
  createLayoutPlugin({manifest: 'path/to/assets-manifest.json', publicPath: '/build/'}),
]);

app.get((req, res) => {
  res.send(
    renderLayout({
      title: 'Home page',
      pluginsOptions: {
        layout: {
          name: 'home',
        },
      },
    }),
  );
});
```

플러그인 옵션:

```typescript
export interface LayoutOptions {
  name: string;
  prefix?: string;
}
```

### @gravity-ui/uikit

body 속성을 추가합니다.

사용법:

```js
import {createRenderFunction, createUikitPlugin} from '@gravity-ui/app-layout';

const renderLayout = createRenderFunction([createUikitPlugin()]);

app.get((req, res) => {
  res.send(
    renderLayout({
      title: 'Home page',
      pluginsOptions: {
        uikit: {
          theme: 'dark',
          direction: 'ltr',
        },
      },
    }),
  );
});
```

플러그인 옵션:

```typescript
interface UikitPluginOptions {
  theme: string;
  direction?: 'ltr' | 'rtl';
}
```

### Remote Versions

페이지에 마이크로프론트엔드 버전 정보를 추가합니다.

이 플러그인은 제공된 마이크로프론트엔드 버전을 포함하는 전역 `window.__REMOTE_VERSIONS__` 객체를 생성하며, 모듈 페더레이션이나 유사한 마이크로프론트엔드 아키텍처에서 원격 모듈의 버전을 결정하는 데 사용할 수 있습니다.

[App Builder](https://github.com/gravity-ui/app-builder?tab=readme-ov-file#module-federation)와 함께 사용하며 `moduleFederation.remotesRuntimeVersioning` 옵션을 통해 해당 버전의 원격 모듈을 자동으로 로드할 수 있습니다.

사용법:

```js
import {createRenderFunction, createRemoteVersionsPlugin} from '@gravity-ui/app-layout';

const renderLayout = createRenderFunction([createRemoteVersionsPlugin()]);

app.get((req, res) => {
  res.send(
    renderLayout({
      title: 'Home page',
      pluginsOptions: {
        remoteVersions: {
          header: '1.2.3',
          footer: '2.1.0',
          sidebar: '0.5.1',
        },
      },
    }),
  );
});
```

플러그인 옵션:

```typescript
type RemoteVersionsPluginOptions = Record<string, string>;
```

### Helpers

모든 플러그인을 생성하는 헬퍼가 있습니다:

```js
import {createMiddleware, createDefaultPlugins} from '@gravity-ui/app-layout';

const renderLayout = createRenderFunction(
    createDefaultPlugins({layout: {manifest: 'path/to/assets-manifest.json'}})
);

app.get((req, res) => {
    res.send(renderLayout({
        title: 'Home page',
        pluginsOptions: {
            layout: {
                name: 'home'
            },
            googleAnalytics: {
                counter: {...}
            },
            yandexMetrika: {
                counter: {...}
            },
        },
    }));
})
```

## Alternative usage

HTML 스트리밍을 통해 `generateRenderContent`, `renderHeadContent`, `renderBodyContent` 등의 부분 렌더러를 사용하는 방법:

```js
import express from 'express';
import htmlescape from 'htmlescape';
import {
  generateRenderContent,
  renderHeadContent,
  renderBodyContent,
  createDefaultPlugins,
} from '@gravity-ui/app-layout';

const app = express();

app.get('/', async function (req, res) {
  res.writeHead(200, {
    'Content-Type': 'text/html',
    'Transfer-Encoding': 'chunked',
  });

  const plugins = createDefaultPlugins({layout: {manifest: 'path/to/assets-manifest.json'}});

  const content = generateRenderContent(plugins, {
    title: 'Home page',
  });

  const {htmlAttributes, helpers, bodyContent} = content;

```

```javascript
res.write(`
        <!DOCTYPE html>
        <html ${helpers.attrs({...htmlAttributes})}>
        <head>
            ${renderHeadContent(content)}
        </head>
        <body ${helpers.attrs(bodyContent.attributes)}>
            ${renderBodyContent(content)}
    `);

  const data = await getUserData();

  res.write(`
            ${content.renderHelpers.renderInlineScript(`
                window.__DATA__ = ${htmlescape(data)};
            `)}
        </body>
        </html>
    `);
  res.end();
});

app.listen(3000);
```