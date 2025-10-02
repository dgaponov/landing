# @gravity-ui/dashkit · [![npm package](https://img.shields.io/npm/v/@gravity-ui/dashkit)](https://www.npmjs.com/package/@gravity-ui/dashkit) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/dashkit/.github/workflows/ci.yaml?branch=main&label=CI&logo=github)](https://github.com/gravity-ui/dashkit/actions/workflows/ci.yaml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/dashkit/)

# DashKit

Una biblioteca para renderizar grids de paneles de control.

## Instalación

```bash
npm i @gravity-ui/dashkit @gravity-ui/uikit
```

## Descripción

La biblioteca se utiliza para alinear widgets en una cuadrícula, redimensionarlos, agregar nuevos y eliminarlos.
El widget es un componente de React. Por ejemplo, texto, gráficos e imágenes.

Los nuevos widgets se agregan mediante un sistema de plugins.

### Plugins

Los plugins son necesarios para crear widgets personalizados.

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
- **editMode**: Indica si el modo de edición está habilitado.
- **onItemEdit**: Se llama al hacer clic para editar un widget.
- **onChange**: Se llama cuando se modifica el config o [itemsStateAndParams](#itemsStateAndParams).
- **onDrop**: Se llama cuando se suelta un elemento desde el ActionPanel utilizando (#DashKitDnDWrapper).
- **onItemMountChange**: Se llama cuando cambia el estado de montaje del elemento.
- **onItemRender**: Se llama cuando se completa el renderizado del elemento.
- **defaultGlobalParams**, **globalParams**: [Parámetros](#Params) que afectan a todos los widgets. En DataLens, `defaultGlobalParams` son parámetros globales establecidos en la configuración del panel de control. `globalParams` son parámetros globales que se pueden establecer en la URL.
- **itemsStateAndParams**: [itemsStateAndParams](#itemsStateAndParams).
- **settings**: Configuración de DashKit.
- **context**: Objeto que se propagará a todos los widgets.
- **overlayControls**: Objeto que sobrescribe los controles del widget durante la edición. Si no se transmite, se mostrarán los controles básicos. Si se pasa `null`, solo se mostrará el botón de cerrar o un menú personalizado.
- **overlayMenuItems**: Elementos de menú desplegable personalizados.
- **noOverlay**: Si es `true`, no se muestran la superposición ni los controles durante la edición.
- **focusable**: Si es `true`, los elementos de la cuadrícula serán enfocables.
- **onItemFocus**: Se llama cuando `focusable` es true y el elemento recibe el foco.
- **onItemBlur**: Se llama cuando `focusable` es true y el elemento pierde el foco.
- **draggableHandleClassName**: Nombre de la clase CSS del elemento que hace que el widget sea arrastrable.
- **onDragStart**: ReactGridLayout se llama cuando comienza el arrastre del elemento.
- **onDrag**: ReactGridLayout se llama durante el arrastre del elemento.
- **onDragStop**: ReactGridLayout se llama cuando se detiene el arrastre del elemento.
- **onResizeStart**: ReactGridLayout se llama cuando comienza el redimensionamiento del elemento.
- **onResize**: ReactGridLayout se llama durante el redimensionamiento del elemento.
- **onResizeStop**: ReactGridLayout se llama cuando se detiene el redimensionamiento del elemento.
- **getPreparedCopyItemOptions**: Se llama para convertir el elemento copiado en un objeto serializable antes de guardarlo en localStorage. Debe usarse en lugar de la prop obsoleta `context.getPreparedCopyItemOptions`.
- **onCopyFulfill**: Se llama cuando se completa la copia del elemento, con `error=null` y `data` definida en caso de éxito, o con `error: Error` sin `data` en caso de error.

## Uso

### Configuración de DashKit

Antes de usar `DashKit` como un componente de React, debe configurarse.

- Establecer el idioma

  ```js
  import {configure, Lang} from '@gravity-ui/uikit';

  configure({lang: Lang.En});
  ```

- DashKit.setSettings

  Se utiliza para la configuración global de DashKit (como los márgenes entre widgets, tamaños predeterminados de widgets y menú de superposición de widgets).

  ```js
  import {DashKit} from '@gravity-ui/dashkit';

  DashKit.setSettings({
    gridLayout: {margin: [8, 8]},
    isMobile: true,
    // menu: [] as Array<MenuItem>,
  });
  ```

- DashKit.registerPlugins

  Registro y configuración de plugins.

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

Ejemplo de Config:

```ts
import {DashKitProps} from '@gravity-ui/dashkit';

```

### Agregar un nuevo elemento a la configuración

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

### Cambiar un elemento existente en la configuración

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

### Eliminar un elemento de la configuración

```ts
import {DashKitProps} from '@gravity-ui/dashkit';

const oldItemsStateAndParams: DashKitProps['itemsStateAndParams'] = {};

const {config: newConfig, itemsStateAndParams} = DashKit.removeItem({
  id: 'tT', // item.id
  config: config,
  itemsStateAndParams: this.state.itemsStateAndParams,
});
```

### Parámetros

```ts
type Params = Record<string, string | string[]>;
```

`DashKit` genera parámetros según los parámetros predeterminados para widgets, enlaces y alias. Estos parámetros son necesarios para la biblioteca [ChartKit](https://github.com/gravity-ui/chartkit).

Orden de generación:

1. `defaultGlobalParams`
2. Parámetros predeterminados de widgets `item.default`
3. `globalParams`
4. Parámetros de [itemsStateAndParams](#itemsStateAndParams) según la cola.

### itemsStateAndParams

Objeto que almacena parámetros y estados de widgets, así como una cola de cambios de parámetros.
Tiene un campo `__meta__` para almacenar la cola e información meta.

```ts
interface StateAndParamsMeta = {
    __meta__: {
        queue: {id: string}[]; // queue
        version: number; // current version itemsStateAndParams
    };
}
```

Y también estados y parámetros de widgets:

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

### Menú

Puedes especificar un menú superpuesto personalizado para widgets de DashKit en modo de edición

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

### Elementos arrastrables desde ActionPanel

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

- **dragImageSrc**: Vista previa de imagen de arrastre, por defecto se usa un PNG base64 transparente de 1px
- **onDragStart**: Callback llamado cuando se arrastra un elemento desde ActionPanel
- **onDragEnd**: Callback llamado cuando se suelta el elemento o se cancela el arrastre

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


#### Ejemplo:

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

### API de CSS

| Nombre                                           | Descripción           |
| :--------------------------------------------- | :-------------------- |
| Variables del panel de acciones                         |                       |
| `--dashkit-action-panel-color`                 | Color de fondo      |
| `--dashkit-action-panel-border-color`          | Color de borde          |
| `--dashkit-action-panel-border-radius`         | Radio de borde         |
| Variables de elementos del panel de acciones                    |                       |
| `--dashkit-action-panel-item-color`            | Color de fondo       |
| `--dashkit-action-panel-item-text-color`       | Color de texto            |
| `--dashkit-action-panel-item-color-hover`      | Color de fondo al pasar el mouse |
| `--dashkit-action-panel-item-text-color-hover` | Color de texto al pasar el mouse      |
| Variables de superposición                              |                       |
| `--dashkit-overlay-border-color`               | Color de borde          |
| `--dashkit-overlay-color`                      | Color de fondo      |
| `--dashkit-overlay-opacity`                    | Opacidad               |
| Variables de elementos de cuadrícula                            |                       |
| `--dashkit-grid-item-edit-opacity`             | Opacidad               |
| `--dashkit-grid-item-border-radius`            | Radio de borde         |
| Variables de marcador de posición                          |                       |
| `--dashkit-placeholder-color`                  | Color de fondo      |
| `--dashkit-placeholder-opacity`                | Opacidad               |

#### Ejemplo de uso

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

## Desarrollo

### Compilación y vigilancia

- Instalar dependencias `npm ci`
- Compilar el proyecto `npm run build`
- Iniciar Storybook `npm run start`

Por defecto, Storybook se ejecuta en `http://localhost:7120/`.
Los cambios nuevos del proyecto no siempre se detectan cuando Storybook está en ejecución, por lo que es mejor recompilar el proyecto manualmente y reiniciar Storybook.


### Ejemplo de configuración de nginx para desarrollo en una máquina de desarrollo

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