# @gravity-ui/timeline [![npm package](https://img.shields.io/npm/v/@gravity-ui/timeline)](https://www.npmjs.com/package/@gravity-ui/timeline) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/timeline/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/timeline/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/timeline/)

一个基于 React 的库，用于构建使用 Canvas 渲染的交互式时间线可视化。

## 文档

详情请参阅 [文档](./docs/docs.md)。

## 功能特性

- 基于 Canvas 的渲染，提供高性能
- 交互式时间线，支持缩放和平移功能
- 支持事件、标记、轴和网格
- 智能标记分组，并自动缩放到组 - 点击分组标记以缩放到其单个组件
- 虚拟化渲染，提升大数据集的性能（仅在时间线内容超出视口时激活）
- 可自定义外观和行为
- 支持 TypeScript，并提供完整的类型定义
- 与 React 集成，使用自定义钩子

## 安装

```bash
npm install @gravity-ui/timeline
```

## 使用方法

时间线组件可以在 React 应用中使用，以下是基本设置：

```tsx
import { TimelineCanvas, useTimeline } from '@gravity-ui/timeline/react';

const MyTimelineComponent = () => {
  const { timeline, api, start, stop } = useTimeline({
    settings: {
      start: Date.now(),
      end: Date.now() + 3600000, // 1 hour from now
      axes: [],
      events: [],
      markers: []
    },
    viewConfiguration: {
      // Optional view configuration
    }
  });

  // timeline - Timeline instance
  // api - CanvasApi instance (same as timeline.api)
  // start - function to initialize timeline with canvas
  // stop - function to destroy timeline

  return (
    <div style={{ width: '100%', height: '100%' }}>
      <TimelineCanvas timeline={timeline} />
    </div>
  );
};
```

### 标记结构

每个标记都需要以下结构：

```typescript
type TimelineMarker = {
  time: number;           // 标记位置的时间戳
  color: string;          // 标记线的颜色
  activeColor: string;    // 标记被选中时的颜色（必需）
  hoverColor: string;     // 标记悬停时的颜色（必需）
  lineWidth?: number;     // 标记线的可选宽度
  label?: string;         // 可选的标签文本
  labelColor?: string;    // 可选的标签颜色
  renderer?: AbstractMarkerRenderer; // 可选的自定义渲染器
  nonSelectable?: boolean;// 标记是否可被选中
  group?: boolean;        // 标记是否表示一个组
};
```

### 标记分组和缩放

时间线会自动对相近的标记进行分组，并提供缩放功能：

```tsx
const MyTimelineComponent = () => {
  const { timeline } = useTimeline({
    settings: {
      start: Date.now(),
      end: Date.now() + 3600000,
      axes: [],
      events: [],
      markers: [
        // These markers will be grouped together
        { time: Date.now(), color: '#ff0000', activeColor: '#ff5252', hoverColor: '#ff1744', label: 'Event 1' },
        { time: Date.now() + 1000, color: '#ff0000', activeColor: '#ff5252', hoverColor: '#ff1744', label: 'Event 2' },
        { time: Date.now() + 2000, color: '#ff0000', activeColor: '#ff5252', hoverColor: '#ff1744', label: 'Event 3' },
      ]
    },
    viewConfiguration: {
      markers: {
        collapseMinDistance: 8,        // Group markers within 8 pixels
        groupZoomEnabled: true,        // Enable zoom on group click
        groupZoomPadding: 0.3,        // 30% padding around group
        groupZoomMaxFactor: 0.3,      // Max zoom factor
      }
    }
  });

  // Listen for group zoom events
  useTimelineEvent(timeline, 'on-group-marker-click', (data) => {
    console.log('Group zoomed:', data);
  });

  return <TimelineCanvas timeline={timeline} />;
};
```

## 工作原理

时间线组件基于 React 构建，提供了一种灵活的方式来创建交互式时间线可视化。下面是其工作方式：

### 组件架构

时间线实现为一个 React 组件，可以通过两个主要对象进行配置：

1. **TimelineSettings**：控制时间线核心行为和外观
   - `start`：时间线的开始时间
   - `end`：时间线的结束时间
   - `axes`：轴配置数组
   - `events`：事件配置数组
   - `markers`：标记配置数组

2. **ViewConfiguration**：管理视觉表示和交互设置
   - 控制外观、缩放级别和交互行为
   - 可以自定义或使用默认值

### 事件处理

时间线组件支持多种交互事件：

- `on-click`：点击时间线时触发
- `on-context-click`：右键点击/上下文菜单时触发
- `on-select-change`：选择变化时触发
- `on-hover`：悬停在时间线元素上时触发
- `on-leave`：鼠标离开时间线元素时触发

事件处理示例：

