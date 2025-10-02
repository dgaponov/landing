# @gravity-ui/dashkit &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/dashkit)](https://www.npmjs.com/package/@gravity-ui/dashkit) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/dashkit/.github/workflows/ci.yaml?branch=main&label=CI&logo=github)](https://github.com/gravity-ui/dashkit/actions/workflows/ci.yaml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/dashkit/)

# DashKit

一个仪表板网格渲染库。

## 安装

```bash
npm i @gravity-ui/dashkit @gravity-ui/uikit
```

## 描述

该库用于将小部件排列在网格中，支持调整大小、添加新小部件以及删除小部件。
小部件是一个 React 组件，例如文本、图形或图像。

通过插件系统添加新小部件。

### 插件

插件用于创建自定义小部件。

### Props

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

- **config**： [配置](#Config)。
- **editMode**：是否启用编辑模式。
- **onItemEdit**：点击编辑小部件时调用。
- **onChange**：配置或 [itemsStateAndParams](#itemsStateAndParams) 发生变化时调用。
- **onDrop**：使用 (#DashKitDnDWrapper) 从 ActionPanel 拖放小部件时调用。
- **onItemMountChange**：小部件挂载状态变化时调用。
- **onItemRender**：小部件渲染完成时调用。
- **defaultGlobalParams**、**globalParams**： [参数](#Params)，影响所有小部件。在 DataLens 中，`defaultGlobalParams` 是仪表板设置中定义的全局参数。`globalParams` 是可以通过 URL 设置的全局参数。
- **itemsStateAndParams**： [itemsStateAndParams](#itemsStateAndParams)。
- **settings**：DashKit 设置。
- **context**：将传递给所有小部件的对象。
- **overlayControls**：编辑时覆盖小部件控制的对象。如果未传递，将显示基本控制。如果传递 `null`，则仅显示关闭按钮或自定义菜单。
- **overlayMenuItems**：自定义下拉菜单项。
- **noOverlay**：如果为 `true`，则编辑时不显示覆盖层和控制。
- **focusable**：如果为 `true`，网格小部件将可聚焦。
- **onItemFocus**：当 `focusable` 为 true 且小部件获得焦点时调用。
- **onItemBlur**：当 `focusable` 为 true 且小部件失去焦点时调用。
- **draggableHandleClassName**：使小部件可拖拽的元素的 CSS 类名。
- **onDragStart**：ReactGridLayout 在小部件开始拖拽时调用。
- **onDrag**：ReactGridLayout 在小部件拖拽过程中调用。
- **onDragStop**：ReactGridLayout 在小部件拖拽停止时调用。
- **onResizeStart**：ReactGridLayout 在小部件开始调整大小时调用。
- **onResize**：ReactGridLayout 在小部件调整大小过程中调用。
- **onResizeStop**：ReactGridLayout 在小部件调整大小停止时调用。
- **getPreparedCopyItemOptions**：在将复制的小部件转换为可序列化对象并保存到 localStorage 之前调用。它应替代已弃用的 `context.getPreparedCopyItemOptions` 属性。
- **onCopyFulfill**：小部件复制完成时调用。如果操作成功，则 `error=null` 并定义 `data`；否则 `error: Error` 且无 `data`。

## 使用方法

### DashKit 配置

在使用 `DashKit` 作为 React 组件之前，必须先进行配置。

- 设置语言

  ```js
  import {configure, Lang} from '@gravity-ui/uikit';

  configure({lang: Lang.En});
  ```

- DashKit.setSettings

  用于全局 DashKit 设置（例如小部件间距、默认小部件大小和小部件覆盖菜单）

  ```js
  import {DashKit} from '@gravity-ui/dashkit';

  DashKit.setSettings({
    gridLayout: {margin: [8, 8]},
    isMobile: true,
    // menu: [] as Array<MenuItem>,
  });
  ```

- DashKit.registerPlugins

  注册和配置插件

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
  salt: string; // 用于生成唯一 ID
  counter: number; // 用于生成唯一 ID，仅递增
  items: ConfigItem[]; // 初始小部件状态
  layout: ConfigLayout[]; // 小部件在网格中的位置 https://github.com/react-grid-layout
  aliases: ConfigAliases; // 参数别名，参见 #Params
  connections: ConfigConnection[]; // 小部件间链接，参见 #Params
}
```

配置示例：

```ts
import {DashKitProps} from '@gravity-ui/dashkit';
```

### 添加新项目到配置中

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

### 修改配置中的现有项目

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

### 从配置中删除项目

```ts
import {DashKitProps} from '@gravity-ui/dashkit';

const oldItemsStateAndParams: DashKitProps['itemsStateAndParams'] = {};

const {config: newConfig, itemsStateAndParams} = DashKit.removeItem({
  id: 'tT', // item.id
  config: config,
  itemsStateAndParams: this.state.itemsStateAndParams,
});
```

### 参数（Params）

```ts
type Params = Record<string, string | string[]>;
```

`DashKit` 会根据小部件、链接和别名的默认参数生成参数。这些参数是 [ChartKit](https://github.com/gravity-ui/chartkit) 库所需的。

生成顺序：

1. `defaultGlobalParams`
2. 小部件的默认参数 `item.default`
3. `globalParams`
4. 根据队列从 [itemsStateAndParams](#itemsStateAndParams) 中获取的参数。

### itemsStateAndParams

一个用于存储小部件参数和状态以及参数变更队列的对象。
它有一个 `__meta__` 字段，用于存储队列和元信息。

```ts
interface StateAndParamsMeta = {
    __meta__: {
        queue: {id: string}[]; // queue
        version: number; // current version itemsStateAndParams
    };
}
```

以及小部件状态和参数：

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

### 菜单（Menu）

您可以在编辑模式下为 DashKit 小部件指定自定义叠加菜单

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

### 来自 ActionPanel 的可拖拽项目

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

- **dragImageSrc**：拖拽图像预览，默认使用透明的 1px PNG base64
- **onDragStart**：从 ActionPanel 拖拽元素时调用的回调
- **onDragEnd**：元素放置或拖拽取消时调用的回调

```ts
type ItemDragProps = {
    type: string; // Plugin type
    layout?: { // Optional. Layout item size for preview and init
        w?: number;
        h?: number;
    };
    extra?: any; // Custom user context
};
```

```ts
type ItemDropProps = {
    commit: () => void; // Callback should be called after all config operations are made
    dragProps: ItemDragProps; // Item drag props
    itemLayout: ConfigLayout; // Calculated item layout dimensions
    newLayout: ConfigLayout[]; // New layout after element is dropped
};
```


#### 示例：

```jsx
const overlayMenuItems = [
  {
    id: 'chart',
    icon: <Icon data={ChartColumn} />,
    title: 'Chart',
    qa: 'chart',
    dragProps: { // ItemDragProps
        type: 'custom', // Registered plugin type
    },
  }
]

const onDrop = (dropProps: ItemDropProps) => {
  // ... add element to your config
  dropProps.commit();
}

<DashKitDnDWrapper>
  <DashKit editMode={true} config={config} onChange={onChange} onDrop={onDrop} />
  <ActionPanel items={overlayMenuItems} />
</DashKitDnDWrapper>
```

### CSS API

| 名称                                           | 描述           |
| :--------------------------------------------- | :------------ |
| 操作面板变量                                   |                |
| `--dashkit-action-panel-color`                 | 背景颜色       |
| `--dashkit-action-panel-border-color`          | 边框颜色       |
| `--dashkit-action-panel-border-radius`         | 边框圆角       |
| 操作面板项变量                                 |                |
| `--dashkit-action-panel-item-color`            | 背景颜色       |
| `--dashkit-action-panel-item-text-color`       | 文本颜色       |
| `--dashkit-action-panel-item-color-hover`      | 悬停背景颜色   |
| `--dashkit-action-panel-item-text-color-hover` | 悬停文本颜色   |
| 覆盖层变量                                     |                |
| `--dashkit-overlay-border-color`               | 边框颜色       |
| `--dashkit-overlay-color`                      | 背景颜色       |
| `--dashkit-overlay-opacity`                    | 不透明度       |
| 网格项变量                                     |                |
| `--dashkit-grid-item-edit-opacity`             | 不透明度       |
| `--dashkit-grid-item-border-radius`            | 边框圆角       |
| 占位符变量                                     |                |
| `--dashkit-placeholder-color`                  | 背景颜色       |
| `--dashkit-placeholder-opacity`                | 不透明度       |

#### 使用示例

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

## 开发

### 构建与监视

- 安装依赖 `npm ci`
- 构建项目 `npm run build`
- 启动 Storybook `npm run start`

默认情况下，Storybook 运行在 `http://localhost:7120/` 上。
当 Storybook 运行时，项目的新变更并不总是会被自动拾取，因此最好手动重新构建项目并重启 Storybook。


### 开发环境中 Nginx 配置示例

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