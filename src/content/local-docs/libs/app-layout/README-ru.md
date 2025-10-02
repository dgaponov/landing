# @gravity-ui/app-layout · [![npm package](https://img.shields.io/npm/v/@gravity-ui/app-layout)](https://www.npmjs.com/package/@gravity-ui/app-layout) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/app-layout/.github/workflows/ci.yml?label=CI&logo=github)](https://github.com/gravity-ui/app-layout/actions/workflows/ci.yml?query=branch:main)

## Установка

```shell
npm install --save-dev @gravity-ui/app-layout
```

## Использование

С `express`:

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

где

```typescript
interface RenderParams<Data, Plugins> {
  // Любые данные, совместимые с JSON, будут установлены в window.__DATA__ на странице
  data?: Data;
  // favicon
  icon?: Icon;
  // nonce для установки в соответствующие теги
  nonce?: string;

  // общие опции
  // Заголовок страницы
  title: string;
  // язык страницы, будет установлен в тег html
  lang?: string;
  isMobile?: boolean;

  // атрибуты html
  htmlAttributes?: string;
  // содержимое тега header
  // мета-теги
  meta?: Meta[];
  // теги link
  links?: Link[];
  // теги script
  scripts?: Script[];
  // теги style
  styleSheets?: Stylesheet[];
  // теги script с встроенным кодом
  inlineScripts?: string[];
  // теги style с встроенными стилями
  inlineStyleSheets?: string[];

  // содержимое тега body
  bodyContent?: {
    // класс для тега body
    className?: string;
    // атрибуты body
    attributes?: string;
    // содержимое body перед div с id root
    beforeRoot?: string;
    // innerHtml содержимое div с id root
    root?: string;
    // содержимое body после div с id root
    afterRoot?: string;
  };
  // опции плагинов
  pluginsOptions?: Partial<PluginsOptions<Plugins>>;
}
```

### Meta

Описывает тег `meta`:

```typescript
interface Meta {
  name: string;
  content: string;
}
```

Пример:

```js
const meta = [
  {name: 'description', content: 'some text'},
  {name: 'robots', content: 'noindex'},
  {name: 'og:title', content: 'Some title'},
];
```

Будет отрендерено как:

```html
<meta name="description" content="some text" />
<meta name="robots" content="noindex" />
<meta property="og:title" content="Some title" />
```

### Icon

Описывает favicon страницы:

```typescript
interface Icon {
  type?: string;
  sizes?: string;
  href?: string;
}
```

Значение по умолчанию:

```js
const icon = {
  type: 'image/png',
  sizes: '16x16',
  href: '/favicon.png',
};
```

### Links

Описывает тег `link`:

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

Пример:

```js
const link = {
  href: 'myFont.woff2',
  rel: 'preload',
  as: 'font',
  type: 'font/woff2',
  crossOrigin: 'anonymous',
};
```

будет отрендерено как:

```html
<link href="myFont.woff2" rel="preload" as="font" type="font/woff2" crossorigin="anonymous" />
```

### Scripts

Описывает ссылку на скрипт с preload:

```typescript
interface Script {
  src: string;
  defer?: boolean;
  async?: boolean;
  crossOrigin?: '' | 'anonymous' | 'use-credentials';
  type?: 'importmap' | 'module' | string;
}
```

Пример:

```js
const script = {
  src: 'url/to/script',
  defer: true,
  async: false,
  crossOrigin: 'anonymous',
};
```

будет отрендерено как:

```html
<link href="url/to/script" rel="preload" as="script" crossorigin="anonymous" />

<script src="url/to/script" defer="true" async="false" crossorigin="anonymous" nonce="..."></script>
```

#### Style sheets

Описывает ссылку на стили:

```typescript
interface Stylesheet {
  href: string;
}
```

Пример:

```js
const styleSheet = {
  href: 'url/to/stylesheet',
};
```

будет отрендерено как:

```html
<link href="url/to/stylesheet" rel="stylesheet" />
```

## Плагины

Функция рендеринга может быть расширена плагинами. Плагин может перезаписывать пользовательское содержимое рендеринга.
Плагин — это объект с свойствами `name` и `apply`:

```typescript
interface Plugin<Options = any, Name = string> {
  name: Name;
  apply: (params: {
    options: Options | undefined; // передается через функцию `renderLayout` в параметре `pluginsOptions`.
    commonOptions: CommonOptions;
    renderContent: RenderContent;
    /** @deprecated используйте `renderContent.helpers` вместо этого */
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

В этом пакете есть несколько плагинов:

### Google analytics

Добавляет счетчик Google Analytics на страницу.

Использование:

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

Параметры плагина:

```typescript
interface GoogleAnalyticsCounter {
  id: string;
}

interface GoogleAnalyticsOptions {
  useBeaconTransport?: boolean;
  counter: GoogleAnalyticsCounter;
}
```

### Яндекс Метрика

Добавляет счётчики метрики Яндекса на страницу.

Пример использования:

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

Параметры плагина:

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

Добавляет скрипты и стили из манифеста webpack-ассетов.

Пример использования:

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

Параметры плагина:

```typescript
export interface LayoutOptions {
  name: string;
  prefix?: string;
}
```

### @gravity-ui/uikit

Добавляет атрибуты для body.

Пример использования:

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

Параметры плагина:

```typescript
interface UikitPluginOptions {
  theme: string;
  direction?: 'ltr' | 'rtl';
}
```

### Remote Versions

Добавляет информацию о версиях микрофронтендов на страницу.

Этот плагин создаёт глобальный объект `window.__REMOTE_VERSIONS__`, содержащий предоставленные версии микрофронтендов. Он может использоваться в архитектурах с модульной федерацией или аналогичными подходами для микрофронтендов, чтобы определить, какие версии удалённых модулей нужно загружать.

Его можно комбинировать с [App Builder](https://github.com/gravity-ui/app-builder?tab=readme-ov-file#module-federation) и опцией `moduleFederation.remotesRuntimeVersioning`, чтобы автоматически загружать удалённые модули с соответствующими версиями.

Пример использования:

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

Параметры плагина:

```typescript
type RemoteVersionsPluginOptions = Record<string, string>;
```

### Helpers

Есть вспомогательная функция для создания всех плагинов:

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

## Альтернативное использование

С рендерерами частей `generateRenderContent`, `renderHeadContent`, `renderBodyContent` через потоковую передачу HTML:

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