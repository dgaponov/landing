# @gravity-ui/illustrations &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/illustrations)](https://www.npmjs.com/package/@gravity-ui/illustrations) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/illustrations/.github/workflows/ci.yml?label=CI&logo=github)](https://github.com/gravity-ui/illustrations/actions/workflows/ci.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/illustrations/)

## Установка

```shell
npm install --save-dev @gravity-ui/illustrations
```

## Использование

### React

#### Подготовка

Настройте тему иллюстраций. Выполните один из следующих шагов:

##### Определение CSS-токенов с собственной цветовой палитрой

Определите следующие CSS-токены в приложении:

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

##### Использование миксинов с темой gravity по умолчанию в SCSS

Используйте следующие миксины для стилизации иллюстраций в разных темах:

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

##### Альтернатива для проектов с уже установленной темой gravity

Если в проекте уже установлен `@gravity-ui/uikit` и используется тема по умолчанию, вы можете просто импортировать `styles.scss` в корневой файл стилей вашего проекта:

```scss
// существующие стили gravity
import '@gravity-ui/uikit/styles/styles.css';
// просто добавьте ещё один импорт ниже
import '@gravity-ui/illustrations/styles/styles.scss';
```

#### Использование компонентов

```js
import NotFound from '@gravity-ui/illustrations/NotFound';
```

или

```js
import {NotFound} from '@gravity-ui/illustrations';
```

### SVG

> Для этого может потребоваться подходящий загрузчик

```js
import notFound from '@gravity-ui/illustrations/svgs/not-found-light.svg';
```

### Разработка

Для обновления иллюстраций в соответствии с новым дизайном измените содержимое SVG-файлов в светлой теме (файлы `<корень-репозитория>/svgs/<имя-иллюстрации>-light.svg`), а затем выполните команду:

```shell
npm run generate
```