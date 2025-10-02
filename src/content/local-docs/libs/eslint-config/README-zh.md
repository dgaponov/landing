# @gravity-ui/eslint-config

## 安装

```
npm install --save-dev eslint @gravity-ui/eslint-config
```

## 使用

在您的项目中添加 `eslint.config.js` 文件，并使用以下内容：

```js
import baseConfig from '@gravity-ui/eslint-config';

export default [
  ...baseConfig,
  {
    // ...其他配置
  },
];
```

基础配置还包括 TypeScript 规则。

### Prettier

如果您正在使用 Prettier，请添加相应的配置：

```js
import baseConfig from '@gravity-ui/eslint-config';
import prettierConfig from '@gravity-ui/eslint-config/prettier';

export default [
  ...baseConfig,
  ...prettierConfig,
  {
    // ...其他配置
  },
];
```

### a11y

如果您想检测可访问性问题，请添加相应的配置：

```js
import baseConfig from '@gravity-ui/eslint-config';
import a11yConfig from '@gravity-ui/eslint-config/a11y';

export default [
  ...baseConfig,
  ...a11yConfig,
  {
    // ...其他配置
  },
];
```

### Order

如果您想强制执行模块导入顺序的约定，请添加相应的配置：

```js
import baseConfig from '@gravity-ui/eslint-config';
import importOrderConfig from '@gravity-ui/eslint-config/import-order';

export default [
  ...baseConfig,
  ...importOrderConfig,
  {
    // ...其他配置
  },
];
```