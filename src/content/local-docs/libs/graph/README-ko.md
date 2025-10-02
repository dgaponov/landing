# @gravity-ui/graph &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/graph)](https://www.npmjs.com/package/@gravity-ui/graph) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/graph/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/graph/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/graph/)

> [0.x에서 1.x로의 마이그레이션 가이드 →](docs/migration-guides/v0-to-v1.md)

두 세계의 장점을 결합한 그래프 시각화 라이브러리입니다:
- 전체 그래프를 볼 때 높은 성능을 위한 Canvas
- 확대 시 풍부한 상호작용을 위한 HTML/React

더 이상 성능과 상호작용성 사이에서 선택할 필요가 없습니다. 대형 다이어그램, 플로우차트, 노드 기반 에디터에 이상적입니다.

## 동기

현대 웹 애플리케이션은 종종 복잡한 시각화와 상호작용성을 요구하지만, 기존 솔루션은 보통 단일 렌더링 기술에 초점을 맞춥니다:

- **Canvas**는 복잡한 그래픽에 높은 성능을 제공하지만, 텍스트 처리와 상호작용성에서 제한적입니다.
- **HTML DOM**은 인터페이스 구현에 편리하지만, 복잡한 그래픽이나 대량의 요소를 다룰 때는 효율이 떨어집니다.

@gravity-ui/graph는 줌 수준에 따라 Canvas와 HTML을 자동으로 전환하여 이 문제를 해결합니다:
- **축소된 보기**: 전체 그래프의 효율적인 렌더링을 위해 Canvas 사용
- **중간 줌**: 기본 상호작용이 포함된 개략도 보기 표시
- **확대된 보기**: 풍부한 상호작용을 위한 HTML/React 컴포넌트로 전환

## 작동 방식

이 라이브러리는 Canvas와 React 컴포넌트 간의 전환을 자동으로 관리하는 스마트 렌더링 시스템을 사용합니다:

1. 낮은 줌 수준에서는 모든 요소가 성능을 위해 Canvas에 렌더링됩니다.
2. 상세 보기에서 확대할 때 `GraphCanvas` 컴포넌트는:
   - 카메라 뷰포트와 스케일 변경을 추적합니다.
   - 부드러운 스크롤을 위한 패딩을 포함해 현재 뷰포트에서 보이는 블록을 계산합니다.
   - 보이는 블록에 대해서만 React 컴포넌트를 렌더링합니다.
   - 스크롤 또는 줌 시 목록을 자동으로 업데이트합니다.
   - 확대 축소 시 React 컴포넌트를 제거합니다.

```typescript
// React 컴포넌트 렌더링 예시
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

## 설치

```bash
npm install @gravity-ui/graph
```

## 예제

### React 예제

[상세 React 컴포넌트 문서](docs/react/usage.md)

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

### Vanilla JavaScript 예제

```javascript
import { Graph } from "@gravity-ui/graph";

// 컨테이너 요소 생성
const container = document.createElement('div');
container.style.width = '100vw';
container.style.height = '100vh';
container.style.overflow = 'hidden';
document.body.appendChild(container);

// 구성으로 그래프 초기화
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

## 실행 예시

- [기본 예시](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--hundred-blocks)
- [대규모 예시](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--five-thousands-blocks)
- [커스텀 블록 뷰](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--custom-schematic-block)
- [베지어 연결](https://preview.gravity-ui.com/graph/?path=/story/stories-main-grapheditor--one-bezier-connection)
- [연결 커스터마이징](https://preview.gravity-ui.com/graph/?path=/story/api-updateconnection--default)

## 문서

### 목차

1. 시스템
   - [컴포넌트 수명 주기](docs/system/component-lifecycle.md)
   - [이벤트](docs/system/events.md)
   - [그래프 설정](docs/system/graph-settings.md)
   - [공개 API](docs/system/public_api.md)
   - [스케줄러 시스템](docs/system/scheduler-system.md)

2. 컴포넌트
   - [캔버스 그래프 컴포넌트](docs/components/canvas-graph-component.md)
   - [블록 컴포넌트](docs/components/block-component.md)
   - [앵커](docs/components/anchors.md)

3. 렌더링
   - [렌더링 메커니즘](docs/rendering/rendering-mechanism.md)
   - [레이어](docs/rendering/layers.md)

4. 블록 및 연결
   - [블록 그룹](docs/blocks/groups.md)
   - [캔버스 연결 시스템](docs/connections/canvas-connection-system.md)