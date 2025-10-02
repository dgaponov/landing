# @gravity-ui/timeline [![npm package](https://img.shields.io/npm/v/@gravity-ui/timeline)](https://www.npmjs.com/package/@gravity-ui/timeline) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/timeline/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/timeline/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/timeline/)

캔버스 렌더링을 사용한 인터랙티브 타임라인 시각화를 구축하기 위한 React 기반 라이브러리입니다.

## 문서

자세한 내용은 [문서](./docs/docs.md)를 참조하세요.

## 기능

- 높은 성능을 위한 캔버스 기반 렌더링
- 확대 및 이동 기능을 지원하는 인터랙티브 타임라인
- 이벤트, 마커, 축 및 그리드 지원
- 자동 그룹 확대를 통한 스마트 마커 그룹화 - 그룹화된 마커를 클릭하면 개별 구성 요소로 확대됩니다
- 대규모 데이터셋에 대한 성능 향상을 위한 가상화 렌더링 (타임라인 콘텐츠가 뷰포트를 초과할 때만 활성화됨)
- 사용자 지정 가능한 외관 및 동작
- 전체 타입 정의를 포함한 TypeScript 지원
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
      end: Date.now() + 3600000, // 현재로부터 1시간 후
      axes: [],
      events: [],
      markers: []
    },
    viewConfiguration: {
      // 선택적 뷰 구성
    }
  });

  // timeline - Timeline 인스턴스
  // api - CanvasApi 인스턴스 (timeline.api와 동일)
  // start - 캔버스로 타임라인을 초기화하는 함수
  // stop - 타임라인을 종료하는 함수

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
  time: number;           // 마커 위치에 대한 타임스탬프
  color: string;          // 마커 선의 색상
  activeColor: string;    // 마커가 선택될 때의 색상 (필수)
  hoverColor: string;     // 마커가 호버될 때의 색상 (필수)
  lineWidth?: number;     // 마커 선의 선택적 너비
  label?: string;         // 선택적 레이블 텍스트
  labelColor?: string;    // 선택적 레이블 색상
  renderer?: AbstractMarkerRenderer; // 선택적 커스텀 렌더러
  nonSelectable?: boolean;// 마커가 선택될 수 있는지 여부
  group?: boolean;        // 마커가 그룹을 나타내는지 여부
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
        // 이 마커들은 함께 그룹화됩니다
        { time: Date.now(), color: '#ff0000', activeColor: '#ff5252', hoverColor: '#ff1744', label: 'Event 1' },
        { time: Date.now() + 1000, color: '#ff0000', activeColor: '#ff5252', hoverColor: '#ff1744', label: 'Event 2' },
        { time: Date.now() + 2000, color: '#ff0000', activeColor: '#ff5252', hoverColor: '#ff1744', label: 'Event 3' },
      ]
    },
    viewConfiguration: {
      markers: {
        collapseMinDistance: 8,        // 8픽셀 이내의 마커를 그룹화
        groupZoomEnabled: true,        // 그룹 클릭 시 확대 활성화
        groupZoomPadding: 0.3,        // 그룹 주변 30% 패딩
        groupZoomMaxFactor: 0.3,      // 최대 확대 비율
      }
    }
  });

  // 그룹 확대 이벤트 감지
  useTimelineEvent(timeline, 'on-group-marker-click', (data) => {
    console.log('Group zoomed:', data);
  });

  return <TimelineCanvas timeline={timeline} />;
};
```

## 작동 원리

타임라인 컴포넌트는 React를 사용해 구축되었으며, 인터랙티브 타임라인 시각화를 유연하게 생성할 수 있습니다. 작동 방식은 다음과 같습니다:

### 컴포넌트 아키텍처

타임라인은 두 개의 주요 객체를 통해 구성할 수 있는 React 컴포넌트로 구현됩니다:

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
- `on-hover`: 타임라인 요소 위에 호버할 때 발생
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

- `useTimeline`: 타임라인 인스턴스와 생명주기를 관리합니다
  - 타임라인을 생성하고 초기화합니다
  - 컴포넌트 언마운트 시 정리 작업을 처리합니다
  - 타임라인 인스턴스에 접근할 수 있게 합니다

- `useTimelineEvent`: 이벤트 구독 및 정리 작업을 처리합니다
  - 이벤트 리스너 생명주기를 관리합니다
  - 언마운트 시 리스너를 자동으로 정리합니다

컴포넌트는 언마운트 시 타임라인 인스턴스의 정리 및 종료를 자동으로 처리합니다.

### 이벤트 구조

타임라인의 이벤트는 다음 구조를 따릅니다:

```typescript
type TimelineEvent = {
  id: string;             // 고유 식별자
  from: number;           // 시작 타임스탬프
  to?: number;            // 종료 타임스탬프 (포인트 이벤트의 경우 선택적)
  axisId: string;         // 이 이벤트가 속한 축의 ID
  trackIndex: number;     // 축 트랙 내 인덱스
  renderer?: AbstractEventRenderer; // 선택적 사용자 지정 렌더러
  color?: string;         // 선택적 이벤트 색상
  selectedColor?: string; // 선택된 상태 색상 (선택적)
};
```

### 직접 TypeScript 사용

Timeline 클래스는 React 없이 TypeScript에서 직접 사용할 수 있습니다. 이는 다른 프레임워크나 순수 JavaScript 애플리케이션과 통합할 때 유용합니다:

```typescript
import { Timeline } from '@gravity-ui/timeline';

