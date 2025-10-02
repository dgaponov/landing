# @gravity-ui/timeline [![npm package](https://img.shields.io/npm/v/@gravity-ui/timeline)](https://www.npmjs.com/package/@gravity-ui/timeline) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/timeline/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/timeline/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/timeline/)

캔버스 렌더링을 사용한 인터랙티브 타임라인 시각화를 구축하기 위한 React 기반 라이브러리입니다.

## 문서

자세한 내용은 [문서](./docs/docs.md)를 참조하세요.

## 주요 기능

- 높은 성능을 위한 캔버스 기반 렌더링
- 확대 및 이동 기능을 지원하는 인터랙티브 타임라인
- 이벤트, 마커, 축 및 그리드 지원
- 자동 그룹 확대를 통한 스마트 마커 그룹화 - 그룹화된 마커를 클릭하면 개별 구성 요소로 확대됩니다
- 대규모 데이터셋에 대한 성능 향상을 위한 가상화 렌더링 (타임라인 콘텐츠가 뷰포트를 초과할 때만 활성화됨)
- 사용자 지정 가능한 외관 및 동작
- 완전한 타입 정의를 포함한 TypeScript 지원
- 커스텀 훅을 통한 React 통합

## 설치

```bash
npm install @gravity-ui/timeline
```

## 사용법

타임라인 컴포넌트는 다음 기본 설정으로 React 애플리케이션에서 사용할 수 있습니다:

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

### 마커 구조

각 마커는 다음 구조를 필요로 합니다:

```typescript
type TimelineMarker = {
  time: number;           // Timestamp for the marker position
  color: string;          // Color of the marker line
  activeColor: string;    // Color when marker is selected (required)
  hoverColor: string;     // Color when marker is hovered (required)
  lineWidth?: number;     // Optional width of the marker line
  label?: string;         // Optional label text
  labelColor?: string;    // Optional label color
  renderer?: AbstractMarkerRenderer; // Optional custom renderer
  nonSelectable?: boolean;// Whether marker can be selected
  group?: boolean;        // Whether marker represents a group
};
```

### 마커 그룹화 및 확대

타임라인은 가까운 마커를 자동으로 그룹화하고 확대 기능을 제공합니다:

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

## 작동 원리

타임라인 컴포넌트는 React를 사용해 구축되었으며, 인터랙티브 타임라인 시각화를 유연하게 생성할 수 있습니다. 작동 방식은 다음과 같습니다:

### 컴포넌트 아키텍처

타임라인은 두 개의 주요 객체를 통해 구성할 수 있는 React 컴포넌트로 구현되었습니다:

1. **TimelineSettings**: 타임라인의 핵심 동작과 외관을 제어합니다
   - `start`: 타임라인의 시작 시간
   - `end`: 타임라인의 종료 시간
   - `axes`: 축 구성 배열
   - `events`: 이벤트 구성 배열
   - `markers`: 마커 구성 배열

2. **ViewConfiguration**: 시각적 표현 및 상호작용 설정을 관리합니다
   - 외관, 확대 수준 및 상호작용 동작을 제어합니다
   - 사용자 지정하거나 기본값을 사용할 수 있습니다

### 이벤트 처리

타임라인 컴포넌트는 여러 인터랙티브 이벤트를 지원합니다:

- `on-click`: 타임라인을 클릭할 때 발생
- `on-context-click`: 오른쪽 클릭/컨텍스트 메뉴 시 발생
- `on-select-change`: 선택이 변경될 때 발생
- `on-hover`: 타임라인 요소 위에 마우스를 올릴 때 발생
- `on-leave`: 마우스가 타임라인 요소를 벗어날 때 발생

이벤트 처리 예시:

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

### React 통합

컴포넌트는 타임라인 관리를 위한 커스텀 훅을 사용합니다:

- `useTimeline`: 타임라인 인스턴스와 생명 주기를 관리합니다
  - 타임라인을 생성하고 초기화합니다
  - 컴포넌트 언마운트 시 정리 작업을 처리합니다
  - 타임라인 인스턴스에 접근할 수 있게 합니다