```tsx
import { useTimelineEvent } from '@gravity-ui/timeline/react';

const MyTimelineComponent = () => {
  const { timeline } = useTimeline({ /* ... */ });

  useTimelineEvent(timeline, 'on-click', (data) => {
    console.log('Timeline clicked:', data);
  });

  useTimelineEvent(timeline, 'on-select-change', (data) => {
    console.log('Selection changed:', data);
  });

  return <TimelineCanvas timeline={timeline} />;
};
```

### React 集成

组件使用自定义钩子来管理时间线：

- `useTimeline`：管理时间线实例及其生命周期
  - 创建并初始化时间线
  - 在组件卸载时处理清理
  - 提供对时间线实例的访问

- `useTimelineEvent`：处理事件订阅和清理
  - 管理事件监听器的生命周期
  - 在卸载时自动清理监听器

组件会自动处理时间线实例的清理和销毁，当组件卸载时。

### 事件结构

时间线中的事件遵循以下结构：

```typescript
type TimelineEvent = {
  id: string;             // Unique identifier
  from: number;           // Start timestamp
  to?: number;            // End timestamp (optional for point events)
  axisId: string;         // ID of the axis this event belongs to
  trackIndex: number;     // Index in the axis track
  renderer?: AbstractEventRenderer; // Optional custom renderer
  color?: string;         // Optional event color
  selectedColor?: string; // Optional selected state color
};
```

### 直接使用 TypeScript

Timeline 类可以在 TypeScript 中直接使用，而无需 React。这对于与其他框架或纯 JavaScript 应用集成非常有用：

```typescript
import { Timeline } from '@gravity-ui/timeline';

const timestamp = Date.now();

// 创建 Timeline 实例
const timeline = new Timeline({
  settings: {
    start: timestamp,
    end: timestamp + 3600000, // 现在起 1 小时后
    axes: [
      {
        id: 'main',
        label: 'Main Axis',
        color: '#000000'
      }
    ],
    events: [
      {
        id: 'event1',
        from: timestamp + 1800000, // 现在起 30 分钟后
        to: timestamp + 2400000,   // 现在起 40 分钟后
        label: 'Sample Event',
        axisId: 'main'
      }
    ],
    markers: [
      {
        id: 'marker1',
        time: timestamp + 1200000, // 现在起 20 分钟后
        label: 'Important Point',
        color: '#ff0000',
        activeColor: '#ff5252',
        hoverColor: '#ff1744'
      }
    ]
  },
  viewConfiguration: {
    // 可选：自定义视图设置
    zoomLevels: [1, 2, 4, 8, 16],
    hideRuler: false,
    showGrid: true
  }
});

// 使用 canvas 元素初始化
const canvas = document.querySelector('canvas');
if (canvas instanceof HTMLCanvasElement) {
  timeline.init(canvas);
}

// 添加事件监听器
timeline.on('on-click', (detail) => {
  console.log('Timeline clicked:', detail);
});

timeline.on('on-select-change', (detail) => {
  console.log('Selection changed:', detail);
});

// 完成后清理
timeline.destroy();
```

Timeline 类提供了丰富的 API 来管理时间线：

- **事件管理**：
  ```typescript
  // 添加事件监听器
  timeline.on('eventClick', (detail) => {
    console.log('Event clicked:', detail);
  });

  // 移除事件监听器
  const handler = (detail) => console.log(detail);
  timeline.on('eventClick', handler);
  timeline.off('eventClick', handler);

  // 发出自定义事件
  timeline.emit('customEvent', { data: 'custom data' });
  ```

- **时间线控制**：
  ```typescript
  // 更新时间线数据
  timeline.api.setEvents([
    {
      id: 'newEvent',
      from: Date.now(),
      to: Date.now() + 3600000,
      label: 'New Event',
      axisId: 'main',
      trackIndex: 0
    }
  ]);

  // 更新轴
  timeline.api.setAxes([
    {
      id: 'newAxis',
      label: 'New Axis',
      color: '#0000ff'
    }
  ]);

  // 更新标记
  timeline.api.setMarkers([
    {
      id: 'newMarker',
      time: Date.now(),
      label: 'New Marker',
      color: '#00ff00',
      activeColor: '#4caf50',
      hoverColor: '#2e7d32'
    }
  ]);
  ```

## 实时示例

在我们的 [Storybook](https://preview.gravity-ui.com/timeline/) 中探索交互式示例：

- [Basic Timeline](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--basic) - 带有事件和轴的简单时间线
- [Endless Timeline](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--endless-timelines) - 带有事件和轴的无尽时间线
- [Markers](https://preview.gravity-ui.com/timeline/?path=/story/timeline-markers--basic) - 带有垂直标记和标签的时间线
- [Custom Events](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--custom-renderer) - 带有自定义事件渲染的时间线


## 开发

### Storybook

此项目包含 Storybook，用于组件开发和文档。

运行 Storybook：

```bash
npm run storybook
```

这将启动 Storybook 开发服务器，端口为 6006。您可以通过 http://localhost:6006 访问它。

构建 Storybook 的静态版本以供部署：

```bash
npm run build-storybook
```

## 许可证

MIT