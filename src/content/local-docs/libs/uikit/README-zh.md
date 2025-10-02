# UIKit &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/uikit?logo=npm)](https://www.npmjs.com/package/@gravity-ui/uikit) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/uikit/.github/workflows/ci.yml?branch=main&label=CI&logo=github)](https://github.com/gravity-ui/uikit/actions/workflows/ci.yml?query=branch:main) [![storybook tests](https://img.shields.io/github/actions/workflow/status/gravity-ui/uikit/.github/workflows/test-storybook.yml?label=Storybook%20Tests&logo=github)](https://github.com/gravity-ui/uikit/actions/workflows/test-storybook.yml) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685?logo=storybook)](https://preview.gravity-ui.com/uikit/)

一套灵活、高效且实用的 React 组件，用于构建丰富的 Web 应用。

<!--GITHUB_BLOCK-->

![Cover image](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/uikit_cover.png)

## 资源

### ![Globe Logo Light](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/globe_light.svg#gh-light-mode-only) ![Globe Logo Dark](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/globe_dark.svg#gh-dark-mode-only) [官网](https://gravity-ui.com)

### ![Documentation Logo Light](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/book-open_light.svg#gh-light-mode-only) ![Documentation Logo Dark](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/book-open_dark.svg#gh-dark-mode-only) [文档](https://gravity-ui.com/components/uikit/alert)

### ![Figma Logo Light](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/figma_light.svg#gh-light-mode-only) ![Figma Logo Dark](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/figma_dark.svg#gh-dark-mode-only) [Figma](<https://www.figma.com/community/file/1271150067798118027/Gravity-UI-Design-System-(Beta)>)

### ![Themer Logo Light](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/bucket-paint_light.svg#gh-light-mode-only) ![Themer Logo Dark](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/bucket-paint_dark.svg#gh-dark-mode-only) [Themer](https://gravity-ui.com/themer)

### ![Storybook Logo Light](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/storybook_light.svg#gh-light-mode-only) ![Storybook Logo Dark](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/storybook_dark.svg#gh-dark-mode-only) [Storybook](https://preview.gravity-ui.com/uikit/)

### ![Community Logo Light](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/telegram_light.svg#gh-light-mode-only) ![Community Logo Dark](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/telegram_dark.svg#gh-dark-mode-only) [社区](https://t.me/gravity_ui)

<!--/GITHUB_BLOCK-->

## 安装

```shell
npm install --save-dev @gravity-ui/uikit
```

## 使用

```jsx
import {Button} from '@gravity-ui/uikit';

const SubmitButton = <Button view="action" size="l" />;
```

### 样式

UIKit 自带基础样式和主题。为了让一切看起来更美观，请在入口文件的顶部引入以下内容：

```js
// index.js

import '@gravity-ui/uikit/styles/fonts.css';
import '@gravity-ui/uikit/styles/styles.css';

// ...
```

UIKit 支持多种主题：浅色、深色及其对比变体。您的应用必须在 `ThemeProvider` 内渲染：

```js
import {createRoot} from 'react-dom/client';
import {ThemeProvider} from '@gravity-ui/uikit';

const root = createRoot(document.getElementById('root'));
root.render(
  <ThemeProvider theme="light">
    <App />
  </ThemeProvider>,
);
```

在 SSR 期间，可以生成初始的根 CSS 类，以避免主题闪烁：

```js
import {getRootClassName} from '@gravity-ui/uikit/server';

const theme = 'dark';
const rootClassName = getRootClassName({theme});

const html = `
<html>
  <body>
    <div id="root" class="${rootClassName}"></div>
  </body>
</html>
`;
```

此外，还有一个 SCSS [mixins](styles/mixins.scss) 文件，包含有用的辅助工具，可在您的应用中使用。

### 国际化 (I18N)

某些组件包含文本令牌（单词和短语）。它们支持两种语言：`en`（默认）和 `ru`。
要设置语言，请使用 `configure` 函数：

```js
// index.js

import {configure} from '@gravity-ui/uikit';

configure({
  lang: 'ru',
});
```

## 开发

要启动开发服务器并运行 Storybook，请执行以下命令：

```shell
git clone git@github.com:gravity-ui/uikit.git
cd uikit
npm ci
npm run start
```