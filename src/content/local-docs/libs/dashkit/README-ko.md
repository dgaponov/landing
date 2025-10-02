# @gravity-ui/dashkit · [![npm package](https://img.shields.io/npm/v/@gravity-ui/dashkit)](https://www.npmjs.com/package/@gravity-ui/dashkit) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/dashkit/.github/workflows/ci.yaml?branch=main&label=CI&logo=github)](https://github.com/gravity-ui/dashkit/actions/workflows/ci.yaml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/dashkit/)

# DashKit

대시보드 그리드를 렌더링하는 라이브러리입니다.

## 설치

```bash
npm i @gravity-ui/dashkit @gravity-ui/uikit
```

## 설명

이 라이브러리는 위젯을 그리드에 배치하고, 크기를 조정하며, 새 위젯을 추가하고 삭제하는 데 사용됩니다.
위젯은 React 컴포넌트입니다. 예를 들어, 텍스트, 그래픽, 이미지 등입니다.

새 위젯은 플러그인 시스템을 통해 추가됩니다.

### 플러그인

플러그인은 사용자 지정 위젯을 생성하는 데 필요합니다.

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

- **config**: [config](#Config).
- **editMode**: 편집 모드가 활성화되었는지 여부.
- **onItemEdit**: 위젯 편집을 클릭할 때 호출됩니다.
- **onChange**: config 또는 [itemsStateAndParams](#itemsStateAndParams)가 변경될 때 호출됩니다.
- **onDrop**: (#DashKitDnDWrapper)를 사용해 ActionPanel에서 아이템이 드롭될 때 호출됩니다.
- **onItemMountChange**: 아이템 마운트 상태가 변경될 때 호출됩니다.
- **onItemRender**: 아이템 렌더링이 완료될 때 호출됩니다.
- **defaultGlobalParams**, **globalParams**: 모든 위젯에 영향을 미치는 [매개변수](#Params). DataLens에서 `defaultGlobalParams`는 대시보드 설정에서 지정된 전역 매개변수입니다. `globalParams`는 URL에서 설정할 수 있는 전역 매개변수입니다.
- **itemsStateAndParams**: [itemsStateAndParams](#itemsStateAndParams).
- **settings**: DashKit 설정.
- **context**: 모든 위젯에 전달될 객체.
- **overlayControls**: 편집 시 위젯 컨트롤을 재정의하는 객체. 전달되지 않으면 기본 컨트롤이 표시됩니다. `null`을 전달하면 닫기 버튼 또는 사용자 지정 메뉴만 표시됩니다.
- **overlayMenuItems**: 사용자 지정 드롭다운 메뉴 아이템.
- **noOverlay**: `true`인 경우 편집 중에 오버레이와 컨트롤이 표시되지 않습니다.
- **focusable**: `true`인 경우 그리드 아이템이 포커스 가능합니다.
- **onItemFocus**: `focusable`이 true이고 아이템이 포커스될 때 호출됩니다.
- **onItemBlur**: `focusable`이 true이고 아이템이 포커스를 잃을 때 호출됩니다.
- **draggableHandleClassName**: 위젯을 드래그할 수 있게 하는 요소의 CSS 클래스 이름.
- **onDragStart**: ReactGridLayout에서 아이템 드래그가 시작될 때 호출됩니다.
- **onDrag**: 아이템 드래그 중에 ReactGridLayout에서 호출됩니다.
- **onDragStop**: 아이템 드래그가 중지될 때 ReactGridLayout에서 호출됩니다.
- **onResizeStart**: 아이템 크기 조정이 시작될 때 ReactGridLayout에서 호출됩니다.
- **onResize**: 아이템 크기 조정 중에 ReactGridLayout에서 호출됩니다.
- **onResizeStop**: 아이템 크기 조정이 중지될 때 ReactGridLayout에서 호출됩니다.
- **getPreparedCopyItemOptions**: 복사된 아이템을 localStorage에 저장하기 전에 직렬화 가능한 객체로 변환할 때 호출됩니다. 더 이상 사용되지 않는 `context.getPreparedCopyItemOptions` prop 대신 사용해야 합니다.
- **onCopyFulfill**: 아이템 복사가 완료될 때 호출됩니다. 성공 시 `error=null`과 정의된 `data`로, 실패 시 `error: Error`와 `data` 없이 호출됩니다.

## 사용법

### DashKit 구성

`DashKit`를 React 컴포넌트로 사용하기 전에 구성해야 합니다.

- 언어 설정

  ```js
  import {configure, Lang} from '@gravity-ui/uikit';

  configure({lang: Lang.En});
  ```

- DashKit.setSettings

  위젯 간 여백, 기본 위젯 크기, 위젯 오버레이 메뉴 등의 전역 DashKit 설정에 사용됩니다.

  ```js
  import {DashKit} from '@gravity-ui/dashkit';

  DashKit.setSettings({
    gridLayout: {margin: [8, 8]},
    isMobile: true,
    // menu: [] as Array<MenuItem>,
  });
  ```

- DashKit.registerPlugins

  플러그인 등록 및 구성

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
  salt: string; // 고유 ID 형성을 위한 값
  counter: number; // 고유 ID 형성을 위한 값, 증가만 함
  items: ConfigItem[]; // 초기 위젯 상태
  layout: ConfigLayout[]; // 그리드上的 위젯 위치 https://github.com/react-grid-layout
  aliases: ConfigAliases; // 매개변수 별칭, #Params 참조
  connections: ConfigConnection[]; // 위젯 간 연결, #Params 참조
}
```

Config 예시:

```ts
import {DashKitProps} from '@gravity-ui/dashkit';

```

### 구성에 새 항목 추가하기

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

### 구성에서 기존 항목 변경하기

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

### 구성에서 항목 삭제하기

```ts
import {DashKitProps} from '@gravity-ui/dashkit';

const oldItemsStateAndParams: DashKitProps['itemsStateAndParams'] = {};

const {config: newConfig, itemsStateAndParams} = DashKit.removeItem({
  id: 'tT', // item.id
  config: config,
  itemsStateAndParams: this.state.itemsStateAndParams,
});
```

### 매개변수(Params)

```ts
type Params = Record<string, string | string[]>;
```

`DashKit`은 위젯, 링크 및 별칭에 대한 기본 매개변수에 따라 매개변수를 생성합니다. 이러한 매개변수는 [ChartKit](https://github.com/gravity-ui/chartkit) 라이브러리에 필요합니다.

생성 순서:

1. `defaultGlobalParams`
2. 기본 위젯 매개변수 `item.default`
3. `globalParams`
4. 큐에 따라 [itemsStateAndParams](#itemsStateAndParams)에서 가져온 매개변수.

### itemsStateAndParams

위젯 매개변수와 상태를 저장하며 매개변수 변경 큐를 관리하는 객체입니다. 큐와 메타 정보를 저장하기 위한 `__meta__` 필드가 있습니다.

```ts
interface StateAndParamsMeta = {
    __meta__: {
        queue: {id: string}[]; // queue
        version: number; // current version itemsStateAndParams
    };
}
```

그리고 위젯 상태와 매개변수:

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

### 메뉴(Menu)

편집 모드에서 DashKit 위젯의 사용자 지정 오버레이 메뉴를 지정할 수 있습니다.

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

### ActionPanel에서 드래그 가능한 항목(Draggable items from ActionPanel)

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

- **dragImageSrc**: 드래그 이미지 미리보기. 기본적으로 투명한 1px PNG base64를 사용합니다.
- **onDragStart**: ActionPanel에서 요소가 드래그될 때 호출되는 콜백입니다.
- **onDragEnd**: 요소가 드롭되거나 드래그가 취소될 때 호출되는 콜백입니다.

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


#### 예시:

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

| 이름                                           | 설명                  |
| :--------------------------------------------- | :-------------------- |
| 액션 패널 변수                                 |                       |
| `--dashkit-action-panel-color`                 | 배경 색상             |
| `--dashkit-action-panel-border-color`          | 테두리 색상           |
| `--dashkit-action-panel-border-radius`         | 테두리 반경           |
| 액션 패널 아이템 변수                          |                       |
| `--dashkit-action-panel-item-color`            | 배경 색상             |
| `--dashkit-action-panel-item-text-color`       | 텍스트 색상           |
| `--dashkit-action-panel-item-color-hover`      | 호버 배경 색상        |
| `--dashkit-action-panel-item-text-color-hover` | 호버 텍스트 색상      |
| 오버레이 변수                                  |                       |
| `--dashkit-overlay-border-color`               | 테두리 색상           |
| `--dashkit-overlay-color`                      | 배경 색상             |
| `--dashkit-overlay-opacity`                    | 불투명도               |
| 그리드 아이템 변수                              |                       |
| `--dashkit-grid-item-edit-opacity`             | 불투명도               |
| `--dashkit-grid-item-border-radius`            | 테두리 반경           |
| 플레이스홀더 변수                               |                       |
| `--dashkit-placeholder-color`                  | 배경 색상             |
| `--dashkit-placeholder-opacity`                | 불투명도               |

#### 사용 예시

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

## 개발

### 빌드 및 감시

- 종속성 빌드 `npm ci`
- 프로젝트 빌드 `npm run build`
- 스토리북 빌드 `npm run start`

기본적으로 스토리북은 `http://localhost:7120/`에서 실행됩니다.
프로젝트의 변경 사항이 스토리북 실행 중에 항상 반영되지 않으므로, 프로젝트를 수동으로 다시 빌드하고 스토리북을 재시작하는 것이 좋습니다.


### 개발 서버를 위한 nginx 설정 예시

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