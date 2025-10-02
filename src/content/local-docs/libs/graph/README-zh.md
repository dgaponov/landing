# @gravity-ui/graph &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/graph)](https://www.npmjs.com/package/@gravity-ui/graph) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/graph/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/graph/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/graph/)

> [从 0.x 迁移到 1.x 的指南 →](docs/migration-guides/v0-to-v1.md)

这是一个图可视化库，结合了两者的优点：
- Canvas 用于查看完整图时的高性能渲染
- HTML/React 用于放大时丰富的交互体验

无需在性能和交互性之间做出选择。非常适合大型图表、流程图和基于节点的编辑器。

## 动机

现代 Web 应用通常需要复杂的可视化和交互性，但现有的解决方案通常只关注单一的渲染技术：

- **Canvas** 在复杂图形方面提供高性能，但文本处理和交互性有限。
- **HTML DOM** 便于构建界面，但对于复杂图形或大量元素来说效率较低。

@gravity-ui/graph 通过根据缩放级别自动在 Canvas 和 HTML 之间切换来解决这个问题：
- **缩小视图**：使用 Canvas 高效渲染整个图
- **中等缩放**：显示示意图视图，提供基本交互
- **放大视图**：切换到 HTML/React 组件，实现丰富的交互

## 工作原理

该库使用智能渲染系统，自动管理 Canvas 和 React 组件之间的切换：

1. 在低缩放级别时，一切都在 Canvas 上渲染以确保性能
2. 当放大到详细视图时，`GraphCanvas` 组件：
   - 跟踪相机视口和缩放变化
   - 计算当前视口中可见的块（并添加填充以实现平滑滚动）
   - 仅为可见块渲染 React 组件
   - 在滚动或缩放时自动更新列表
   - 在缩小时移除 React 组件

```typescript
// React 组件渲染示例
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

## 安装

```bash
npm install @gravity-ui/graph
```

## 示例

### React 示例

[详细的 React 组件文档](docs/react/usage.md)

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

### Vanilla JavaScript 示例

```javascript
import { Graph } from "@gravity-ui/graph";

// 创建容器元素
const container = document.createElement('div');
container.style.width = '100vw';
container.style.height = '100vh';
container.style.overflow = 'hidden';
document.body.appendChild(container);

// 使用配置初始化图
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

## 实时示例

- [基础示例](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--hundred-blocks)
- [大规模示例](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--five-thousands-blocks)
- [自定义块视图](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--custom-schematic-block)
- [贝塞尔曲线连接](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--one-bezier-connection)
- [连接自定义](https://preview.gravity-ui.com/graph/?path=/story/api-updateconnection--default)

## 文档

### 目录

1. 系统
   - [组件生命周期](docs/system/component-lifecycle.md)
   - [事件](docs/system/events.md)
   - [图形设置](docs/system/graph-settings.md)
   - [公共 API](docs/system/public_api.md)
   - [调度器系统](docs/system/scheduler-system.md)

2. 组件
   - [画布图形组件](docs/components/canvas-graph-component.md)
   - [块组件](docs/components/block-component.md)
   - [锚点](docs/components/anchors.md)

3. 渲染
   - [渲染机制](docs/rendering/rendering-mechanism.md)
   - [图层](docs/rendering/layers.md)

4. 块和连接
   - [块组](docs/blocks/groups.md)
   - [画布连接系统](docs/connections/canvas-connection-system.md)