const timestamp = Date.now();

// 타임라인 인스턴스 생성
const timeline = new Timeline({
  settings: {
    start: timestamp,
    end: timestamp + 3600000, // 지금부터 1시간 후
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
        from: timestamp + 1800000, // 지금부터 30분 후
        to: timestamp + 2400000,   // 지금부터 40분 후
        label: 'Sample Event',
        axisId: 'main'
      }
    ],
    markers: [
      {
        id: 'marker1',
        time: timestamp + 1200000, // 지금부터 20분 후
        label: 'Important Point',
        color: '#ff0000',
        activeColor: '#ff5252',
        hoverColor: '#ff1744'
      }
    ]
  },
  viewConfiguration: {
    // 선택적: 뷰 설정 사용자 지정
    zoomLevels: [1, 2, 4, 8, 16],
    hideRuler: false,
    showGrid: true
  }
});

// 캔버스 요소로 초기화
const canvas = document.querySelector('canvas');
if (canvas instanceof HTMLCanvasElement) {
  timeline.init(canvas);
}

// 이벤트 리스너 추가
timeline.on('on-click', (detail) => {
  console.log('Timeline clicked:', detail);
});

timeline.on('on-select-change', (detail) => {
  console.log('Selection changed:', detail);
});

// 완료 시 정리
timeline.destroy();
```

Timeline 클래스는 타임라인 관리를 위한 풍부한 API를 제공합니다:

- **이벤트 관리**:
  ```typescript
  // 이벤트 리스너 추가
  timeline.on('eventClick', (detail) => {
    console.log('Event clicked:', detail);
  });

  // 이벤트 리스너 제거
  const handler = (detail) => console.log(detail);
  timeline.on('eventClick', handler);
  timeline.off('eventClick', handler);

  // 사용자 지정 이벤트 발생
  timeline.emit('customEvent', { data: 'custom data' });
  ```

- **타임라인 제어**:
  ```typescript
  // 타임라인 데이터 업데이트
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

  // 축 업데이트
  timeline.api.setAxes([
    {
      id: 'newAxis',
      label: 'New Axis',
      color: '#0000ff'
    }
  ]);

  // 마커 업데이트
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

## 라이브 예제

인터랙티브 예제를 [Storybook](https://preview.gravity-ui.com/timeline/)에서 확인하세요:

- [기본 타임라인](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--basic) - 이벤트와 축이 포함된 간단한 타임라인
- [무한 타임라인](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--endless-timelines) - 이벤트와 축이 포함된 무한 타임라인
- [마커](https://preview.gravity-ui.com/timeline/?path=/story/timeline-markers--basic) - 수직 마커와 레이블이 포함된 타임라인
- [사용자 지정 이벤트](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--custom-renderer) - 사용자 지정 이벤트 렌더링이 포함된 타임라인


## 개발

### Storybook

이 프로젝트는 컴포넌트 개발과 문서를 위한 Storybook을 포함합니다.

Storybook 실행:

```bash
npm run storybook
```

이 명령은 포트 6006에서 Storybook 개발 서버를 시작합니다. http://localhost:6006에서 접근할 수 있습니다.

배포를 위한 Storybook의 정적 버전 빌드:

```bash
npm run build-storybook
```

## 라이선스

MIT