# @gravity-ui/app-layout · [![npm package](https://img.shields.io/npm/v/@gravity-ui/app-layout)](https://www.npmjs.com/package/@gravity-ui/app-layout) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/app-layout/.github/workflows/ci.yml?label=CI&logo=github)](https://github.com/gravity-ui/app-layout/actions/workflows/ci.yml?query=branch:main)

## Installation

```shell
npm install --save-dev @gravity-ui/app-layout
```

## Verwendung

Mit `express`:

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

wobei

```typescript
interface RenderParams<Data, Plugins> {
  // Beliebige JSON-kompatible Daten, die auf der Seite in window.__DATA__ gesetzt werden
  data?: Data;
  // Favicon
  icon?: Icon;
  // Nonce, das auf den entsprechenden Tags gesetzt wird
  nonce?: string;

  // Gemeinsame Optionen
  // Titel der Seite
  title: string;
  // Sprache der Seite, die im html-Tag gesetzt wird
  lang?: string;
  isMobile?: boolean;

  // HTML-Attribute
  htmlAttributes?: string;
  // Inhalt des header-Tags
  // Meta-Tags
  meta?: Meta[];
  // Link-Tags
  links?: Link[];
  // Script-Tags
  scripts?: Script[];
  // Style-Tags
  styleSheets?: Stylesheet[];
  // Script-Tags mit eingebettetem Code
  inlineScripts?: string[];
  // Style-Tags mit eingebetteten Stilen
  inlineStyleSheets?: string[];

  // Inhalt des body-Tags
  bodyContent?: {
    // Klassenname für das body-Tag
    className?: string;
    // Body-Attribute
    attributes?: string;
    // Body-Inhalt vor dem div-Tag mit id root
    beforeRoot?: string;
    // innerHtml-Inhalt des div-Tags mit id root
    root?: string;
    // Body-Inhalt nach dem div-Tag mit id root
    afterRoot?: string;
  };
  // Plugin-Optionen
  pluginsOptions?: Partial<PluginsOptions<Plugins>>;
}
```

### Meta

Beschreibt das `meta`-Tag:

```typescript
interface Meta {
  name: string;
  content: string;
}
```

Beispiel:

```js
const meta = [
  {name: 'description', content: 'some text'},
  {name: 'robots', content: 'noindex'},
  {name: 'og:title', content: 'Some title'},
];
```

Wird gerendert als:

```html
<meta name="description" content="some text" />
<meta name="robots" content="noindex" />
<meta property="og:title" content="Some title" />
```

### Icon

Beschreibt das Favicon der Seite:

```typescript
interface Icon {
  type?: string;
  sizes?: string;
  href?: string;
}
```

Standardwert ist:

```js
const icon = {
  type: 'image/png',
  sizes: '16x16',
  href: '/favicon.png',
};
```

### Links

Beschreibt das `link`-Tag:

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

Beispiel:

```js
const link = {
  href: 'myFont.woff2',
  rel: 'preload',
  as: 'font',
  type: 'font/woff2',
  crossOrigin: 'anonymous',
};
```

Wird gerendert als:

```html
<link href="myFont.woff2" rel="preload" as="font" type="font/woff2" crossorigin="anonymous" />
```

### Scripts

Beschreibt den Link zu einem Script mit Preload:

```typescript
interface Script {
  src: string;
  defer?: boolean;
  async?: boolean;
  crossOrigin?: '' | 'anonymous' | 'use-credentials';
  type?: 'importmap' | 'module' | string;
}
```

Beispiel:

```js
const script = {
  src: 'url/to/script',
  defer: true,
  async: false,
  crossOrigin: 'anonymous',
};
```

Wird gerendert als:

```html
<link href="url/to/script" rel="preload" as="script" crossorigin="anonymous" />

<script src="url/to/script" defer="true" async="false" crossorigin="anonymous" nonce="..."></script>
```

#### Stylesheets

Beschreibt den Link zu Stilen:

```typescript
interface Stylesheet {
  href: string;
}
```

Beispiel:

```js
const styleSheet = {
  href: 'url/to/stylesheet',
};
```

Wird gerendert als:

```html
<link href="url/to/stylesheet" rel="stylesheet" />
```

## Plugins

Die Render-Funktion kann durch Plugins erweitert werden. Ein Plugin kann den vom Benutzer definierten Render-Inhalt überschreiben.
Ein Plugin ist ein Objekt mit den Eigenschaften `name` und `apply`:

```typescript
interface Plugin<Options = any, Name = string> {
  name: Name;
  apply: (params: {
    options: Options | undefined; // Wird über den `renderLayout`-Funktionsparameter `pluginsOptions` übergeben.
    commonOptions: CommonOptions;
    renderContent: RenderContent;
    /** @deprecated Verwenden Sie stattdessen `renderContent.helpers` */
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

Dieses Paket enthält einige Plugins:

### Google Analytics

Fügt den Google-Analytics-Zähler auf der Seite hinzu.

Verwendung:

```js
import {createRenderFunction, createGoogleAnalyticsPlugin} from '@gravity-ui/app-layout';

const renderLayout = createRenderFunction([createGoogleAnalyticsPlugin()]);

```

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

Plugin-Optionen:

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

Fügt Yandex-Metriken-Zähler auf der Seite hinzu.

Verwendung:

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

Plugin-Optionen:

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

Fügt Skripte und Styles aus der Webpack-Assets-Manifest-Datei hinzu.

Verwendung:

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

Plugin-Optionen:

```typescript
export interface LayoutOptions {
  name: string;
  prefix?: string;
}
```

### @gravity-ui/uikit

Fügt Body-Attribute hinzu.

Verwendung:

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

Plugin-Optionen:

```typescript
interface UikitPluginOptions {
  theme: string;
  direction?: 'ltr' | 'rtl';
}
```

### Remote Versions

Fügt Informationen zu Microfrontend-Versionen auf der Seite hinzu.

Dieses Plugin erstellt ein globales `window.__REMOTE_VERSIONS__`-Objekt, das die angegebenen Microfrontend-Versionen enthält. Dies kann von Module Federation oder ähnlichen Microfrontend-Architekturen genutzt werden, um zu bestimmen, welche Versionen der Remote-Module geladen werden sollen.

Es kann in Kombination mit [App Builder](https://github.com/gravity-ui/app-builder?tab=readme-ov-file#module-federation) und der Option `moduleFederation.remotesRuntimeVersioning` verwendet werden, um Remote-Module automatisch mit den entsprechenden Versionen zu laden.

Verwendung:

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

Plugin-Optionen:

```typescript
type RemoteVersionsPluginOptions = Record<string, string>;
```

### Helpers

Es gibt einen Helfer, um alle Plugins zu erstellen:

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

## Alternative Verwendung

Mit Teile-Renderern `generateRenderContent`, `renderHeadContent`, `renderBodyContent` über HTML-Streaming:

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