- `useTimelineEvent`: 이벤트 구독 및 정리 작업을 처리합니다
  - 이벤트 리스너 생명 주기를 관리합니다
  - 언마운트 시 리스너를 자동으로 정리합니다

컴포넌트는 언마운트 시 타임라인 인스턴스의 정리 및 소멸을 자동으로 처리합니다.

### 이벤트 구조

타임라인의 이벤트는 다음 구조를 따릅니다:

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

### 직접 TypeScript 사용

Timeline 클래스는 React 없이 TypeScript에서 직접 사용할 수 있습니다. 이는 다른 프레임워크나 순수 JavaScript 애플리케이션과 통합할 때 유용합니다:

```typescript
import { Timeline } from '@gravity-ui/timeline';

const timestamp = Date.now();

// Create a timeline instance
const timeline = new Timeline({
  settings: {
    start: timestamp,
    end: timestamp + 3600000, // 1 hour from now
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
        from: timestamp + 1800000, // 30 minutes from now
        to: timestamp + 2400000,   // 40 minutes from now
        label: 'Sample Event',
        axisId: 'main'
      }
    ],
    markers: [
      {
        id: 'marker1',
        time: timestamp + 1200000, // 20 minutes from now
        label: 'Important Point',
        color: '#ff0000',
        activeColor: '#ff5252',
        hoverColor: '#ff1744'
      }
    ]
  },
  viewConfiguration: {
    // Optional: customize view settings
    zoomLevels: [1, 2, 4, 8, 16],
    hideRuler: false,
    showGrid: true
  }
});

// Initialize with a canvas element
const canvas = document.querySelector('canvas');
if (canvas instanceof HTMLCanvasElement) {
  timeline.init(canvas);
}

// Add event listeners
timeline.on('on-click', (detail) => {
  console.log('Timeline clicked:', detail);
});

timeline.on('on-select-change', (detail) => {
  console.log('Selection changed:', detail);
});

// Clean up when done
timeline.destroy();
```

Timeline 클래스는 타임라인을 관리하기 위한 풍부한 API를 제공합니다:

- **이벤트 관리**:
  ```typescript
  // Add event listener
  timeline.on('eventClick', (detail) => {
    console.log('Event clicked:', detail);
  });

  // Remove event listener
  const handler = (detail) => console.log(detail);
  timeline.on('eventClick', handler);
  timeline.off('eventClick', handler);

  // Emit custom events
  timeline.emit('customEvent', { data: 'custom data' });
  ```

- **타임라인 제어**:
  ```typescript
  // Update timeline data
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

  // Update axes
  timeline.api.setAxes([
    {
      id: 'newAxis',
      label: 'New Axis',
      color: '#0000ff'
    }
  ]);

  // Update markers
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

## 실시간 예제

[Storybook](https://preview.gravity-ui.com/timeline/)에서 인터랙티브 예제를 확인해 보세요:

- [기본 타임라인](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--basic) - 이벤트와 축이 포함된 간단한 타임라인
- [무한 타임라인](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--endless-timelines) - 이벤트와 축이 포함된 무한 타임라인
- [마커](https://preview.gravity-ui.com/timeline/?path=/story/timeline-markers--basic) - 수직 마커와 레이블이 포함된 타임라인
- [커스텀 이벤트](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--custom-renderer) - 커스텀 이벤트 렌더링이 포함된 타임라인


## 개발

### Storybook

이 프로젝트는 컴포넌트 개발과 문서화를 위한 Storybook을 포함합니다.

Storybook을 실행하려면:

```bash
npm run storybook
```

이 명령은 6006 포트에서 Storybook 개발 서버를 시작합니다. http://localhost:6006에서 접근할 수 있습니다.

배포를 위한 Storybook의 정적 버전을 빌드하려면:

```bash
npm run build-storybook
```

## 라이선스

MIT