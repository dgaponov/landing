# @gravity-ui/graph &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/graph)](https://www.npmjs.com/package/@gravity-ui/graph) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/graph/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/graph/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/graph/)

> [Guide de migration de 0.x vers 1.x →](docs/migration-guides/v0-to-v1.md)

Une bibliothèque de visualisation de graphes qui allie le meilleur des deux mondes :
- Canvas pour des performances élevées lors de l'affichage du graphe complet
- HTML/React pour des interactions riches une fois zoomé

Fini le choix entre performance et interactivité. Idéale pour les diagrammes volumineux, les organigrammes et les éditeurs basés sur des nœuds.

## Motivation

Les applications web modernes nécessitent souvent une visualisation et une interactivité complexes, mais les solutions existantes se concentrent généralement sur une seule technologie de rendu :

- **Canvas** offre des performances élevées pour les graphiques complexes, mais est limité pour la gestion du texte et l'interactivité.
- **HTML DOM** est pratique pour les interfaces, mais moins efficace pour les graphiques complexes ou un grand nombre d'éléments.

@gravity-ui/graph résout cela en basculant automatiquement entre Canvas et HTML en fonction du niveau de zoom :
- **Zoom arrière** : Utilise Canvas pour un rendu efficace du graphe complet
- **Zoom moyen** : Affiche une vue schématique avec une interactivité de base
- **Zoom avant** : Passe à des composants HTML/React pour des interactions riches

## Fonctionnement

La bibliothèque utilise un système de rendu intelligent qui gère automatiquement la transition entre Canvas et les composants React :

1. À des niveaux de zoom faibles, tout est rendu sur Canvas pour des performances optimales
2. Lors du zoom vers une vue détaillée, le composant `GraphCanvas` :
   - Suit les changements de viewport et d'échelle de la caméra
   - Calcule quels blocs sont visibles dans le viewport actuel (avec un rembourrage pour un défilement fluide)
   - Rend uniquement les composants React pour les blocs visibles
   - Met à jour automatiquement la liste lors du défilement ou du zoom
   - Supprime les composants React lors du zoom arrière

```typescript
// Exemple de rendu de composants React
const MyGraph = () => {
  return (
    <GraphCanvas
      graph={graph}
      renderBlock={(graph, block) => (
        <MyCustomBlockComponent 
          graph={graph} 
          block={block}
        />
      )}
    />
  );
};
```

[Storybook](https://preview.gravity-ui.com/graph/)

## Installation

```bash
npm install @gravity-ui/graph
```

## Exemples

### Exemple React

[Documentation détaillée sur les composants React](docs/react/usage.md)

```typescript
import { EAnchorType, Graph, GraphState } from "@gravity-ui/graph";
import { GraphCanvas, GraphBlock, useGraph } from "@gravity-ui/graph/react";
import React from "react";

const config = {};

export function GraphEditor() {
  const { graph, setEntities, start } = useGraph(config);

  useEffect(() => {
    setEntities({
      blocks: [
        {
          is: "block-action",
          id: "action_1",
          x: -100,
          y: -450,
          width: 126,
          height: 126,
          selected: true,
          name: "Block #1",
          anchors: [
            {
              id: "out1",
              blockId: "action_1",
              type: EAnchorType.OUT,
              index: 0
            }
          ],
        },
        {
          id: "action_2",
          is: "block-action",
          x: 253,
          y: 176,
          width: 126,
          height: 126,
          selected: false,
          name: "Block #2",
          anchors: [
            {
              id: "in1",
              blockId: "action_2",
              type: EAnchorType.IN,
              index: 0
            }
          ],
        }
      ],
      connections: [
        {
          sourceBlockId: "action_1",
          sourceAnchorId: "out1",
          targetBlockId: "action_2",
          targetAnchorId: "in1",
        }
      ]
    });
  }, [setEntities]);

  const renderBlockFn = (graph, block) => {
    return <GraphBlock graph={graph} block={block}>{block.id}</GraphBlock>;
  };

  return (
    <GraphCanvas
      graph={graph}
      renderBlock={renderBlockFn}
      onStateChanged={({ state }) => {
        if (state === GraphState.ATTACHED) {
          start();
          graph.zoomTo("center", { padding: 300 });
        }
      }}
    />
  );
}
```

### Exemple JavaScript vanilla

```javascript
import { Graph } from "@gravity-ui/graph";

// Créer un élément conteneur
const container = document.createElement('div');
container.style.width = '100vw';
container.style.height = '100vh';
container.style.overflow = 'hidden';
document.body.appendChild(container);

// Initialiser le graphe avec la configuration
const graph = new Graph({
    configurationName: "example",
    blocks: [],
    connections: [],
    settings: {
        canDragCamera: true,
        canZoomCamera: true,
        useBezierConnections: true,
        showConnectionArrows: true
    }
}, container);

```

```javascript
// Ajouter des blocs et des connexions
graph.setEntities({
    blocks: [
        {
            is: "block-action",
            id: "block1",
            x: 100,
            y: 100,
            width: 120,
            height: 120,
            name: "Bloc n°1",
            anchors: [
                {
                    id: "out1",
                    blockId: "block1",
                    type: EAnchorType.OUT,
                    index: 0
                }
            ]
        },
        {
            is: "block-action",
            id: "block2",
            x: 300,
            y: 300,
            width: 120,
            height: 120,
            name: "Bloc n°2",
            anchors: [
                {
                    id: "in1",
                    blockId: "block2",
                    type: EAnchorType.IN,
                    index: 0
                }
            ]
        }
    ],
    connections: [
        {
            sourceBlockId: "block1",
            sourceAnchorId: "out1",
            targetBlockId: "block2",
            targetAnchorId: "in1"
        }
    ]
});

// Démarrer le rendu
graph.start();

// Centrer la vue
graph.zoomTo("center", { padding: 100 });
```

## Exemples en direct

- [Exemple basique](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--hundred-blocks)
- [Exemple à grande échelle](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--five-thousands-blocks)
- [Vue de blocs personnalisés](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--custom-schematic-block)
- [Connexion en Bézier](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--one-bezier-connection)
- [Personnalisation des connexions](https://preview.gravity-ui.com/graph/?path=/story/api-updateconnection--default)

## Documentation

### Table des matières

1. Système
   - [Cycle de vie des composants](docs/system/component-lifecycle.md)
   - [Événements](docs/system/events.md)
   - [Paramètres du graphe](docs/system/graph-settings.md)
   - [API publique](docs/system/public_api.md)
   - [Système de planification](docs/system/scheduler-system.md)

2. Composants
   - [Composant de graphe canvas](docs/components/canvas-graph-component.md)
   - [Composant de bloc](docs/components/block-component.md)
   - [Ancrages](docs/components/anchors.md)

3. Rendu
   - [Mécanisme de rendu](docs/rendering/rendering-mechanism.md)
   - [Couches](docs/rendering/layers.md)

4. Blocs et connexions
   - [Groupes de blocs](docs/blocks/groups.md)
   - [Système de connexions canvas](docs/connections/canvas-connection-system.md)