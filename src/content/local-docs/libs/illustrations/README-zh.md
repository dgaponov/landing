# @gravity-ui/illustrations &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/illustrations)](https://www.npmjs.com/package/@gravity-ui/illustrations) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/illustrations/.github/workflows/ci.yml?label=CI&logo=github)](https://github.com/gravity-ui/illustrations/actions/workflows/ci.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/illustrations/)

## 安装

```shell
npm install --save-dev @gravity-ui/illustrations
```

## 使用

### React

#### 准备

设置插图主题。请执行以下任一步骤：

##### 使用自定义颜色调色板定义 CSS 变量

在应用中定义以下 CSS 变量：

```scss
--gil-color-object-base: rgb(255, 190, 92);
--gil-color-object-accent-heavy: rgb(211, 101, 7);
--gil-color-object-hightlight: rgb(255, 216, 157);
--gil-color-shadow-over-object: rgb(211, 158, 80);
--gil-color-background-lines: rgb(140, 140, 140);
--gil-color-background-shapes: rgb(242, 242, 242);
--gil-color-object-accent-light: rgb(255, 255, 255);
--gil-color-object-danger: rgb(255, 0, 61);
```

##### 在 SCSS 中使用 mixin 和默认 Gravity 主题

使用以下 mixin 来为不同主题的插图进行样式设置

```scss
@import '@gravity-ui/illustrations/styles/theme.scss';

.g-root {
  &_theme_light {
    @include g-illustrations-colors-light;
  }

  &_theme_light-hc {
    @include g-illustrations-colors-light-hc;
  }

  &_theme_dark {
    @include g-illustrations-colors-dark;
  }

  &_theme_dark-hc {
    @include g-illustrations-colors-dark-hc;
  }
}
```

##### 对于已安装 Gravity 主题的项目的替代方案

或者，如果项目中已安装 `@gravity-ui/uikit` 并使用默认主题，您只需在项目根样式文件中导入 `styles.scss`：

```scss
// 现有的 Gravity 样式定义
import '@gravity-ui/uikit/styles/styles.css';
// 在下方添加一个导入
import '@gravity-ui/illustrations/styles/styles.scss';
```

#### 组件使用

```js
import NotFound from '@gravity-ui/illustrations/NotFound';
```

或

```js
import {NotFound} from '@gravity-ui/illustrations';
```

### SVG

> 您可能需要合适的加载器来处理此操作

```js
import notFound from '@gravity-ui/illustrations/svgs/not-found-light.svg';
```

### 开发

要根据新设计更新插图，请修改浅色主题的 SVG 文件内容（位于 `<此仓库根目录>/svgs/<插图名称>-light.svg` 文件），然后运行命令：

```shell
npm run generate
```