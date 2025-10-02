# @gravity-ui/page-constructor · [![npm package](https://img.shields.io/npm/v/@gravity-ui/page-constructor)](https://www.npmjs.com/package/@gravity-ui/page-constructor) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/page-constructor/ci.yml?branch=main&label=CI)](https://github.com/gravity-ui/page-constructor/actions/workflows/ci.yml?query=branch:main) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/page-constructor/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/page-constructor/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/page-constructor/)

## Конструктор страниц

`Page-constructor` — это библиотека для рендеринга веб-страниц или их частей на основе данных в формате `JSON` (поддержка формата `YAML` будет добавлена позже).

При создании страниц используется компонентный подход: страница собирается из набора готовых блоков, которые можно размещать в любом порядке. Каждый блок имеет определённый тип и набор параметров входных данных.

Формат входных данных и список доступных блоков описаны в [документации](https://preview.gravity-ui.com/page-constructor/?path=/docs/documentation-blocks--docs).

## Установка

```shell
npm install @gravity-ui/page-constructor
```

## Быстрый старт

Сначала нужен проект на React и какой-нибудь сервер. Например, можно создать проект на React с помощью Vite и сервером Express или использовать приложение Next.js — оно сразу включает клиентскую и серверную части.

Установите необходимые зависимости:

```shell
npm install @gravity-ui/page-constructor @diplodoc/transform @gravity-ui/uikit
```

Вставьте `Page Constructor` на страницу. Для корректной работы его нужно обернуть в `PageConstructorProvider`:

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

Это самый простой пример подключения. Чтобы разметка YFM работала, контент нужно обрабатывать на сервере и получать на клиенте.

Если ваш сервер — отдельное приложение, то установите page-constructor:

```shell
npm install @gravity-ui/page-constructor
```

Для обработки YFM во всех базовых блоках вызовите `contentTransformer` и передайте туда контент и опции:

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

На клиенте добавьте вызов эндпоинта для получения контента:

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

### Готовый шаблон

Для запуска нового проекта можно использовать [готовый шаблон на Next.js](https://github.com/gravity-ui/page-constructor-website-template), который мы подготовили.

### Сборщик статических сайтов

[Page Constructor Builder](https://github.com/gravity-ui/page-constructor-builder) — утилита командной строки для сборки статических страниц из конфигураций YAML с использованием @gravity-ui/page-constructor.

## Документация

### Параметры

```typescript
interface PageConstructorProps {
  content: PageContent; // Данные блоков в формате JSON.
  shouldRenderBlock?: ShouldRenderBlock; // Функция, которая вызывается при рендеринге каждого блока и позволяет задать условия его отображения.
  custom?: Custom; // Пользовательские блоки (см. «Настройка»).
  renderMenu?: () => React.ReactNode; // Функция для рендеринга меню страницы с навигацией (планируем добавить рендеринг версии меню по умолчанию).
  navigation?: NavigationData; // Данные навигации для использования компонента навигации в формате JSON
  isBranded?: boolean; // Если true, добавляет футер со ссылкой на https://gravity-ui.com/. Для большей настройки используйте компонент BrandFooter.
}

interface PageConstructorProviderProps {
  isMobile?: boolean; // Флаг, указывающий, что код выполняется в мобильном режиме.
  locale?: LocaleContextProps; // Информация о языке и домене (используется при генерации и форматировании ссылок).
  location?: Location; // API браузера или истории роутера, URL страницы.
  analytics?: AnalyticsContextProps; // Функция для обработки событий аналитики

  ssrConfig?: SSR; // Флаг, указывающий, что код выполняется на серверной стороне.
  theme?: 'light' | 'dark'; // Тема для рендеринга страницы.
  mapsContext?: MapsContextType; // Параметры для карты: apikey, type, scriptSrc, nonce
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

### Утилиты для сервера

Пакет предоставляет набор серверных утилит для преобразования вашего контента.

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

Под капотом используется пакет для преобразования Yandex Flavored Markdown в HTML — `diplodoc/transform`, поэтому он также входит в peer-зависимости.

Вы также можете использовать полезные утилиты в нужных местах, например, в своих кастомных компонентах.

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

Больше утилит вы можете найти в этом [разделе](https://github.com/gravity-ui/page-constructor/tree/main/src/text-transform).

### Подробная документация по серверным утилитам и трансформерам

Для полного руководства по использованию серверных утилит, включая подробные объяснения и продвинутые сценарии, ознакомьтесь с [дополнительной главой о серверных утилитах](./docs/data-preparation.md).

### Кастомные блоки

Конструктор страниц позволяет использовать блоки, определённые пользователем в своём приложении. Блоки — это обычные React-компоненты.

Чтобы передать кастомные блоки в конструктор:

1. Создайте блок в своём приложении.

2. В вашем коде создайте объект, где ключом является тип блока (строка), а значением — импортированный компонент блока.

3. Передайте созданный объект в параметр `custom.blocks`, `custom.headers` или `custom.subBlocks` компонента `PageConstructor` (`custom.headers` указывает на заголовки блоков, которые будут отображаться отдельно над общим контентом).

4. Теперь вы можете использовать созданный блок в входных данных (параметр `content`), указав его тип и данные.

Чтобы использовать миксины и переменные стилей конструктора при создании кастомных блоков, добавьте импорт в ваш файл:

```css
@import '~@gravity-ui/page-constructor/styles/styles.scss';
```

Чтобы использовать шрифт по умолчанию, добавьте импорт в ваш файл:

```css
@import '~@gravity-ui/page-constructor/styles/fonts.scss';
```

### Загружаемые блоки

Иногда необходимо, чтобы блок рендерился на основе данных, которые нужно загрузить. В этом случае используются загружаемые блоки.

Чтобы добавить кастомные `loadable`-блоки, передайте в `PageConstructor` свойство `custom.loadable` с именами источников данных (строка) для компонента в качестве ключа и объектом в качестве значения.

```typescript
export interface LoadableConfigItem {
  fetch: FetchLoadableData; // метод загрузки данных
  component: React.ComponentType; // компонент, куда передать загруженные данные
}

type FetchLoadableData<TData = any> = (blockKey: string) => Promise<TData>;
```

### Сетка

Конструктор страниц использует сетку `bootstrap` и её реализацию на основе React-компонентов, которые вы можете использовать в своём проекте (в том числе отдельно от конструктора).

Пример использования:

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

### Навигация

Навигацию страниц также можно использовать отдельно от конструктора:

```jsx
import {Navigation} from '@gravity-ui/page-constructor';

const Page= ({data, logo}: React.PropsWithChildren<PageProps>) => <Navigation data={data} logo={logo} />;
```

### Блоки

Каждый блок — это атомарный компонент верхнего уровня. Они хранятся в директории `src/units/constructor/blocks`.

### Саб-блоки

Саб-блоки — это компоненты, которые можно использовать в свойстве `children` блока. В конфигурации указывается список дочерних компонентов из саб-блоков. После рендеринга эти саб-блоки передаются в блок как `children`.

### Как добавить новый блок в `page-constructor`

1. В директории `src/blocks` или `src/sub-blocks` создайте папку с кодом блока или саб-блока.

2. Добавьте имя блока или саб-блока в enum `BlockType` или `SubBlockType` и опишите его свойства в файле `src/models/constructor-items/blocks.ts` или `src/models/constructor-items/sub-blocks.ts` аналогично существующим.

3. Добавьте экспорт для блока в файл `src/blocks/index.ts` и для саб-блока в `src/sub-blocks/index.ts`.

4. Добавьте новый компонент или блок в маппинг в `src/constructor-items.ts`.

5. Добавьте валидатор для нового блока:

   - Добавьте файл `schema.ts` в директорию блока или саб-блока. В этом файле опишите валидатор параметров компонента в формате [`json-schema`](http://json-schema.org/).
   - Экспортируйте его в файле `schema/validators/blocks.ts` или `schema/validators/sub-blocks.ts`.
   - Добавьте его в `enum` или `selectCases` в файле `schema/index.ts`.

6. В директории блока добавьте файл `README.md` с описанием входных параметров.

7. В директории блока добавьте демо для Storybook в папку `__stories__`. Всё демо-содержимое для story должно быть размещено в `data.json` в директории story. Общий `Story` должен принимать тип пропсов блока, иначе в Storybook будут отображаться некорректные пропсы блока.

8. Добавьте шаблон данных блока в папку `src/editor/data/templates/`, имя файла должно соответствовать типу блока.

9. (опционально) Добавьте иконку превью блока в папку `src/editor/data/previews/`, имя файла должно соответствовать типу блока.

### Темы

`PageConstructor` позволяет использовать темы: вы можете устанавливать разные значения для отдельных свойств блоков в зависимости от выбранной в приложении темы.

Чтобы добавить тему к свойству блока:

1. В файле `models/blocks.ts` определите тип соответствующего свойства блока с помощью generics `ThemeSupporting<T>`, где `T` — тип свойства.

2. В файле с React-компонентом блока получите значение свойства с темой через `getThemedValue` и хук `useTheme` (см. примеры в блоке `MediaBlock.tsx`).

3. Добавьте поддержку темы в валидатор свойства: в файле `schema.ts` блока оберните это свойство в `withTheme`.

### i18n

`page-constructor` — это библиотека на базе `uikit`, и мы используем экземпляр `i18n` из uikit. Чтобы настроить интернационализацию, достаточно использовать `configure` из uikit:

```typescript
import {configure} from '@gravity-ui/uikit';

configure({
  lang: 'ru',
});
```

### Карты

Чтобы использовать карты, укажите тип карты, `scriptSrc` и `apiKey` в поле `mapContext` в `PageConstructorProvider`.

Вы можете определить переменные окружения для dev-режима в файле `.env.development` в корне проекта.
`STORYBOOK_GMAP_API_KEY` — apiKey для Google Maps.

### Аналитика

#### Инициализация

Чтобы начать использовать любую аналитику, передайте обработчик в конструктор. Обработчик должен быть создан на стороне проекта. Обработчик будет получать объекты событий `default` и `custom`. Переданный обработчик будет срабатывать при кликах на кнопки, ссылки, навигацию и элементы управления. Поскольку один обработчик используется для обработки всех событий, обратите внимание на то, как обрабатывать разные события при его создании. Есть предопределённые поля, которые помогут вам построить сложную логику.

Передайте `autoEvents: true` в конструктор, чтобы автоматически срабатывали настроенные события.

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

Объект события имеет только одно обязательное поле — `name`. Также у него есть предопределённые поля, которые помогают управлять сложной логикой. Например, `counter.include` позволяет отправлять событие в конкретный счётчик, если в проекте используется несколько систем аналитики.

```ts
type AnalyticsEvent<T = {}> = T & {
  name: string;
  type?: string;
  counters?: AnalyticsCounters;
  context?: string;
};
```

Возможно настроить тип события, необходимый для проекта.

```ts
type MyEventType = AnalyticsEvent<{
  [key: string]?: string; // поддерживается только тип 'string'
}>;
```

#### Селектор счётчика

Возможно настроить событие, указав, в какую систему аналитики его отправлять.

```ts
type AnalyticsCounters = {
  include?: string[]; // массив ID счётчиков аналитики, которые будут применены
  exclude?: string[]; // массив ID счётчиков аналитики, которые не будут применены
};
```

#### Параметр context

Передайте значение `context`, чтобы указать место в проекте, где срабатывает событие.

Используйте селектор ниже или создайте логику, подходящую для нужд проекта.

```ts
// analyticsHandler.ts
if (isCounterAllowed(counterName, counters)) {
  analyticsCounter.reachGoal(counterName, name, parameters);
}
```

#### Зарезервированные типы событий

Несколько предопределённых типов событий используются для пометки автоматически настроенных событий. Используйте эти типы для фильтрации событий по умолчанию, например.

```ts
enum PredefinedEventTypes {
  Default = 'default-event', // события по умолчанию, которые срабатывают при каждом клике на кнопку
  Play = 'play', // событие React-плеера
  Stop = 'stop', // событие React-плеера
}
```

## Разработка

```bash
npm ci
npm run dev
```

#### Примечание о Vite

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

Для Vite необходимо установить плагин `vite-plugin-dynamic-import` и настроить конфигурацию так, чтобы динамические импорты работали.

## Процесс релиза

В обычных случаях мы используем два типа коммитов:

1. fix: коммит типа fix исправляет ошибку в кодовой базе (это соответствует PATCH в Semantic Versioning).
2. feat: коммит типа feat добавляет новую функцию в кодовую базу (это соответствует MINOR в Semantic Versioning).
3. BREAKING CHANGE: коммит с футером BREAKING CHANGE: или с ! после типа/области вводит breaking-изменение API (соответствует MAJOR в Semantic Versioning). BREAKING CHANGE может быть частью коммитов любого типа.
4. Чтобы вручную установить версию пакета релиза, добавьте `Release-As: <version>` в сообщение коммита, например.

```bash
git commit -m 'chore: bump release

Release-As: 1.2.3'
```

Подробную информацию можно найти [здесь](https://www.conventionalcommits.org/en/v1.0.0/).

Когда вы получите одобрение вашего pull request от владельцев кода и пройдёте все проверки, выполните следующие шаги:

1. Проверьте, есть ли pull request от робота с изменениями от другого контрибьютора (он выглядит как `chore(main): release 0.0.0`). Если он существует, выясните, почему он не слит. Если контрибьютор согласен на совместный релиз версии, перейдите к следующему шагу. Если нет, попросите его слить свою версию, затем перейдите к следующему шагу.
2. Squash и merge ваш PR (важно слить новую версию с помощью Github Actions).
3. Дождитесь, пока робот создаст PR с новой версией пакета и информацией о ваших изменениях в CHANGELOG.md. Процесс можно увидеть на [вкладке Actions](https://github.com/gravity-ui/page-constructor/actions).
4. Проверьте свои изменения в CHANGELOG.md и одобрите PR робота.
5. Squash и merge PR. Процесс релиза можно увидеть на [вкладке Actions](https://github.com/gravity-ui/page-constructor/actions).

### Релиз альфа-версий

Если вы хотите слить альфа-версию пакета из своей ветки, вы можете сделать это вручную:

1. Перейдите на вкладку Actions.
2. Выберите workflow "Release alpha version" слева.
3. Справа вы увидите кнопку "Run workflow". Здесь можно выбрать ветку.
4. Также есть поле для ручной версии. Если это первый альфа-релиз из вашей ветки, ничего не устанавливайте. После первого релиза вы должны вручную указать новую версию, поскольку package.json не меняется, если ветка может устареть很快. Используйте префикс `alpha` в ручной версии, иначе получите ошибку.
5. Нажмите "Run workflow" и дождитесь завершения действия. Вы можете выпускать столько версий, сколько нужно, но не злоупотребляйте этим и выпускайте версии только при реальной необходимости. В других случаях используйте [npm pack](https://docs.npmjs.com/cli/v7/commands/npm-pack).

### Релиз бета-версий major

Если вы хотите слить новую major-версию, вам, вероятно, понадобятся бета-версии перед стабильной. Выполните следующие шаги:

1. Создайте или обновите ветку `beta`.
2. Добавьте туда свои изменения.
3. Когда вы будете готовы к новой бета-версии, слейте её вручную с пустым коммитом (или добавьте это сообщение с футером к последнему коммиту):

```bash
git commit -m 'fix: last commit

Release-As: 3.0.0-beta.0' --allow-empty
```

4. Робот создаст новый PR в ветку `beta` с обновлённым CHANGELOG.md и повышенной версией пакета.
5. Вы можете повторять это столько раз, сколько нужно. Когда вы будете готовы слить последнюю major-версию без бета-тега, создайте PR из ветки `beta` в `main`. Обратите внимание, что версия пакета с бета-тегом — это нормально. Робот знает об этом и правильно изменит её. `3.0.0-beta.0` станет `3.0.0`.

### Процесс релиза для предыдущих major-версий

Если вы хотите слить новую версию в предыдущую major после коммита в main, выполните следующие шаги:

1. Обновите необходимую ветку. Названия веток предыдущих major-релизов:
   1. `version-1.x.x/fixes` — для major 1.x.x
   2. `version-2.x.x` — для major 2.x.x
2. Создайте новую ветку от ветки предыдущего major-релиза.
3. Cherry-pick ваш коммит из ветки `main`.
4. Создайте PR, получите одобрение и слейте в ветку предыдущего major-релиза.
5. Squash и merge ваш PR (важно слить новую версию с помощью Github Actions).
6. Дождитесь, пока робот создаст PR с новой версией пакета и информацией о ваших изменениях в CHANGELOG.md. Процесс можно увидеть на [вкладке Actions](https://github.com/gravity-ui/page-constructor/actions).
7. Проверьте свои изменения в CHANGELOG.md и одобрите PR робота.
8. Squash и merge PR. Процесс релиза можно увидеть на [вкладке Actions](https://github.com/gravity-ui/page-constructor/actions).

## Редактор конструктора страниц

Редактор предоставляет пользовательский интерфейс для управления содержимым страницы с предпросмотром в реальном времени.

Как использовать:

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

## Банк памяти

Этот проект включает в себя всесторонний **Memory Bank** — коллекцию файлов документации в формате Markdown, которые предоставляют подробную информацию об архитектуре проекта, компонентах и шаблонах использования. Memory Bank особенно полезен при работе с ИИ-агентами, поскольку содержит структурированную информацию о:

- **Обзор проекта**: Основные требования, цели и контекст
- **Документация компонентов**: Подробные руководства по использованию всех компонентов
- **Архитектура системы**: Технические шаблоны и дизайнерские решения
- **Прогресс разработки**: Текущее состояние и детали реализации

### Использование Memory Bank

Memory Bank находится в директории `memory-bank/` и состоит из обычных файлов Markdown, которые можно читать как любую другую документацию:

- `projectbrief.md` — Основной документ с ключевыми требованиями
- `productContext.md` — Цель проекта и цели пользовательского опыта
- `systemPatterns.md` — Архитектура и технические решения
- `techContext.md` — Технологии, настройка и ограничения
- `activeContext.md` — Текущий фокус работы и недавние изменения
- `progress.md` — Статус реализации и известные проблемы
- `usage/` — Документация по использованию конкретных компонентов
- `storybookComponents.md` — Детали интеграции с Storybook

### Для ИИ-агентов

При работе с ИИ-агентами над этим проектом Memory Bank служит всесторонней базой знаний, которая помогает агентам понять:

- Структура проекта и шаблоны
- API компонентов и примеры использования
- Рабочие процессы разработки и лучшие практики
- Текущее состояние реализации и следующие шаги

ИИ-агенты могут читать эти файлы, чтобы быстро войти в контекст проекта и принимать более обоснованные решения по изменениям кода и реализации.

## Тесты

Подробная документация доступна по [ссылке](./test-utils/docs/README.md).