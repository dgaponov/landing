# @gravity-ui/eslint-config

## Установка

```
npm install --save-dev eslint @gravity-ui/eslint-config
```

## Использование

Добавьте файл `eslint.config.js` в ваш проект со следующим содержимым:

```js
import baseConfig from '@gravity-ui/eslint-config';

export default [
  ...baseConfig,
  {
    // ...other config
  },
];
```

Базовая конфигурация также включает правила для TypeScript.

### Prettier

Если вы используете Prettier, добавьте соответствующую конфигурацию:

```js
import baseConfig from '@gravity-ui/eslint-config';
import prettierConfig from '@gravity-ui/eslint-config/prettier';

export default [
  ...baseConfig,
  ...prettierConfig,
  {
    // ...other config
  },
];
```

### a11y

Если вы хотите выявлять проблемы доступности, добавьте соответствующую конфигурацию:

```js
import baseConfig from '@gravity-ui/eslint-config';
import a11yConfig from '@gravity-ui/eslint-config/a11y';

export default [
  ...baseConfig,
  ...a11yConfig,
  {
    // ...other config
  },
];
```

### Order

Если вы хотите навязывать конвенцию в порядке импорта модулей, добавьте соответствующую конфигурацию:

```js
import baseConfig from '@gravity-ui/eslint-config';
import importOrderConfig from '@gravity-ui/eslint-config/import-order';

export default [
  ...baseConfig,
  ...importOrderConfig,
  {
    // ...other config
  },
];
```