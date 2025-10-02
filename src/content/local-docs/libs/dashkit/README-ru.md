# @gravity-ui/dashkit · [![npm package](https://img.shields.io/npm/v/@gravity-ui/dashkit)](https://www.npmjs.com/package/@gravity-ui/dashkit) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/dashkit/.github/workflows/ci.yaml?branch=main&label=CI&logo=github)](https://github.com/gravity-ui/dashkit/actions/workflows/ci.yaml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/dashkit/)

# DashKit

Библиотека для рендеринга сетки дашборда.

## Установка

```bash
npm i @gravity-ui/dashkit @gravity-ui/uikit
```

## Описание

Библиотека используется для выстраивания виджетов в сетку, изменения их размера, добавления новых и удаления существующих.
Виджет — это React-компонент. Например, текст, графики или изображения.

Новые виджеты добавляются через систему плагинов.

### Плагины

Плагины необходимы для создания кастомных виджетов.

### Пропсы

```ts
type ItemManipulationCallback = (eventData: {
    layout: Layouts;
    oldItem: Layout;
    newItem: Layout;
    placeholder: Layout;
    e: MouseEvent;
    element: HTMLElement;
}) => void;

interface DashKitProps {
  config: Config;
  editMode: boolean;
  onItemEdit: ({id}: {id: string}) => void;
  onChange: (data: {config: Config; itemsStateAndParams: ItemsStateAndParams}) => void;
  onDrop: (dropProps: ItemDropProps) => void;
  onItemMountChange: (item: ConfigItem, state: {isAsync: boolead; isMounted: boolean}) => void;
  onItemRender: (item: ConfigItem) => void;

  onDragStart?: ItemManipulationCallback;
  onDrag?: ItemManipulationCallback;
  onDragStop?: ItemManipulationCallback;
  onResizeStart?: ItemManipulationCallback;
  onResize?: ItemManipulationCallback;
  onResizeStop?: ItemManipulationCallback;

  defaultGlobalParams: GlobalParams;
  globalParams: GlobalParams;
  itemsStateAndParams: ItemsStateAndParams;
  settings: SettingsProps;
  context: ContextProps;
  overlayControls?: Record<string, OverlayControlItem[]> | null;
  overlayMenuItems?: MenuItems[] | null;
  noOverlay?: boolean;

  focusable?: boolean;
  onItemFocus: (item: ConfigItem) => void;
  onItemBlur: (item: ConfigItem) => void;

  draggableHandleClassName?: string;
  getPreparedCopyItemOptions?: (options: PreparedCopyItemOptions) => PreparedCopyItemOptions;
  onCopyFulfill?: (error: null | Error, data?: PreparedCopyItemOptions) => void;
}
```

