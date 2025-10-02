# @gravity-ui/dashkit &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/dashkit)](https://www.npmjs.com/package/@gravity-ui/dashkit) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/dashkit/.github/workflows/ci.yaml?branch=main&label=CI&logo=github)](https://github.com/gravity-ui/dashkit/actions/workflows/ci.yaml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/dashkit/)

# DashKit

一个用于渲染仪表板网格的库。

## 安装

```bash
npm i @gravity-ui/dashkit @gravity-ui/uikit
```

## 描述

该库用于将小部件排列在网格中，支持调整大小、添加新小部件以及删除小部件。
小部件是一个 React 组件，例如文本、图表和图像。

新小部件通过插件系统添加。

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
- **onChange**：当配置或 [itemsStateAndParams](#itemsStateAndParams) 发生变化时调用。
- **onDrop**：使用 (#DashKitDnDWrapper) 从 ActionPanel 拖放小部件时调用。
- **onItemMountChange**：小部件挂载状态变化时调用。
- **onItemRender**：小部件渲染完成时调用。
- **defaultGlobalParams**、**globalParams**： [参数](#Params)，影响所有小部件。在 DataLens 中，`defaultGlobalParams` 是仪表板设置中定义的全局参数。`globalParams` 是可以通过 URL 设置的全局参数。
- **itemsStateAndParams**： [itemsStateAndParams](#itemsStateAndParams)。
- **settings**：DashKit 设置。
- **context**：将传递给所有小部件的对象。
- **overlayControls**：编辑时覆盖小部件控制的对象。如果未传递，将显示基本控制。如果传递 `null`，则仅显示关闭按钮或自定义菜单。
- **overlayMenuItems**：自定义下拉菜单项。
- **noOverlay**：如果为 `true`，则在编辑时不显示覆盖层和控制。
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
- **getPreparedCopyItemOptions**：在将复制的小部件转换为可序列化对象并保存到 localStorage 之前调用。应使用它代替已弃用的 `context.getPreparedCopyItemOptions` 属性。
- **onCopyFulfill**：小部件复制完成时调用。如果操作成功，则 `error=null` 且 `data` 已定义；否则 `error: Error` 且无 `data`。

## 使用

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

向配置添加一个新项：

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
    // 可选。如果新项需要在当前布局中插入，并带有预定义的尺寸
    layout: { // 当前项在 'Ea' 之前插入
      h: 6,
      w: 12,
      x: 0,
      y: 2,
    },,
  },
  config: config,
  options: {
    // 可选。当从 ActionPanel 拖入新元素时，现有点的布局值更新
    updateLayout: newLayout,
  },
});
```

更改配置中的现有项：

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

从配置中删除一个项：

```ts
import {DashKitProps} from '@gravity-ui/dashkit';

const oldItemsStateAndParams: DashKitProps['itemsStateAndParams'] = {};

const {config: newConfig, itemsStateAndParams} = DashKit.removeItem({
  id: 'tT', // item.id
  config: config,
  itemsStateAndParams: this.state.itemsStateAndParams,
});
```

### 参数

```ts
type Params = Record<string, string | string[]>;
```

`DashKit` 根据小部件、链接和别名的默认参数生成参数。这些参数是 [ChartKit](https://github.com/gravity-ui/chartkit) 库所需的。

生成顺序：

1. `defaultGlobalParams`
2. 小部件的默认参数 `item.default`
3. `globalParams`
4. 根据队列从 [itemsStateAndParams](#itemsStateAndParams) 中的参数。

### itemsStateAndParams

用于存储小部件参数和状态以及参数更改队列的对象。
它有一个 `__meta__` 字段，用于存储队列和元信息。

```ts
interface StateAndParamsMeta = {
    __meta__: {
        queue: {id: string}[]; // 队列
        version: number; // 当前 itemsStateAndParams 版本
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

### 菜单

您可以在编辑模式下为 DashKit 小部件指定自定义叠加菜单

```ts
type MenuItem = {
  id: string; // 唯一 ID
  title?: string; // 字符串标题
  icon?: ReactNode; // 图标节点
  iconSize?: number | string; // 图标大小，以像素为单位（数字）或带单位的字符串
  handler?: (item: ConfigItem) => void; // 自定义项操作处理程序
  visible?: (item: ConfigItem) => boolean; // 可选的可见性处理程序，用于过滤菜单项
  className?: string; // 自定义类属性
};

// 在设置中使用菜单项数组
<Dashkit overlayMenuItems={[] as Array<MenuItem> | null} />

[已弃用]
// overlayMenuItems 属性优先级高于 setSettings 菜单
DashKit.setSettings({menu: [] as Array<MenuItem>});
```

### 来自 ActionPanel 的可拖拽项

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

- **dragImageSrc**：拖拽图像预览，默认使用透明 1px PNG base64
- **onDragStart**：从 ActionPanel 拖拽元素时调用的回调
- **onDragEnd**：元素放置或拖拽取消时调用的回调

```ts
type ItemDragProps = {
    type: string; // 插件类型
    layout?: { // 可选。预览和初始化时的布局项大小
        w?: number;
        h?: number;
    };
    extra?: any; // 自定义用户上下文
};
```

```ts
type ItemDropProps = {
    commit: () => void; // 在所有配置操作完成后调用此回调
    dragProps: ItemDragProps; // 项拖拽属性
    itemLayout: ConfigLayout; // 计算后的项布局尺寸
    newLayout: ConfigLayout[]; // 元素放置后的新布局
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
        type: 'custom', // 已注册的插件类型
    },
  }
]

const onDrop = (dropProps: ItemDropProps) => {
  // ... 将元素添加到您的配置中
  dropProps.commit();
}

<DashKitDnDWrapper>
  <DashKit editMode={true} config={config} onChange={onChange} onDrop={onDrop} />
  <ActionPanel items={overlayMenuItems} />
</DashKitDnDWrapper>
```

### CSS API

| 名称                                           | 描述           |
| :--------------------------------------------- | :------------- |
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
当 Storybook 运行时，项目的新更改并不总是会被自动拾取，因此最好手动重新构建项目并重启 Storybook。


### 开发环境中 nginx 配置示例

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