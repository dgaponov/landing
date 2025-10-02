# @gravity-ui/graph · [![npm package](https://img.shields.io/npm/v/@gravity-ui/graph)](https://www.npmjs.com/package/@gravity-ui/graph) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/graph/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/graph/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/graph/)

> [Migrationshandbuch von 0.x zu 1.x →](docs/migration-guides/v0-to-v1.md)

Eine Graphik-Visualisierungs-Bibliothek, die das Beste aus beiden Welten vereint:
- Canvas für hohe Leistung beim Anzeigen des gesamten Graphen
- HTML/React für reiche Interaktionen beim Hineinzoomen

Kein Kompromiss mehr zwischen Leistung und Interaktivität. Ideal für große Diagramme, Flussdiagramme und knotenbasierte Editoren.

## Motivation

Moderne Web-Anwendungen erfordern oft komplexe Visualisierungen und Interaktivität, doch bestehende Lösungen konzentrieren sich in der Regel auf eine einzige Rendering-Technologie:

- **Canvas** bietet hohe Leistung für komplexe Grafiken, ist aber bei der Textbehandlung und Interaktivität eingeschränkt.
- **HTML DOM** eignet sich hervorragend für Benutzeroberflächen, ist jedoch weniger effizient bei komplexen Grafiken oder großen Mengen an Elementen.

@gravity-ui/graph löst dieses Problem, indem es automatisch zwischen Canvas und HTML je nach Zoom-Level wechselt:
- **Ausgezoomt**: Nutzt Canvas für ein effizientes Rendering des gesamten Graphen
- **Mittlerer Zoom**: Zeigt eine schematische Ansicht mit grundlegender Interaktivität
- **Hineingezoomt**: Wechselt zu HTML/React-Komponenten für umfassende Interaktionen

## Funktionsweise

Die Bibliothek verwendet ein intelligentes Rendering-System, das den Übergang zwischen Canvas und React-Komponenten automatisch steuert:

1. Bei niedrigen Zoom-Leveln wird alles auf Canvas gerendert, um die Leistung zu optimieren
2. Beim Hineinzoomen in die Detailansicht übernimmt die `GraphCanvas`-Komponente:
   - Die Verfolgung von Änderungen am Kamera-Viewport und der Skalierung
   - Die Berechnung, welche Blöcke im aktuellen Viewport sichtbar sind (mit Puffer für flüssiges Scrollen)
   - Das Rendering von React-Komponenten nur für sichtbare Blöcke
   - Die automatische Aktualisierung der Liste beim Scrollen oder Zoomen
   - Das Entfernen von React-Komponenten beim Auszoomen

```typescript
// Beispiel für das Rendering von React-Komponenten
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

## Beispiele

### React-Beispiel

[Detaillierte Dokumentation zu React-Komponenten](docs/react/usage.md)

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

### Vanilla-JavaScript-Beispiel

```javascript
import { Graph } from "@gravity-ui/graph";

// Container-Element erstellen
const container = document.createElement('div');
container.style.width = '100vw';
container.style.height = '100vh';
container.style.overflow = 'hidden';
document.body.appendChild(container);

// Graph mit Konfiguration initialisieren
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
// Add blocks and connections
graph.setEntities({
    blocks: [
        {
            is: "block-action",
            id: "block1",
            x: 100,
            y: 100,
            width: 120,
            height: 120,
            name: "Block #1",
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
            name: "Block #2",
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

// Start rendering
graph.start();

// Center the view
graph.zoomTo("center", { padding: 100 });
```

## Live-Beispiele

- [Einfaches Beispiel](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--hundred-blocks)
- [Beispiel mit großem Umfang](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--five-thousands-blocks)
- [Ansicht mit benutzerdefinierten Blöcken](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--custom-schematic-block)
- [Bezier-Verbindung](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--one-bezier-connection)
- [Anpassung von Verbindungen](https://preview.gravity-ui.com/graph/?path=/story/api-updateconnection--default)

## Dokumentation

### Inhaltsverzeichnis

1. System
   - [Komponenten-Lebenszyklus](docs/system/component-lifecycle.md)
   - [Ereignisse](docs/system/events.md)
   - [Graph-Einstellungen](docs/system/graph-settings.md)
   - [Öffentliche API](docs/system/public_api.md)
   - [Planer-System](docs/system/scheduler-system.md)

2. Komponenten
   - [Canvas-Graph-Komponente](docs/components/canvas-graph-component.md)
   - [Block-Komponente](docs/components/block-component.md)
   - [Ankerpunkte](docs/components/anchors.md)

3. Rendering
   - [Rendering-Mechanismus](docs/rendering/rendering-mechanism.md)
   - [Ebenen](docs/rendering/layers.md)

4. Blöcke und Verbindungen
   - [Block-Gruppen](docs/blocks/groups.md)
   - [Canvas-Verbindungs-System](docs/connections/canvas-connection-system.md)