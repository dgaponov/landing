# @gravity-ui/dashkit · [![npm package](https://img.shields.io/npm/v/@gravity-ui/dashkit)](https://www.npmjs.com/package/@gravity-ui/dashkit) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/dashkit/.github/workflows/ci.yaml?branch=main&label=CI&logo=github)](https://github.com/gravity-ui/dashkit/actions/workflows/ci.yaml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/dashkit/)

# DashKit

Eine Bibliothek zum Rendern von Dashboard-Gittern.

## Installation

```bash
npm i @gravity-ui/dashkit @gravity-ui/uikit
```

## Beschreibung

Die Bibliothek dient dazu, Widgets in einem Gitter anzuordnen, sie zu ändern, neue hinzuzufügen und zu löschen.
Ein Widget ist eine React-Komponente. Zum Beispiel Text, Diagramme oder Bilder.

Neue Widgets werden über ein Plugin-System hinzugefügt.

### Plugins

Plugins sind erforderlich, um benutzerdefinierte Widgets zu erstellen.

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

- **config**: [Config](#Config).
- **editMode**: Gibt an, ob der Bearbeitungsmodus aktiviert ist.
- **onItemEdit**: Wird aufgerufen, wenn auf ein Widget zum Bearbeiten geklickt wird.
- **onChange**: Wird aufgerufen, wenn die Config oder [itemsStateAndParams](#itemsStateAndParams) geändert werden.
- **onDrop**: Wird aufgerufen, wenn ein Element aus dem ActionPanel mit (#DashKitDnDWrapper) abgelegt wird.
- **onItemMountChange**: Wird aufgerufen, wenn sich der Mount-Status eines Elements ändert.
- **onItemRender**: Wird aufgerufen, wenn das Rendern eines Elements abgeschlossen ist.
- **defaultGlobalParams**, **globalParams**: [Parameter](#Params), die alle Widgets beeinflussen. In DataLens sind `defaultGlobalParams` globale Parameter, die in den Dashboard-Einstellungen festgelegt werden. `globalParams` sind globale Parameter, die in der URL gesetzt werden können.
- **itemsStateAndParams**: [itemsStateAndParams](#itemsStateAndParams).
- **settings**: Einstellungen für DashKit.
- **context**: Ein Objekt, das an alle Widgets weitergegeben wird.
- **overlayControls**: Ein Objekt, das die Widget-Steuerelemente während der Bearbeitung überschreibt. Wenn es nicht übermittelt wird, werden die Standardsteuerelemente angezeigt. Wenn `null` übergeben wird, wird nur der Schließen-Button oder ein benutzerdefiniertes Menü angezeigt.
- **overlayMenuItems**: Benutzerdefinierte Dropdown-Menüelemente.
- **noOverlay**: Wenn `true`, werden Overlay und Steuerelemente während der Bearbeitung nicht angezeigt.
- **focusable**: Wenn `true`, sind die Gitterelemente fokussierbar.
- **onItemFocus**: Wird aufgerufen, wenn `focusable` auf `true` steht und ein Element fokussiert wird.
- **onItemBlur**: Wird aufgerufen, wenn `focusable` auf `true` steht und ein Element den Fokus verliert.
- **draggableHandleClassName**: CSS-Klassenname des Elements, das das Widget verschiebbar macht.
- **onDragStart**: ReactGridLayout wird aufgerufen, wenn das Ziehen eines Elements beginnt.
- **onDrag**: ReactGridLayout wird aufgerufen, während ein Element gezogen wird.
- **onDragStop**: ReactGridLayout wird aufgerufen, wenn das Ziehen eines Elements endet.
- **onResizeStart**: ReactGridLayout wird aufgerufen, wenn das Ändern der Größe eines Elements beginnt.
- **onResize**: ReactGridLayout wird aufgerufen, während die Größe eines Elements geändert wird.
- **onResizeStop**: ReactGridLayout wird aufgerufen, wenn das Ändern der Größe eines Elements endet.
- **getPreparedCopyItemOptions**: Wird aufgerufen, um ein kopiertes Element in ein serialisierbares Objekt umzuwandeln, bevor es in localStorage gespeichert wird. Es sollte anstelle des veralteten Props `context.getPreparedCopyItemOptions` verwendet werden.
- **onCopyFulfill**: Wird aufgerufen, wenn das Kopieren eines Elements abgeschlossen ist – mit `error=null` und definiertem `data` bei erfolgreicher Ausführung oder mit `error: Error` ohne `data` im Fehlerfall.

## Verwendung

### DashKit-Konfiguration

Bevor `DashKit` als React-Komponente verwendet wird, muss es konfiguriert werden.

- Sprache festlegen

  ```js
  import {configure, Lang} from '@gravity-ui/uikit';

  configure({lang: Lang.En});
  ```

- DashKit.setSettings

  Wird für globale DashKit-Einstellungen verwendet (z. B. Abstände zwischen Widgets, Standardgrößen von Widgets und Overlay-Menü für Widgets).

  ```js
  import {DashKit} from '@gravity-ui/dashkit';

  DashKit.setSettings({
    gridLayout: {margin: [8, 8]},
    isMobile: true,
    // menu: [] as Array<MenuItem>,
  });
  ```

- DashKit.registerPlugins

  Registrierung und Konfiguration von Plugins

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
  salt: string; // zur Erstellung einer eindeutigen ID
  counter: number; // zur Erstellung einer eindeutigen ID, wird nur erhöht
  items: ConfigItem[]; // anfängliche Widget-Zustände
  layout: ConfigLayout[]; // Widget-Position auf dem Gitter https://github.com/react-grid-layout
  aliases: ConfigAliases; // Aliase für Parameter siehe #Params
  connections: ConfigConnection[]; // Verbindungen zwischen Widgets siehe #Params
}
```

Config-Beispiel:

```ts
import {DashKitProps} from '@gravity-ui/dashkit';
```

```typescript
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

Fügen Sie ein neues Element zur Konfiguration hinzu:

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

Ändern Sie ein bestehendes Element in der Konfiguration:

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

Löschen Sie ein Element aus der Konfiguration:

```ts
import {DashKitProps} from '@gravity-ui/dashkit';

const oldItemsStateAndParams: DashKitProps['itemsStateAndParams'] = {};

const {config: newConfig, itemsStateAndParams} = DashKit.removeItem({
  id: 'tT', // item.id
  config: config,
  itemsStateAndParams: this.state.itemsStateAndParams,
});
```

### Parameter

```ts
type Params = Record<string, string | string[]>;
```

`DashKit` generiert Parameter basierend auf den Standardparametern für Widgets, Links und Aliase. Diese Parameter sind für die [ChartKit](https://github.com/gravity-ui/chartkit)-Bibliothek erforderlich.

Generierungsreihenfolge:

1. `defaultGlobalParams`
2. Standard-Widget-Parameter `item.default`
3. `globalParams`
4. Parameter aus [itemsStateAndParams](#itemsStateAndParams) gemäß der Warteschlange.

### itemsStateAndParams

Objekt, das Widget-Parameter und -Zustände sowie eine Warteschlange für Parameteränderungen speichert.
Es enthält ein `__meta__`-Feld zur Speicherung der Warteschlange und Meta-Informationen.

```ts
interface StateAndParamsMeta = {
    __meta__: {
        queue: {id: string}[]; // queue
        version: number; // current version itemsStateAndParams
    };
}
```

Sowie Widget-Zustände und -Parameter:

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

### Menü

Sie können ein benutzerdefiniertes Overlay-Menü für DashKit-Widgets im Bearbeitungsmodus angeben.

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

### Ziehbarer Elemente aus dem ActionPanel

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

- **dragImageSrc**: Drag-Vorschau-Bild, standardmäßig wird ein transparentes 1px-PNG-Base64 verwendet
- **onDragStart**: Rückruf, der aufgerufen wird, wenn ein Element aus dem ActionPanel gezogen wird
- **onDragEnd**: Rückruf, der aufgerufen wird, wenn ein Element abgelegt oder der Drag-Vorgang abgebrochen wird
```

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


#### Beispiel:

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

### CSS-API

| Name                                           | Beschreibung          |
| :--------------------------------------------- | :-------------------- |
| Action-Panel-Variablen                         |                       |
| `--dashkit-action-panel-color`                 | Hintergrundfarbe      |
| `--dashkit-action-panel-border-color`          | Rahmenfarbe           |
| `--dashkit-action-panel-border-radius`         | Rahmenradius          |
| Action-Panel-Element-Variablen                 |                       |
| `--dashkit-action-panel-item-color`            | Hintergrundfarbe      |
| `--dashkit-action-panel-item-text-color`       | Textfarbe             |
| `--dashkit-action-panel-item-color-hover`      | Hover-Hintergrundfarbe|
| `--dashkit-action-panel-item-text-color-hover` | Hover-Textfarbe       |
| Overlay-Variablen                              |                       |
| `--dashkit-overlay-border-color`               | Rahmenfarbe           |
| `--dashkit-overlay-color`                      | Hintergrundfarbe      |
| `--dashkit-overlay-opacity`                    | Deckkraft             |
| Grid-Element-Variablen                         |                       |
| `--dashkit-grid-item-edit-opacity`             | Deckkraft             |
| `--dashkit-grid-item-border-radius`            | Rahmenradius          |
| Platzhalter-Variablen                          |                       |
| `--dashkit-placeholder-color`                  | Hintergrundfarbe      |
| `--dashkit-placeholder-opacity`                | Deckkraft             |

#### Verwendungsbeispiel

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

## Entwicklung

### Build & Watch

- Installiere Abhängigkeiten: `npm ci`
- Baue das Projekt: `npm run build`
- Starte Storybook: `npm run start`

Standardmäßig läuft Storybook unter `http://localhost:7120/`.
Neue Änderungen aus dem Projekt werden nicht immer automatisch erkannt, wenn Storybook läuft. Es ist daher besser, das Projekt manuell neu zu bauen und Storybook neu zu starten.

### Beispiel für eine nginx-Konfiguration in der Entwicklungsumgebung auf einem Dev-Rechner

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