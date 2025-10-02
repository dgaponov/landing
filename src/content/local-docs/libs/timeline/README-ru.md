# @gravity-ui/timeline [![npm package](https://img.shields.io/npm/v/@gravity-ui/timeline)](https://www.npmjs.com/package/@gravity-ui/timeline) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/timeline/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/timeline/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/timeline/)

Библиотека на базе React для создания интерактивных визуализаций временных шкал с рендерингом на canvas.

## Документация

Подробности см. в [Документации](./docs/docs.md).

## Возможности

- Рендеринг на основе canvas для высокой производительности
- Интерактивная временная шкала с возможностями масштабирования и панорамирования
- Поддержка событий, маркеров, осей и сетки
- Умная группировка маркеров с автоматическим масштабированием до группы — кликните по сгруппированным маркерам, чтобы увеличить их отдельные компоненты
- Виртуализированный рендеринг для улучшения производительности с большими наборами данных (активируется только когда содержимое временной шкалы превышает область просмотра)
- Настраиваемый внешний вид и поведение
- Поддержка TypeScript с полными определениями типов
- Интеграция с React через пользовательские хуки

## Установка

```bash
npm install @gravity-ui/timeline
```

## Использование

Компонент временной шкалы можно использовать в React-приложениях с такой базовой настройкой:

```tsx
import { TimelineCanvas, useTimeline } from '@gravity-ui/timeline/react';

const MyTimelineComponent = () => {
  const { timeline, api, start, stop } = useTimeline({
    settings: {
      start: Date.now(),
      end: Date.now() + 3600000, // 1 час от текущего времени
      axes: [],
      events: [],
      markers: []
    },
    viewConfiguration: {
      // Необязательная конфигурация вида
    }
  });

  // timeline - экземпляр Timeline
  // api - экземпляр CanvasApi (тот же, что и timeline.api)
  // start - функция для инициализации временной шкалы с canvas
  // stop - функция для уничтожения временной шкалы

  return (
    <div style={{ width: '100%', height: '100%' }}>
      <TimelineCanvas timeline={timeline} />
    </div>
  );
};
```

### Структура маркера

Каждый маркер требует следующей структуры:

```typescript
type TimelineMarker = {
  time: number;           // Метка времени для позиции маркера
  color: string;          // Цвет линии маркера
  activeColor: string;    // Цвет при выборе маркера (обязательно)
  hoverColor: string;     // Цвет при наведении на маркер (обязательно)
  lineWidth?: number;     // Необязательная ширина линии маркера
  label?: string;         // Необязательный текст метки
  labelColor?: string;    // Необязательный цвет метки
  renderer?: AbstractMarkerRenderer; // Необязательный пользовательский рендерер
  nonSelectable?: boolean;// Можно ли выбрать маркер
  group?: boolean;        // Является ли маркер группой
};
```

### Группировка маркеров и масштабирование

Временная шкала автоматически группирует маркеры, которые расположены близко друг к другу, и предоставляет функциональность масштабирования:

```tsx
const MyTimelineComponent = () => {
  const { timeline } = useTimeline({
    settings: {
      start: Date.now(),
      end: Date.now() + 3600000,
      axes: [],
      events: [],
      markers: [
        // Эти маркеры будут сгруппированы вместе
        { time: Date.now(), color: '#ff0000', activeColor: '#ff5252', hoverColor: '#ff1744', label: 'Event 1' },
        { time: Date.now() + 1000, color: '#ff0000', activeColor: '#ff5252', hoverColor: '#ff1744', label: 'Event 2' },
        { time: Date.now() + 2000, color: '#ff0000', activeColor: '#ff5252', hoverColor: '#ff1744', label: 'Event 3' },
      ]
    },
    viewConfiguration: {
      markers: {
        collapseMinDistance: 8,        // Группировать маркеры в пределах 8 пикселей
        groupZoomEnabled: true,        // Включить масштабирование при клике по группе
        groupZoomPadding: 0.3,        // 30% отступа вокруг группы
        groupZoomMaxFactor: 0.3,      // Максимальный коэффициент масштабирования
      }
    }
  });

  // Отслеживание событий масштабирования группы
  useTimelineEvent(timeline, 'on-group-marker-click', (data) => {
    console.log('Group zoomed:', data);
  });

  return <TimelineCanvas timeline={timeline} />;
};
```

## Как это работает

Компонент временной шкалы построен с использованием React и предоставляет гибкий способ создания интерактивных визуализаций временных шкал. Вот как это работает:

### Архитектура компонента

Временная шкала реализована как React-компонент, который настраивается через два основных объекта:

1. **TimelineSettings**: Управляет основным поведением и внешним видом временной шкалы
   - `start`: Начальное время временной шкалы
   - `end`: Конечное время временной шкалы
   - `axes`: Массив конфигураций осей
   - `events`: Массив конфигураций событий
   - `markers`: Массив конфигураций маркеров

2. **ViewConfiguration**: Управляет визуальным представлением и настройками взаимодействия
   - Управляет внешним видом, уровнями масштабирования и поведением взаимодействия
   - Может быть настроена или использовать значения по умолчанию

### Обработка событий

Компонент временной шкалы поддерживает несколько интерактивных событий:

- `on-click`: Срабатывает при клике по временной шкале
- `on-context-click`: Срабатывает при правом клике/контекстном меню
- `on-select-change`: Срабатывает при изменении выбора
- `on-hover`: Срабатывает при наведении на элементы временной шкалы
- `on-leave`: Срабатывает при уходе мыши с элементов временной шкалы

Пример обработки событий:

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

### Интеграция с React

Компонент использует пользовательские хуки для управления временной шкалой:

- `useTimeline`: Управляет экземпляром временной шкалы и её жизненным циклом
  - Создаёт и инициализирует временную шкалу
  - Обрабатывает очистку при размонтировании компонента
  - Предоставляет доступ к экземпляру временной шкалы

- `useTimelineEvent`: Обрабатывает подписку на события и очистку
  - Управляет жизненным циклом обработчиков событий
  - Автоматически очищает обработчики при размонтировании

Компонент автоматически обрабатывает очистку и уничтожение экземпляра временной шкалы при размонтировании.

### Структура событий

События в временной шкале следуют этой структуре:

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

### Прямое использование TypeScript

Класс Timeline можно использовать напрямую в TypeScript без React. Это полезно для интеграции с другими фреймворками или обычными JavaScript-приложениями:

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

Класс Timeline предоставляет богатый API для управления временной шкалой:

- **Управление событиями**:
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

- **Управление временной шкалой**:
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

## Живые примеры

Изучите интерактивные примеры в нашем [Storybook](https://preview.gravity-ui.com/timeline/):

- [Basic Timeline](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--basic) - Простая временная шкала с событиями и осями
- [Endless Timeline](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--endless-timelines) - Бесконечная временная шкала с событиями и осями
- [Markers](https://preview.gravity-ui.com/timeline/?path=/story/timeline-markers--basic) - Временная шкала с вертикальными маркерами и подписями
- [Custom Events](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--custom-renderer) - Временная шкала с пользовательским рендерингом событий


## Разработка

### Storybook

Этот проект включает Storybook для разработки компонентов и документации.

Чтобы запустить Storybook:

```bash
npm run storybook
```

Это запустит сервер разработки Storybook на порту 6006. Вы можете открыть его по адресу http://localhost:6006.

Чтобы собрать статическую версию Storybook для развертывания:

```bash
npm run build-storybook
```

## Лицензия

MIT