- **config**: [config](#Config).
- **editMode**: Включен ли режим редактирования.
- **onItemEdit**: Вызывается при клике для редактирования виджета.
- **onChange**: Вызывается при изменении config или [itemsStateAndParams](#itemsStateAndParams).
- **onDrop**: Вызывается при сбросе элемента из ActionPanel с использованием (#DashKitDnDWrapper).
- **onItemMountChange**: Вызывается при изменении состояния монтирования элемента.
- **onItemRender**: Вызывается после завершения рендеринга элемента.
- **defaultGlobalParams**, **globalParams**: [Параметры](#Params), которые влияют на все виджеты. В DataLens `defaultGlobalParams` — это глобальные параметры, установленные в настройках дашборда. `globalParams` — это глобальные параметры, которые можно установить в URL.
- **itemsStateAndParams**: [itemsStateAndParams](#itemsStateAndParams).
- **settings**: Настройки DashKit.
- **context**: Объект, который будет передан как проп в все виджеты.
- **overlayControls**: Объект, который переопределяет элементы управления виджета во время редактирования. Если не передан, отображаются базовые элементы управления. Если передан `null`, отображается только кнопка закрытия или кастомное меню.
- **overlayMenuItems**: Кастомные элементы выпадающего меню.
- **noOverlay**: Если `true`, во время редактирования не отображаются оверлей и элементы управления.
- **focusable**: Если `true`, элементы сетки будут фокусируемыми.
- **onItemFocus**: Вызывается, когда `focusable` равно `true` и элемент получает фокус.
- **onItemBlur**: Вызывается, когда `focusable` равно `true` и элемент теряет фокус.
- **draggableHandleClassName**: CSS-класс элемента, который делает виджет перетаскиваемым.
- **onDragStart**: Вызывается ReactGridLayout при начале перетаскивания элемента.
- **onDrag**: Вызывается ReactGridLayout во время перетаскивания элемента.
- **onDragStop**: Вызывается ReactGridLayout при завершении перетаскивания элемента.
- **onResizeStart**: Вызывается ReactGridLayout при начале изменения размера элемента.
- **onResize**: Вызывается ReactGridLayout во время изменения размера элемента.
- **onResizeStop**: Вызывается ReactGridLayout при завершении изменения размера элемента.
- **getPreparedCopyItemOptions**: Вызывается для преобразования скопированного элемента в сериализуемый объект перед сохранением в localStorage. Должен использоваться вместо устаревшего пропа `context.getPreparedCopyItemOptions`.
- **onCopyFulfill**: Вызывается после завершения копирования элемента: с `error=null` и определённым `data` при успешной операции или с `error: Error` без `data` в случае ошибки.

## Использование

### Конфигурация DashKit

Перед использованием `DashKit` как React-компонента его необходимо настроить.

- Установка языка

  ```js
  import {configure, Lang} from '@gravity-ui/uikit';

  configure({lang: Lang.En});
  ```

- DashKit.setSettings

  Используется для глобальных настроек DashKit (например, отступы между виджетами, размеры виджетов по умолчанию и меню оверлея виджета).

  ```js
  import {DashKit} from '@gravity-ui/dashkit';

  DashKit.setSettings({
    gridLayout: {margin: [8, 8]},
    isMobile: true,
    // menu: [] as Array<MenuItem>,
  });
  ```

- DashKit.registerPlugins

  Регистрация и настройка плагинов.

  ```js
  import {DashKit} from '@gravity-ui/dashkit';
  import {pluginTitle, pluginText} from '@gravity-ui/dashkit';

  DashKit.registerPlugins(
    pluginTitle,
    pluginText.setSettings({
      apiHandler({text}) {
        return api.getMarkdown(text);
      },
    }),
  );

  DashKit.registerPlugins({
    type: 'custom',
    defaultLayout: {
      w: 10,
      h: 8,
    },
    renderer: function CustomPlugin() {
      return <div>Custom widget with custom controls</div>;
    },
  });
  ```

### Config

```ts
export interface Config {
  salt: string; // to form a unique id
  counter: number; // to form a unique id, only increases
  items: ConfigItem[]; //  initial widget states
  layout: ConfigLayout[]; // widget position on the grid https://github.com/react-grid-layout
  aliases: ConfigAliases; // aliases for parameters see #Params
  connections: ConfigConnection[]; // links between widgets see #Params
}
```

Пример Config:

```ts
import {DashKitProps} from '@gravity-ui/dashkit';

```

```ts
const config: DashKitProps['config'] = {
  salt: '0.46703554571365613',
  counter: 4,
  items: [
    {
      id: 'tT',
      data: {
        size: 'm',
        text: 'Caption',
        showInTOC: true,
      },
      type: 'title',
      namespace: 'default',
      orderId: 1,
    },
    {
      id: 'Ea',
      data: {
        text: 'mode _editActive',
        _editActive: true,
      },
      type: 'text',
      namespace: 'default',
    },
    {
      id: 'zR',
      data: {
        text: '### Text',
      },
      type: 'text',
      namespace: 'default',
      orderId: 0,
    },
    {
      id: 'Dk',
      data: {
        foo: 'bar',
      },
      type: 'custom',
      namespace: 'default',
      orderId: 5,
    },
  ],
  layout: [
    {
      h: 2,
      i: 'tT',
      w: 36,
      x: 0,
      y: 0,
    },
    {
      h: 6,
      i: 'Ea',
      w: 12,
      x: 0,
      y: 2,
    },
    {
      h: 6,
      i: 'zR',
      w: 12,
      x: 12,
      y: 2,
    },
    {
      h: 4,
      i: 'Dk',
      w: 8,
      x: 0,
      y: 8,
    },
  ],
  aliases: {},
  connections: [],
};
```

Добавление нового элемента в конфигурацию:

```ts
const newLayout = updateLayout: [
  {
    h: 6,
    i: 'Ea',
    w: 12,
    x: 0,
    y: 6,
  },
  {
    h: 4,
    i: 'Dk',
    w: 8,
    x: 0,
    y: 12,
  },
];

const newConfig = DashKit.setItem({
  item: {
    data: {
      text: `Some text`,
    },
    namespace: 'default',
    type: 'text',
    // Optional. If new item needed to be inserted in current layout with predefined dimensions
    layout: { // Current item inseterted before 'Ea'
      h: 6,
      w: 12,
      x: 0,
      y: 2,
    },,
  },
  config: config,
  options: {
    // Optional. New layout values for existing items when new element is dropped from ActionPanel
    updateLayout: newLayout,
  },
});
```

Изменение существующего элемента в конфигурации:

```ts
const newConfig = DashKit.setItem({
  item: {
    id: 'tT', // item.id
    data: {
      size: 'm',
      text: `New caption`,
    },
    namespace: 'default',
    type: 'title',
  },
  config: config,
});
```

Удаление элемента из конфигурации:

```ts
import {DashKitProps} from '@gravity-ui/dashkit';

const oldItemsStateAndParams: DashKitProps['itemsStateAndParams'] = {};

const {config: newConfig, itemsStateAndParams} = DashKit.removeItem({
  id: 'tT', // item.id
  config: config,
  itemsStateAndParams: this.state.itemsStateAndParams,
});
```

### Параметры

```ts
type Params = Record<string, string | string[]>;
```

`DashKit` генерирует параметры в соответствии с параметрами по умолчанию для виджетов, ссылок и алиасов. Эти параметры необходимы для библиотеки [ChartKit](https://github.com/gravity-ui/chartkit).

Порядок генерации:

1. `defaultGlobalParams`
2. Параметры виджетов по умолчанию `item.default`
3. `globalParams`
4. Параметры из [itemsStateAndParams](#itemsStateAndParams) в соответствии с очередью.

### itemsStateAndParams

Объект, который хранит параметры и состояния виджетов, а также очередь изменений параметров.
Он содержит поле `__meta__` для хранения очереди и метаинформации.

```ts
interface StateAndParamsMeta = {
    __meta__: {
        queue: {id: string}[]; // queue
        version: number; // current version itemsStateAndParams
    };
}
```

А также состояния и параметры виджетов:

```ts
interface ItemsStateAndParamsBase {
  [itemId: string]: {
    state?: Record<string, any>;
    params?: Params;
  };
}
```

```ts
type ItemsStateAndParams = StateAndParamsMeta & ItemsStateAndParamsBase;
```

### Меню

Вы можете указать пользовательское меню наложения для виджетов DashKit в режиме редактирования

```ts
type MenuItem = {
  id: string; // uniq id
  title?: string; // string title
  icon?: ReactNode; // node of icon
  iconSize?: number | string; // icon size in px as number or as string with units
  handler?: (item: ConfigItem) => void; // custom item action handler
  visible?: (item: ConfigItem) => boolean; // optional visibility handler for filtering menu items
  className?: string; // custom class property
};

// use array of menu items in settings
<Dashkit overlayMenuItems={[] as Array<MenuItem> | null} />

[deprecated]
// overlayMenuItems property has greater priority over setSettings menu
DashKit.setSettings({menu: [] as Array<MenuItem>});
```

### Перетаскиваемые элементы из ActionPanel

#### DashKitDnDWrapper

```ts
type DraggedOverItem = {
  h: number;
  w: number;
  type: string;
  parent: string;
  i?: number;
};

interface DashKitDnDWrapperProps {
  dragImageSrc?: string;
  onDragStart?: (dragProps: ItemDragProps) => void;
  onDragEnd?: () => void;
  onDropDragOver?: (draggedItem: DraggedOverItem, sharedItem: DraggedOverItem | null) => void | boolean;
}
```

- **dragImageSrc**: Предварительный просмотр изображения при перетаскивании, по умолчанию используется прозрачный 1px PNG в base64
- **onDragStart**: Callback, вызываемый при начале перетаскивания элемента из ActionPanel
- **onDragEnd**: Callback, вызываемый при сбросе элемента или отмене перетаскивания

```ts
type ItemDragProps = {
    type: string; // Тип плагина
    layout?: { // Необязательно. Размер элемента макета для предварительного просмотра и инициализации
        w?: number;
        h?: number;
    };
    extra?: any; // Пользовательский контекст
};
```

```ts
type ItemDropProps = {
    commit: () => void; // Колбэк, который должен быть вызван после выполнения всех операций с конфигурацией
    dragProps: ItemDragProps; // Свойства перетаскиваемого элемента
    itemLayout: ConfigLayout; // Рассчитанные размеры макета элемента
    newLayout: ConfigLayout[]; // Новый макет после сброса элемента
};
```


#### Пример:

```jsx
const overlayMenuItems = [
  {
    id: 'chart',
    icon: <Icon data={ChartColumn} />,
    title: 'Chart',
    qa: 'chart',
    dragProps: { // ItemDragProps
        type: 'custom', // Зарегистрированный тип плагина
    },
  }
]

const onDrop = (dropProps: ItemDropProps) => {
  // ... добавьте элемент в вашу конфигурацию
  dropProps.commit();
}

<DashKitDnDWrapper>
  <DashKit editMode={true} config={config} onChange={onChange} onDrop={onDrop} />
  <ActionPanel items={overlayMenuItems} />
</DashKitDnDWrapper>
```

### API CSS

| Название                                      | Описание              |
| :-------------------------------------------- | :------------------- |
| Переменные панели действий                    |                      |
| `--dashkit-action-panel-color`                | Цвет фона            |
| `--dashkit-action-panel-border-color`         | Цвет границы         |
| `--dashkit-action-panel-border-radius`        | Радиус границы       |
| Переменные элементов панели действий          |                      |
| `--dashkit-action-panel-item-color`           | Цвет фона            |
| `--dashkit-action-panel-item-text-color`      | Цвет текста          |
| `--dashkit-action-panel-item-color-hover`     | Цвет фона при наведении |
| `--dashkit-action-panel-item-text-color-hover`| Цвет текста при наведении |
| Переменные оверлея                            |                      |
| `--dashkit-overlay-border-color`              | Цвет границы         |
| `--dashkit-overlay-color`                     | Цвет фона            |
| `--dashkit-overlay-opacity`                   | Непрозрачность       |
| Переменные элементов сетки                    |                      |
| `--dashkit-grid-item-edit-opacity`            | Непрозрачность       |
| `--dashkit-grid-item-border-radius`           | Радиус границы       |
| Переменные заглушки                           |                      |
| `--dashkit-placeholder-color`                 | Цвет фона            |
| `--dashkit-placeholder-opacity`               | Непрозрачность       |

#### Пример использования

```css
.custom-theme-wrapper {
  --dashkit-grid-item-edit-opacit: 1;
  --dashkit-overlay-color: var(--g-color-base-float);
  --dashkit-overlay-border-color: var(--g-color-base-float);
  --dashkit-overlay-opacity: 0.5;

  --dashkit-action-panel-border-color: var(--g-color-line-info);
  --dashkit-action-panel-color: var(--g-color-base-float-accent);
  --dashkit-action-panel-border-radius: var(--g-border-radius-xxl);
}
```

```tsx
// ....

const CustomThemeWrapper = (props: {
  dashkitProps: DashkitProps;
  actionPanelProps: ActionPanelProps;
}) => {
  return (
    <div className="custom-theme-wrapper">
      <Dashkit {...props.dashkitProps} />
      <ActionPanel {...props.actionPanelProps} />
    </div>
  );
};
```

## Разработка

### Сборка и наблюдение

- Установка зависимостей `npm ci`
- Сборка проекта `npm run build`
- Запуск storybook `npm run start`

По умолчанию storybook запускается на `http://localhost:7120/`.
Новые изменения из проекта не всегда подхватываются во время работы storybook, поэтому лучше пересобрать проект вручную и перезапустить storybook.


### Пример конфигурации nginx для разработки на dev-машине

```bash
server {
    server_name dashkit.username.ru;

    include common/ssl;

    access_log /home/username/logs/common.access.log;
    error_log /home/username/logs/common.error.log;

    root /home/username/projects/dashkit;

    location / {
        try_files $uri @node;
    }

    location @node {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://127.0.0.1:7120;
        proxy_redirect off;
    }
}

```