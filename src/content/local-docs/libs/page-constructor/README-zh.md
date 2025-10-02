# @gravity-ui/page-constructor &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/page-constructor)](https://www.npmjs.com/package/@gravity-ui/page-constructor) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/page-constructor/ci.yml?branch=main&label=CI)](https://github.com/gravity-ui/page-constructor/actions/workflows/ci.yml?query=branch:main) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/page-constructor/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/page-constructor/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/page-constructor/)

## 页面构建器

`Page-constructor` 是一个用于基于 `JSON` 数据渲染网页或其部分的库（后续将添加对 `YAML` 格式的支持）。

在创建页面时，采用组件化方法：页面由一组可按任意顺序放置的现成块构建而成。每个块都有特定的类型和一组输入数据参数。

有关输入数据格式和可用块列表，请参阅[文档](https://preview.gravity-ui.com/page-constructor/?path=/docs/documentation-blocks--docs)。

## 安装

```shell
npm install @gravity-ui/page-constructor
```

## 快速开始

首先，我们需要一个 React 项目和某种服务器。例如，您可以使用 Vite 创建 React 项目并搭配 Express 服务器，或者直接创建 Next.js 应用——它同时具备客户端和服务器端。

安装所需的依赖：

```shell
npm install @gravity-ui/page-constructor @diplodoc/transform @gravity-ui/uikit
```

将 `Page Constructor` 插入到页面中。为了正常工作，它必须被包裹在 `PageConstructorProvider` 中：

```tsx
import {PageConstructor, PageConstructorProvider} from '@gravity-ui/page-constructor';
import '@gravity-ui/page-constructor/styles/styles.scss';

const App = () => {
  const content = {
    blocks: [
      {
        type: 'header-block',
        title: 'Hello world',
        background: {color: '#f0f0f0'},
        description:
          '**Congratulations!** Have you built a [page-constructor](https://github.com/gravity-ui/page-constructor) into your website',
      },
    ],
  };

  return (
    <PageConstructorProvider>
      <PageConstructor content={content} />
    </PageConstructorProvider>
  );
};

export default App;
```

这是一个最简单的集成示例。为了让 YFM 标记正常工作，您需要在服务器端处理内容，并在客户端接收它。

如果您的服务器是一个独立的应用程序，则需要安装 page-constructor：

```shell
npm install @gravity-ui/page-constructor
```

要处理所有基础块中的 YFM，请调用 `contentTransformer` 并传递内容和选项：

```ts
const express = require('express');
const app = express();
const {contentTransformer} = require('@gravity-ui/page-constructor/server');

const content = {
  blocks: [
    {
      type: 'header-block',
      title: 'Hello world',
      background: {color: '#f0f0f0'},
      description:
        '**Congratulations!** Have you built a [page-constructor](https://github.com/gravity-ui/page-constructor) into your website',
    },
  ],
};

app.get('/content', (req, res) => {
  res.send({content: contentTransformer({content, options: {lang: 'en'}})});
});

app.listen(3000);
```

在客户端，添加一个端点调用来接收内容：

```tsx
import {PageConstructor, PageConstructorProvider} from '@gravity-ui/page-constructor';
import '@gravity-ui/page-constructor/styles/styles.scss';
import {useEffect, useState} from 'react';

const App = () => {
  const [content, setContent] = useState();

  useEffect(() => {
    (async () => {
      const response = await fetch('http://localhost:3000/content').then((r) => r.json());
      setContent(response.content);
    })();
  }, []);

  return (
    <PageConstructorProvider>
      <PageConstructor content={content} />
    </PageConstructorProvider>
  );
};

export default App;
```

### 现成模板

要启动新项目，您可以使用我们准备的[基于 Next.js 的现成模板](https://github.com/gravity-ui/page-constructor-website-template)。

### 静态站点构建器

[Page Constructor Builder](https://github.com/gravity-ui/page-constructor-builder) - 一个命令行实用工具，用于从 YAML 配置使用 @gravity-ui/page-constructor 构建静态页面。

## 文档

### 参数

```typescript
interface PageConstructorProps {
  content: PageContent; // 以 JSON 格式的块数据。
  shouldRenderBlock?: ShouldRenderBlock; // 在渲染每个块时调用的函数，可用于设置其显示条件。
  custom?: Custom; // 自定义块（参见“自定义”部分）。
  renderMenu?: () => React.ReactNode; // 渲染页面导航菜单的函数（我们计划添加默认菜单版本的渲染支持）。
  navigation?: NavigationData; // 以 JSON 格式的导航数据，用于在组件中使用导航。
  isBranded?: boolean; // 如果为 true，则添加指向 https://gravity-ui.com/ 的页脚。尝试 BrandFooter 组件以获得更多自定义选项。
}

interface PageConstructorProviderProps {
  isMobile?: boolean; // 表示代码在移动模式下执行的标志。
  locale?: LocaleContextProps; // 关于语言和域名的信息（用于生成和格式化链接）。
  location?: Location; // 浏览器或路由器历史记录的 API，即页面 URL。
  analytics?: AnalyticsContextProps; // 处理分析事件的函数。

  ssrConfig?: SSR; // 表示代码在服务器端运行的标志。
  theme?: 'light' | 'dark'; // 渲染页面的主题。
  mapsContext?: MapsContextType; // 地图参数：apikey、type、scriptSrc、nonce。
}

export interface PageContent extends Animatable {
  blocks: Block[];
  menu?: Menu;
  background?: MediaProps;
}

interface Custom {
  blocks?: CustomItems;
  subBlocks?: CustomItems;
  headers?: CustomItems;
  loadable?: LoadableConfig;
}

type ShouldRenderBlock = (block: Block, blockKey: string) => Boolean;

interface Location {
  history?: History;
  search?: string;
  hash?: string;
  pathname?: string;
  hostname?: string;
}

interface Locale {
  lang?: Lang;
  tld?: string;
}

interface SSR {
  isServer?: boolean;
}

interface NavigationData {
  logo: NavigationLogo;
  header: HeaderData;
}

interface NavigationLogo {
  icon: ImageProps;
  text?: string;
  url?: string;
}

interface HeaderData {
  leftItems: NavigationItem[];
  rightItems?: NavigationItem[];
}
```

```typescript
interface NavigationLogo {
  icon: ImageProps;
  text?: string;
  url?: string;
}
```

### 服务器工具

该包提供了一组服务器工具，用于转换您的内容。

```ts
const {fullTransform} = require('@gravity-ui/page-constructor/server');

const {html} = fullTransform(content, {
  lang,
  extractTitle: true,
  allowHTML: true,
  path: __dirname,
  plugins,
});
```

在底层，该包使用 `diplodoc/transform` 来将 Yandex 风格 Markdown 转换为 HTML，因此它也被列为 peer 依赖。

您也可以在需要的地方使用这些有用的工具，例如在自定义组件中。

```ts
const {
  typografToText,
  typografToHTML,
  yfmTransformer,
} = require('@gravity-ui/page-constructor/server');

const post = {
  title: typografToText(title, lang),
  content: typografToHTML(content, lang),
  description: yfmTransformer(lang, description, {plugins}),
};
```

您可以在[此部分](https://github.com/gravity-ui/page-constructor/tree/main/src/text-transform)找到更多工具。

### 服务器工具和转换器的详细文档

有关使用服务器工具的全面指南，包括详细解释和高级用例，请访问[服务器工具使用附加章节](./docs/data-preparation.md)。

### 自定义块

页面构造器允许您使用在应用中用户定义的块。这些块是普通的 React 组件。

要将自定义块传递给构造器：

1. 在您的应用中创建一个块。

2. 在您的代码中，创建一个对象，以块类型（字符串）作为键，以导入的块组件作为值。

3. 将您创建的对象传递给 `PageConstructor` 组件的 `custom.blocks`、`custom.headers` 或 `custom.subBlocks` 参数（`custom.headers` 指定要在一般内容上方单独渲染的块标题）。

4. 现在，您可以在输入数据（`content` 参数）中使用创建的块，通过指定其类型和数据。

在创建自定义块时，要使用 mixin 和构造器样式变量，请在您的文件中添加导入：

```css
@import '~@gravity-ui/page-constructor/styles/styles.scss';
```

要使用默认字体，请在您的文件中添加导入：

```css
@import '~@gravity-ui/page-constructor/styles/fonts.scss';
```

### 可加载块

有时，块需要根据要加载的数据来渲染自身。在这种情况下，使用可加载块。

要添加自定义 `loadable` 块，请向 `PageConstructor` 传递 `custom.loadable` 属性，其中键为组件的数据源名称（字符串），值为一个对象。

```typescript
export interface LoadableConfigItem {
  fetch: FetchLoadableData; // 数据加载方法
  component: React.ComponentType; // 用于传递已加载数据
}

type FetchLoadableData<TData = any> = (blockKey: string) => Promise<TData>;
```

### 网格

页面构造器使用 `bootstrap` 网格及其基于 React 组件的实现，您可以在自己的项目中使用这些组件（包括独立于构造器）。

使用示例：

```jsx
import {Grid, Row, Col} from '@gravity-ui/page-constructor';

const Page = ({children}: PropsWithChildren<PageProps>) => (
  <Grid>
    <Row>
      <Col sizes={{lg: 4, sm: 6, all: 12}}>{children}</Col>
    </Row>
  </Grid>
);
```

### 导航

页面导航也可以独立于构造器使用：

```jsx
import {Navigation} from '@gravity-ui/page-constructor';

const Page= ({data, logo}: React.PropsWithChildren<PageProps>) => <Navigation data={data} logo={logo} />;
```

### 块

每个块都是一个原子化的顶级组件。它们存储在 `src/units/constructor/blocks` 目录中。

### 子块

子块是可以在块 `children` 属性中使用的组件。在配置中，指定子块的子组件列表。渲染后，这些子块会作为 `children` 传递给块。

### 如何向 `page-constructor` 添加新块

1. 在 `src/blocks` 或 `src/sub-blocks` 目录中，为块或子块代码创建一个文件夹。

2. 将块或子块名称添加到枚举 `BlockType` 或 `SubBlockType`，并在 `src/models/constructor-items/blocks.ts` 或 `src/models/constructor-items/sub-blocks.ts` 文件中描述其属性，类似于现有的方式。

3. 在 `src/blocks/index.ts` 文件中为块添加导出，在 `src/sub-blocks/index.ts` 文件中为子块添加导出。

4. 在 `src/constructor-items.ts` 中的映射中添加新组件或块。

5. 为新块添加验证器：

   - 在块或子块目录中添加 `schema.ts` 文件。在该文件中，使用 [`json-schema`](http://json-schema.org/) 格式描述组件的参数验证器。
   - 在 `schema/validators/blocks.ts` 或 `schema/validators/sub-blocks.ts` 文件中导出它。
   - 在 `schema/index.ts` 文件中的 `enum` 或 `selectCases` 中添加它。

6. 在块目录中添加 `README.md` 文件，描述输入参数。

7. 在块目录的 `__stories__` 文件夹中添加 Storybook 示例。所有 Story 的示例内容应放置在 Story 目录的 `data.json` 中。通用的 `Story` 必须接受块 props 的类型，否则 Storybook 中会显示不正确的块 props。

8. 将块数据模板添加到 `src/editor/data/templates/` 文件夹，文件名应匹配块类型。

9. （可选）将块预览图标添加到 `src/editor/data/previews/` 文件夹，文件名应匹配块类型。

### 主题

`PageConstructor` 允许您使用主题：您可以根据应用中选择的主题为单个块属性设置不同的值。

要为块属性添加主题：

1. 在 `models/blocks.ts` 文件中，使用 `ThemeSupporting<T>` 泛型定义相应块属性的类型，其中 `T` 是属性的类型。

2. 在包含块 `react` 组件的文件中，通过 `getThemedValue` 和 `useTheme` 钩子获取带主题的属性值（参见 `MediaBlock.tsx` 块中的示例）。

3. 为属性验证器添加主题支持：在块的 `schema.ts` 文件中，用 `withTheme` 包装此属性。

### i18n

`page-constructor` 是一个基于 `uikit` 的库，我们使用 uikit 中的 `i18n` 实例。要设置国际化，只需使用 uikit 中的 `configure`：

```typescript
import {configure} from '@gravity-ui/uikit';

configure({
  lang: 'ru',
});
```

### 地图

要使用地图，请在 `PageConstructorProvider` 中的 `mapContext` 字段中放置地图类型、scriptSrc 和 apiKey。

您可以在项目根目录的 .env.development 文件中为开发模式定义环境变量。
`STORYBOOK_GMAP_API_KEY` - Google Maps 的 apiKey

### 分析

#### 初始化

要开始使用任何分析，请向构造器传递一个处理程序。该处理程序必须在项目端创建。处理程序将接收 `default` 和 `custom` 事件对象。传递的处理程序将在按钮、链接、导航和控件点击时触发。由于一个处理程序用于所有事件处理，请在创建处理程序时注意如何处理不同事件。有一些预定义字段来帮助您构建复杂逻辑。

向构造器传递 `autoEvents: true` 以自动触发配置的事件。

```ts
function sendEvents(events: MyEventType []) {
  ...
}

<PageConstructorProvider
    ...

    analytics={{sendEvents, autoEvents: true}}

    ...
/>
```

事件对象只有一个必需字段 - `name`。它还具有预定义字段，这些字段有助于管理复杂逻辑。例如，`counter.include` 可以帮助在项目中使用多个分析系统时，将事件发送到特定的计数器。

```ts
type AnalyticsEvent<T = {}> = T & {
  name: string;
  type?: string;
  counters?: AnalyticsCounters;
  context?: string;
};
```

可以为项目配置所需的事件类型。

```ts
type MyEventType = AnalyticsEvent<{
  [key: string]?: string; // 仅支持 'string' 类型
}>;
```

#### 计数器选择器

可以配置事件发送到哪个分析系统。

```ts
type AnalyticsCounters = {
  include?: string[]; // 将应用的分析计数器 ID 数组
  exclude?: string[]; // 不应用的分析计数器 ID 数组
};
```

#### context 参数

通过传递 `context` 值来定义事件在项目中的触发位置。

使用下面的选择器，或创建满足项目需求的逻辑。

```ts
// analyticsHandler.ts
if (isCounterAllowed(counterName, counters)) {
  analyticsCounter.reachGoal(counterName, name, parameters);
}
```

#### 保留的事件类型

几个预定义的事件类型用于标记自动配置的事件。例如，可以使用这些类型来过滤默认事件。

```ts
enum PredefinedEventTypes {
  Default = 'default-event', // 默认事件，在每个按钮点击时触发
  Play = 'play', // React 播放器事件
  Stop = 'stop', // React 播放器事件
}
```

## 开发

```bash
npm ci
npm run dev
```

#### 关于 Vite 的说明

```ts
import react from '@vitejs/plugin-react-swc';
import dynamicImport from 'vite-plugin-dynamic-import';

export default defineConfig({
  plugins: [
    react(),
    dynamicImport({
      filter: (id) => id.includes('/node_modules/@gravity-ui/page-constructor'),
    }),
  ],
});
```

对于 Vite，您需要安装 `vite-plugin-dynamic-import` 插件，并配置配置文件以使动态导入正常工作。

## 发布流程

在通常情况下，我们使用两种类型的提交：

1. fix: 此类型的提交修补代码库中的 bug（这对应于语义化版本控制中的 PATCH）。
2. feat: 此类型的提交向代码库引入新功能（这对应于语义化版本控制中的 MINOR）。
3. BREAKING CHANGE: 带有 BREAKING CHANGE: 页脚的提交，或在类型/范围后附加 ! 的提交，会引入破坏性 API 变更（对应于语义化版本控制中的 MAJOR）。BREAKING CHANGE 可以是任何类型的提交的一部分。
4. 要手动设置发布包版本，您需要在提交消息中添加 `Release-As: <version>`，例如：

```bash
git commit -m 'chore: bump release

Release-As: 1.2.3'
```

您可以在[这里](https://www.conventionalcommits.org/en/v1.0.0/)查看所有信息。

当您从代码所有者获得拉取请求的批准并通过所有检查后，请执行以下操作：

1. 检查是否存在来自机器人的发布拉取请求，其中包含其他贡献者的变更（看起来像 `chore(main): release 0.0.0`）。如果存在，请检查为什么它尚未合并。如果贡献者同意发布共享版本，请继续下一步。如果不同意，请让他发布他的版本，然后继续下一步。
2. 压缩并合并您的 PR（使用 Github Actions 发布新版本非常重要）。
3. 等待机器人创建一个带有新包版本和 CHANGELOG.md 中变更信息的 PR。您可以在[Actions 标签页](https://github.com/gravity-ui/page-constructor/actions)上查看进程。
4. 检查 CHANGELOG.md 中的变更并批准机器人的 PR。
5. 压缩并合并 PR。您可以在[Actions 标签页](https://github.com/gravity-ui/page-constructor/actions)上查看发布进程。

### Alpha 版本发布

如果您想从您的分支手动发布包的 alpha 版本，可以这样做：

1. 转到 Actions 标签页。
2. 在左侧页面选择工作流“Release alpha version”。
3. 在右侧，您会看到“Run workflow”按钮。在这里可以选择分支。
4. 您还可以看到手动版本字段。如果这是您分支中的首次 alpha 发布，请不要在这里设置任何内容。在首次发布后，您必须手动设置新版本，因为如果分支即将过期，我们不会更改 package.json。在手动版本中使用 `alpha` 前缀，否则会出错。
5. 点击“Run workflow”并等待操作完成。您可以发布任意多个版本，但请不要滥用它，只在真正需要时发布版本。在其他情况下，使用[npm pack](https://docs.npmjs.com/cli/v7/commands/npm-pack)。

### Beta-major 版本发布

如果您想发布新主版本，在稳定版本之前可能需要 beta 版本，请执行以下操作：

1. 创建或更新分支 `beta`。
2. 在其中添加您的变更。
3. 当您准备好新 beta 版本时，使用空提交手动发布它（或者您可以将带有页脚的此提交消息添加到最后一个提交）：

```bash
git commit -m 'fix: last commit

Release-As: 3.0.0-beta.0' --allow-empty
```

4. 发布后，机器人将向 `beta` 分支创建一个新 PR，其中包含更新的 CHANGELOG.md 和包版本提升。
5. 您可以重复此操作任意多次。当您准备好发布不带 beta 标签的最新主版本时，您必须从 `beta` 分支向 `main` 分支创建 PR。请注意，您的包版本带有 beta 标签是正常的。机器人知道这一点并会正确更改它。`3.0.0-beta.0` 将变为 `3.0.0`。

### 先前主版本的发布流程

如果您想在将提交到 main 后，在先前主版本中发布新版本，请执行以下操作：

1. 更新必要的分支，先前主发布分支名称为：
   1. `version-1.x.x/fixes` - 用于主版本 1.x.x
   2. `version-2.x.x` - 用于主版本 2.x.x
2. 从先前主发布分支检出新分支。
3. 从 `main` 分支 cherry-pick 您的提交。
4. 创建 PR，获得批准并合并到先前主发布分支。
5. 压缩并合并您的 PR（使用 Github Actions 发布新版本非常重要）。
6. 等待机器人创建一个带有新包版本和 CHANGELOG.md 中变更信息的 PR。您可以在[Actions 标签页](https://github.com/gravity-ui/page-constructor/actions)上查看进程。
7. 检查 CHANGELOG.md 中的变更并批准机器人的 PR。
8. 压缩并合并 PR。您可以在[Actions 标签页](https://github.com/gravity-ui/page-constructor/actions)上查看发布进程。

## 页面构造函数编辑器

编辑器提供页面内容管理的用户界面，并支持实时预览。

使用方法：

```tsx
import {Editor} from '@gravity-ui/page-constructor/editor';

interface MyAppEditorProps {
  initialContent: PageContent;
  transformContent: ContentTransformer;
  onChange: (content: PageContent) => void;
}

export const MyAppEditor = ({initialContent, onChange, transformContent}: MyAppEditorProps) => (
  <Editor content={initialContent} onChange={onChange} transformContent={transformContent} />
);
```

## 记忆库

这个项目包含一个全面的 **Memory Bank** - 这是一个 Markdown 文档文件集合，提供有关项目架构、组件和使用模式的详细信息。Memory Bank 在与 AI 代理合作时特别有用，因为它包含关于以下内容的结构化信息：

- **项目概述**：核心需求、目标和上下文
- **组件文档**：所有组件的详细使用指南
- **系统架构**：技术模式和设计决策
- **开发进度**：当前状态和实现细节

### 使用 Memory Bank

Memory Bank 位于 `memory-bank/` 目录中，由常规的 Markdown 文件组成，可以像其他文档一样阅读：

- `projectbrief.md` - 包含核心需求的基础文档
- `productContext.md` - 项目目的和用户体验目标
- `systemPatterns.md` - 架构和技术决策
- `techContext.md` - 技术、设置和约束
- `activeContext.md` - 当前工作重点和最近更改
- `progress.md` - 实现状态和已知问题
- `usage/` - 特定组件的使用文档
- `storybookComponents.md` - Storybook 集成细节

### 对于 AI 代理

在与 AI 代理合作这个项目时，Memory Bank 作为一个全面的知识库，帮助代理理解：

- 项目结构和模式
- 组件 API 和使用示例
- 开发工作流程和最佳实践
- 当前实现状态和下一步

AI 代理可以阅读这些文件，快速了解项目上下文，并对代码更改和实现做出更明智的决策。

## 测试

全面的文档可在提供的 [link](./test-utils/docs/README.md) 中获取。