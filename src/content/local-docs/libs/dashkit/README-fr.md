# @gravity-ui/dashkit · [![npm package](https://img.shields.io/npm/v/@gravity-ui/dashkit)](https://www.npmjs.com/package/@gravity-ui/dashkit) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/dashkit/.github/workflows/ci.yaml?branch=main&label=CI&logo=github)](https://github.com/gravity-ui/dashkit/actions/workflows/ci.yaml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/dashkit/)

# DashKit

Une bibliothèque pour le rendu de grilles de tableau de bord.

## Installation

```bash
npm i @gravity-ui/dashkit @gravity-ui/uikit
```

## Description

Cette bibliothèque permet d'aligner des widgets dans une grille, de les redimensionner, d'en ajouter de nouveaux et de les supprimer.
Un widget est un composant React. Par exemple, du texte, des graphiques ou des images.

Les nouveaux widgets sont ajoutés via un système de plugins.

### Plugins

Les plugins sont nécessaires pour créer des widgets personnalisés.

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

- **config** : [config](#Config).
- **editMode** : Indique si le mode édition est activé.
- **onItemEdit** : Appelée lors du clic pour éditer un widget.
- **onChange** : Appelée lorsque le config ou [itemsStateAndParams](#itemsStateAndParams) sont modifiés.
- **onDrop** : Appelée lorsqu'un élément est déposé depuis l'ActionPanel en utilisant (#DashKitDnDWrapper).
- **onItemMountChange** : Appelée lorsque l'état de montage d'un élément change.
- **onItemRender** : Appelée lorsque le rendu d'un élément est terminé.
- **defaultGlobalParams**, **globalParams** : [Paramètres](#Params) qui affectent tous les widgets. Dans DataLens, `defaultGlobalParams` sont les paramètres globaux définis dans les paramètres du tableau de bord. `globalParams` sont les paramètres globaux qui peuvent être définis dans l'URL.
- **itemsStateAndParams** : [itemsStateAndParams](#itemsStateAndParams).
- **settings** : Paramètres de DashKit.
- **context** : Objet qui sera transmis en tant que prop à tous les widgets.
- **overlayControls** : Objet qui remplace les contrôles des widgets pendant l'édition. S'il n'est pas transmis, les contrôles de base s'affichent. S'il est `null`, seul le bouton de fermeture ou un menu personnalisé s'affiche.
- **overlayMenuItems** : Éléments de menu déroulant personnalisés.
- **noOverlay** : Si `true`, l'overlay et les contrôles ne s'affichent pas pendant l'édition.
- **focusable** : Si `true`, les éléments de la grille seront focusables.
- **onItemFocus** : Appelée lorsque `focusable` est true et qu'un élément est focalisé.
- **onItemBlur** : Appelée lorsque `focusable` est true et qu'un élément perd le focus.
- **draggableHandleClassName** : Nom de la classe CSS de l'élément qui rend le widget traînant.
- **onDragStart** : Appelée par ReactGridLayout au début du glisser d'un élément.
- **onDrag** : Appelée par ReactGridLayout pendant le glisser d'un élément.
- **onDragStop** : Appelée par ReactGridLayout à la fin du glisser d'un élément.
- **onResizeStart** : Appelée par ReactGridLayout au début du redimensionnement d'un élément.
- **onResize** : Appelée par ReactGridLayout pendant le redimensionnement d'un élément.
- **onResizeStop** : Appelée par ReactGridLayout à la fin du redimensionnement d'un élément.
- **getPreparedCopyItemOptions** : Appelée pour convertir l'élément copié en objet sérialisable avant de le sauvegarder dans le localStorage. Elle doit être utilisée à la place de la prop dépréciée `context.getPreparedCopyItemOptions`.
- **onCopyFulfill** : Appelée lorsque la copie d'un élément est terminée, avec `error=null` et un `data` défini en cas de succès, ou avec `error: Error` sans `data` sinon.

## Utilisation

### Configuration de DashKit

Avant d'utiliser `DashKit` en tant que composant React, il doit être configuré.

- Définir la langue

  ```js
  import {configure, Lang} from '@gravity-ui/uikit';

  configure({lang: Lang.En});
  ```

- DashKit.setSettings

  Utilisée pour les paramètres globaux de DashKit (tels que les marges entre les widgets, les tailles par défaut des widgets et le menu overlay des widgets).

  ```js
  import {DashKit} from '@gravity-ui/dashkit';

  DashKit.setSettings({
    gridLayout: {margin: [8, 8]},
    isMobile: true,
    // menu: [] as Array<MenuItem>,
  });
  ```

- DashKit.registerPlugins

  Enregistrement et configuration des plugins.

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
  salt: string; // pour former un identifiant unique
  counter: number; // pour former un identifiant unique, ne fait qu'augmenter
  items: ConfigItem[]; // états initiaux des widgets
  layout: ConfigLayout[]; // position des widgets sur la grille https://github.com/react-grid-layout
  aliases: ConfigAliases; // alias pour les paramètres voir #Params
  connections: ConfigConnection[]; // liens entre les widgets voir #Params
}
```

Exemple de config :

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

Ajoutez un nouvel élément à la configuration :

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
    // Optionnel. Si le nouvel élément doit être inséré dans la mise en page actuelle avec des dimensions prédéfinies
    layout: { // L'élément actuel est inséré avant 'Ea'
      h: 6,
      w: 12,
      x: 0,
      y: 2,
    },,
  },
  config: config,
  options: {
    // Optionnel. Nouvelles valeurs de mise en page pour les éléments existants lorsque un nouvel élément est déposé depuis l'ActionPanel
    updateLayout: newLayout,
  },
});
```

Modifiez un élément existant dans la configuration :

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

Supprimez un élément de la configuration :

```ts
import {DashKitProps} from '@gravity-ui/dashkit';

const oldItemsStateAndParams: DashKitProps['itemsStateAndParams'] = {};

const {config: newConfig, itemsStateAndParams} = DashKit.removeItem({
  id: 'tT', // item.id
  config: config,
  itemsStateAndParams: this.state.itemsStateAndParams,
});
```

### Paramètres

```ts
type Params = Record<string, string | string[]>;
```

`DashKit` génère des paramètres en fonction des paramètres par défaut pour les widgets, les liens et les alias. Ces paramètres sont nécessaires pour la bibliothèque [ChartKit](https://github.com/gravity-ui/chartkit).

Ordre de génération :

1. `defaultGlobalParams`
2. Paramètres par défaut des widgets `item.default`
3. `globalParams`
4. Paramètres issus de [itemsStateAndParams](#itemsStateAndParams) selon la file d'attente.

### itemsStateAndParams

Objet qui stocke les paramètres et les états des widgets ainsi qu'une file d'attente pour les changements de paramètres.
Il possède un champ `__meta__` pour stocker la file d'attente et les informations méta.

```ts
interface StateAndParamsMeta = {
    __meta__: {
        queue: {id: string}[]; // file d'attente
        version: number; // version actuelle de itemsStateAndParams
    };
}
```

Et aussi les états et paramètres des widgets :

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

### Menu

Vous pouvez spécifier un menu de superposition personnalisé pour les widgets DashKit en mode édition

```ts
type MenuItem = {
  id: string; // identifiant unique
  title?: string; // titre sous forme de chaîne
  icon?: ReactNode; // nœud d'icône
  iconSize?: number | string; // taille de l'icône en px sous forme de nombre ou de chaîne avec unités
  handler?: (item: ConfigItem) => void; // gestionnaire d'action personnalisé pour l'élément
  visible?: (item: ConfigItem) => boolean; // gestionnaire de visibilité optionnel pour filtrer les éléments du menu
  className?: string; // propriété de classe personnalisée
};

// utilisez un tableau d'éléments de menu dans les paramètres
<Dashkit overlayMenuItems={[] as Array<MenuItem> | null} />

[déprécié]
// La propriété overlayMenuItems a une priorité plus élevée que setSettings menu
DashKit.setSettings({menu: [] as Array<MenuItem>});
```

### Éléments traînables depuis l'ActionPanel

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

- **dragImageSrc** : Aperçu de l'image de glisser-déposer, par défaut une image PNG transparente de 1 px en base64
- **onDragStart** : Callback appelée lorsque l'élément est glissé depuis l'ActionPanel
- **onDragEnd** : Callback appelée lorsque l'élément est déposé ou que le glisser est annulé

```ts
type ItemDragProps = {
    type: string; // Type de plugin
    layout?: { // Optionnel. Taille de l'élément de mise en page pour l'aperçu et l'initialisation
        w?: number;
        h?: number;
    };
    extra?: any; // Contexte utilisateur personnalisé
};
```

```ts
type ItemDropProps = {
    commit: () => void; // Rappel à exécuter après toutes les opérations de configuration
    dragProps: ItemDragProps; // Propriétés de glisser de l'élément
    itemLayout: ConfigLayout; // Dimensions calculées de la mise en page de l'élément
    newLayout: ConfigLayout[]; // Nouvelle mise en page après dépôt de l'élément
};
```


#### Exemple :

```jsx
const overlayMenuItems = [
  {
    id: 'chart',
    icon: <Icon data={ChartColumn} />,
    title: 'Chart',
    qa: 'chart',
    dragProps: { // ItemDragProps
        type: 'custom', // Type de plugin enregistré
    },
  }
]

const onDrop = (dropProps: ItemDropProps) => {
  // ... ajouter l'élément à votre configuration
  dropProps.commit();
}

<DashKitDnDWrapper>
  <DashKit editMode={true} config={config} onChange={onChange} onDrop={onDrop} />
  <ActionPanel items={overlayMenuItems} />
</DashKitDnDWrapper>
```

### API CSS

| Nom                                            | Description          |
| :--------------------------------------------- | :------------------- |
| Variables du panneau d'action                  |                      |
| `--dashkit-action-panel-color`                 | Couleur de fond      |
| `--dashkit-action-panel-border-color`          | Couleur de bordure   |
| `--dashkit-action-panel-border-radius`         | Rayon de bordure     |
| Variables des éléments du panneau d'action     |                      |
| `--dashkit-action-panel-item-color`            | Couleur de fond      |
| `--dashkit-action-panel-item-text-color`       | Couleur de texte     |
| `--dashkit-action-panel-item-color-hover`      | Couleur de fond au survol |
| `--dashkit-action-panel-item-text-color-hover` | Couleur de texte au survol |
| Variables de superposition                      |                      |
| `--dashkit-overlay-border-color`               | Couleur de bordure   |
| `--dashkit-overlay-color`                      | Couleur de fond      |
| `--dashkit-overlay-opacity`                    | Opacité              |
| Variables des éléments de grille               |                      |
| `--dashkit-grid-item-edit-opacity`             | Opacité              |
| `--dashkit-grid-item-border-radius`            | Rayon de bordure     |
| Variables de l'espace réservé                   |                      |
| `--dashkit-placeholder-color`                  | Couleur de fond      |
| `--dashkit-placeholder-opacity`                | Opacité              |

#### Exemple d'utilisation

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
      <DashKit {...props.dashkitProps} />
      <ActionPanel {...props.actionPanelProps} />
    </div>
  );
};
```

## Développement

### Build et watch

- Compiler les dépendances `npm ci`
- Compiler le projet `npm run build`
- Compiler Storybook `npm run start`

Par défaut, Storybook s'exécute sur `http://localhost:7120/`.
Les nouveaux changements du projet ne sont pas toujours détectés lorsque Storybook est en cours d'exécution, il est donc préférable de recompiler manuellement le projet et de redémarrer Storybook.


### Exemple de configuration nginx pour le développement sur une machine de dev

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