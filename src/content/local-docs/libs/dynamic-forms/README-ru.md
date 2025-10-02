# @gravity-ui/dynamic-forms &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/dynamic-forms)](https://www.npmjs.com/package/@gravity-ui/dynamic-forms) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/dynamic-forms/.github/workflows/ci.yml?label=CI&logo=github)](https://github.com/gravity-ui/dynamic-forms/actions/workflows/ci.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/dynamic-forms/)

Библиотека на базе JSON Schema для создания форм и работы с их значениями.

## Установка

```shell
npm install --save-dev @gravity-ui/dynamic-forms
```

## Использование

```jsx
import {DynamicField, Spec, dynamicConfig} from '@gravity-ui/dynamic-forms';

// Для встраивания в final-form
<DynamicField name={name} spec={spec} config={config} />;

import {DynamicView, dynamicViewConfig} from '@gravity-ui/dynamic-forms';

// Для просмотра значений
<DynamicView value={value} spec={spec} config={dynamicViewConfig} />;
```

### I18N

Некоторые компоненты содержат текстовые токены (слова и фразы), доступные на двух языках: `en` (по умолчанию) и `ru`. Чтобы установить язык, используйте функцию `configure`:

```js
// index.js

import {configure, Lang} from '@gravity-ui/dynamic-forms';

configure({lang: Lang.Ru});
```

## Разработка

Чтобы запустить сервер разработки со Storybook, выполните следующую команду:

```shell
npm ci
npm run dev
```