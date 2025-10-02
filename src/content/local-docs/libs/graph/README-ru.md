# @gravity-ui/graph · [![npm package](https://img.shields.io/npm/v/@gravity-ui/graph)](https://www.npmjs.com/package/@gravity-ui/graph) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/graph/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/graph/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/graph/)

> [Руководство по миграции с 0.x на 1.x →](docs/migration-guides/v0-to-v1.md)

Библиотека для визуализации графов, которая сочетает в себе лучшее из двух миров:
- Canvas для высокой производительности при просмотре всего графа
- HTML/React для богатого взаимодействия при приближении

Больше не нужно выбирать между производительностью и интерактивностью. Идеально подходит для больших диаграмм, блок-схем и редакторов на основе узлов.

## Мотивация

Современные веб-приложения часто требуют сложной визуализации и интерактивности, но существующие решения обычно ориентированы на одну технологию рендеринга:

- **Canvas** обеспечивает высокую производительность для сложной графики, но ограничен в обработке текста и интерактивности.
- **HTML DOM** удобен для интерфейсов, но менее эффективен для сложной графики или большого количества элементов.

@gravity-ui/graph решает эту проблему, автоматически переключаясь между Canvas и HTML в зависимости от уровня масштаба:
- **При отдалении**: использует Canvas для эффективного рендеринга всего графа
- **Средний масштаб**: отображает схематический вид с базовой интерактивностью
- **При приближении**: переключается на компоненты HTML/React для богатого взаимодействия

## Как это работает

Библиотека использует умную систему рендеринга, которая автоматически управляет переходом между Canvas и компонентами React:

1. На низких уровнях масштаба всё рендерится на Canvas для обеспечения производительности
2. При приближении к детализированному виду компонент `GraphCanvas`:
   - Отслеживает изменения области просмотра камеры и масштаба
   - Вычисляет, какие блоки видны в текущей области просмотра (с отступами для плавной прокрутки)
   - Рендерит компоненты React только для видимых блоков
   - Автоматически обновляет список при прокрутке или масштабировании
   - Удаляет компоненты React при отдалении

```typescript
// Пример рендеринга компонентов React
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

## Установка

```bash
npm install @gravity-ui/graph
```

## Примеры

### Пример на React

[Подробная документация по компонентам React](docs/react/usage.md)

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

### Пример на чистом JavaScript

```javascript
import { Graph } from "@gravity-ui/graph";

// Создание контейнера
const container = document.createElement('div');
container.style.width = '100vw';
container.style.height = '100vh';
container.style.overflow = 'hidden';
document.body.appendChild(container);

// Инициализация графа с конфигурацией
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

## Живые примеры

- [Базовый пример](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--hundred-blocks)
- [Пример большого масштаба](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--five-thousands-blocks)
- [Пользовательский вид блоков](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--custom-schematic-block)
- [Соединение Безье](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--one-bezier-connection)
- [Настройка соединений](https://preview.gravity-ui.com/graph/?path=/story/api-updateconnection--default)

## Документация

### Оглавление

1. Система
   - [Жизненный цикл компонента](docs/system/component-lifecycle.md)
   - [События](docs/system/events.md)
   - [Настройки графа](docs/system/graph-settings.md)
   - [Публичный API](docs/system/public_api.md)
   - [Система планировщика](docs/system/scheduler-system.md)

2. Компоненты
   - [Компонент Canvas Graph](docs/components/canvas-graph-component.md)
   - [Компонент блока](docs/components/block-component.md)
   - [Якоря](docs/components/anchors.md)

3. Рендеринг
   - [Механизм рендеринга](docs/rendering/rendering-mechanism.md)
   - [Слои](docs/rendering/layers.md)

4. Блоки и соединения
   - [Группы блоков](docs/blocks/groups.md)
   - [Система соединений Canvas](docs/connections/canvas-connection-system.md)