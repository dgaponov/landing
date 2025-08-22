# UIKit & middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/uikit) (https://www.npmjs.com/package/@gravity-ui/uikit) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/uikit/.github/workflows/ci.yml?branch=main&label=CI&logo=github) (https://github.com/gravity-ui/uikit/actions/workflows/ci.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685) (https://preview.gravity-ui.com/uikit/)

Гибкий, высоко практичный и эффективный набор React-компонентов для создания сложных веб-приложений.

<!--GITHUB_BLOCK-->

![Обложка](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/uikit_cover.png)

## Ресурсы

### ![Логотип глобуса (светлая тема)](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/globe_light.svg#gh-light-mode-only) ![Логотип глобуса (тёмная тема)](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/globe_dark.svg#gh-dark-mode-only) [Веб-сайт](https://gravity-ui.com)

### ![Логотип документации (светлая тема)](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/book-open_light.svg#gh-light-mode-only) ![Логотип документации (тёмная тема)](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/book-open_dark.svg#gh-dark-mode-only) [Документация](https://gravity-ui.com/components/uikit/alert)

### ![Логотип Figma (светлая тема)](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/figma_light.svg#gh-light-mode-only) ![Логотип Figma (тёмная тема)](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/figma_dark.svg#gh-dark-mode-only) [Figma](https://www.figma.com/community/file/1271150067798118027/Gravity-UI-Design-System-(Beta))

### ![Логотип Themer (светлая тема)](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/bucket-paint_light.svg#gh-light-mode-only) ![Логотип Themer (тёмная тема)](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/bucket-paint_dark.svg#gh-dark-mode-only) [Themer](https://gravity-ui.com/themer)

### ![Логотип Storybook (светлая тема)](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/storybook_light.svg#gh-light-mode-only) ![Логотип Storybook (тёмная тема)](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/storybook_dark.svg#gh-dark-mode-only) [Storybook](https://preview.gravity-ui.com/uikit/)

### ![Логотип сообщества (светлая тема)](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/telegram_light.svg#gh-light-mode-only) ![Логотип сообщества (тёмная тема)](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/telegram_dark.svg#gh-dark-mode-only) [Сообщество](https://t.me/gravity_ui)

<!--/GITHUB_BLOCK-->

## Установка

```shell
npm install --save-dev @gravity-ui/uikit
```

## Использование

```jsx
import {Button} from '@gravity-ui/uikit';

const SubmitButton = <Button view="action" size="l" />;
```

### Стили

UIKit поставляется с базовыми стилями и темой. Для корректного отображения подключите эти файлы в начале вашего основного файла:

```js
// index.js

import '@gravity-ui/uikit/styles/fonts.css';
import '@gravity-ui/uikit/styles/styles.css';

// ...
```

UIKit поддерживает различные темы: светлую, тёмную и их контрастные варианты. Ваше приложение должно быть обёрнуто в `ThemeProvider`:

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

Для предотвращения мерцания темы при SSR можно предварительно сгенерировать корневые CSS-классы:

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

Дополнительно доступен файл SCSS [миксинов](styles/mixins.scss) с полезными помощниками для вашего приложения.

### Интернационализация (I18N)

Некоторые компоненты содержат текстовые элементы. Доступны две локали: `en` (по умолчанию) и `ru`.
Для настройки языка используйте функцию `configure`:

```js
// index.js

import {configure} from '@gravity-ui/uikit';

configure({
  lang: 'ru',
});
```

## Разработка

Для запуска сервера разработки со Storybook выполните команду:

```shell
npm start
```

<p align="center">
  <a href="README.md">English</a> |
  <a href="README.zh-CN.md">简体中文</a> |
  <a href="README.es-ES.md">Español</a> |
  <span>Русский</span>
</p>