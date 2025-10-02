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

Компонент временной шкалы можно использовать в приложениях React с такой базовой настройкой:

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

### Структура маркера

Каждый маркер требует следующей структуры:

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

### Группировка маркеров и масштабирование

Временная шкала автоматически группирует маркеры, расположенные близко друг к другу, и предоставляет функциональность масштабирования:

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

## Как это работает

Компонент временной шкалы построен с использованием React и предоставляет гибкий способ создания интерактивных визуализаций временных шкал. Вот как это работает:

### Архитектура компонента

Временная шкала реализована как компонент React, который можно настроить через два основных объекта:

1. **TimelineSettings**: Управляет основным поведением и внешним видом временной шкалы
   - `start`: Время начала временной шкалы
   - `end`: Время окончания временной шкалы
   - `axes`: Массив конфигураций осей
   - `events`: Массив конфигураций событий
   - `markers`: Массив конфигураций маркеров

2. **ViewConfiguration**: Управляет визуальным представлением и настройками взаимодействия
   - Контролирует внешний вид, уровни масштабирования и поведение взаимодействия
   - Может быть настроен или использовать значения по умолчанию

### Обработка событий

Компонент временной шкалы поддерживает несколько интерактивных событий:

- `on-click`: Срабатывает при клике по временной шкале
- `on-context-click`: Срабатывает при правом клике/вызове контекстного меню
- `on-select-change`: Срабатывает при изменении выделения
- `on-hover`: Срабатывает при наведении на элементы временной шкалы
- `on-leave`: Срабатывает при уходе мыши от элементов временной шкалы

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

- `useTimelineEvent`: Обрабатывает подписки на события и очистку
  - Управляет жизненным циклом обработчиков событий
  - Автоматически очищает обработчики при размонтировании

Компонент автоматически обрабатывает очистку и уничтожение экземпляра временной шкалы при размонтировании.

### Структура событий

События в временной шкале следуют этой структуре:

```typescript
type TimelineEvent = {
  id: string;             // Уникальный идентификатор
  from: number;           // Метка времени начала
  to?: number;            // Метка времени окончания (опционально для событий-точек)
  axisId: string;         // ID оси, к которой относится событие
  trackIndex: number;     // Индекс в треке оси
  renderer?: AbstractEventRenderer; // Опциональный пользовательский рендерер
  color?: string;         // Опциональный цвет события
  selectedColor?: string; // Опциональный цвет в выбранном состоянии
};
```

### Прямое использование в TypeScript

Класс Timeline можно использовать напрямую в TypeScript без React. Это полезно для интеграции с другими фреймворками или обычными JavaScript-приложениями:

```typescript
import { Timeline } from '@gravity-ui/timeline';

const timestamp = Date.now();

// Создание экземпляра timeline
const timeline = new Timeline({
  settings: {
    start: timestamp,
    end: timestamp + 3600000, // 1 час от текущего времени
    axes: [
      {
        id: 'main',
        label: 'Основная ось',
        color: '#000000'
      }
    ],
    events: [
      {
        id: 'event1',
        from: timestamp + 1800000, // 30 минут от текущего времени
        to: timestamp + 2400000,   // 40 минут от текущего времени
        label: 'Пример события',
        axisId: 'main'
      }
    ],
    markers: [
      {
        id: 'marker1',
        time: timestamp + 1200000, // 20 минут от текущего времени
        label: 'Важная точка',
        color: '#ff0000',
        activeColor: '#ff5252',
        hoverColor: '#ff1744'
      }
    ]
  },
  viewConfiguration: {
    // Опционально: настройка вида
    zoomLevels: [1, 2, 4, 8, 16],
    hideRuler: false,
    showGrid: true
  }
});

// Инициализация с элементом canvas
const canvas = document.querySelector('canvas');
if (canvas instanceof HTMLCanvasElement) {
  timeline.init(canvas);
}

// Добавление обработчиков событий
timeline.on('on-click', (detail) => {
  console.log('Клик по timeline:', detail);
});

timeline.on('on-select-change', (detail) => {
  console.log('Выбор изменён:', detail);
});

// Очистка при завершении
timeline.destroy();
```

Класс Timeline предоставляет богатый API для управления timeline:

- **Управление событиями**:
  ```typescript
  // Добавление обработчика события
  timeline.on('eventClick', (detail) => {
    console.log('Событие кликнуто:', detail);
  });

  // Удаление обработчика события
  const handler = (detail) => console.log(detail);
  timeline.on('eventClick', handler);
  timeline.off('eventClick', handler);

  // Выпуск пользовательских событий
  timeline.emit('customEvent', { data: 'пользовательские данные' });
  ```

- **Управление timeline**:
  ```typescript
  // Обновление данных событий
  timeline.api.setEvents([
    {
      id: 'newEvent',
      from: Date.now(),
      to: Date.now() + 3600000,
      label: 'Новое событие',
      axisId: 'main',
      trackIndex: 0
    }
  ]);

  // Обновление осей
  timeline.api.setAxes([
    {
      id: 'newAxis',
      label: 'Новая ось',
      color: '#0000ff'
    }
  ]);

  // Обновление маркеров
  timeline.api.setMarkers([
    {
      id: 'newMarker',
      time: Date.now(),
      label: 'Новый маркер',
      color: '#00ff00',
      activeColor: '#4caf50',
      hoverColor: '#2e7d32'
    }
  ]);
  ```

## Живые примеры

Изучите интерактивные примеры в нашем [Storybook](https://preview.gravity-ui.com/timeline/):

- [Basic Timeline](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--basic) - Простой timeline с событиями и осями
- [Endless Timeline](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--endless-timelines) - Бесконечный timeline с событиями и осями
- [Markers](https://preview.gravity-ui.com/timeline/?path=/story/timeline-markers--basic) - Timeline с вертикальными маркерами и подписями
- [Custom Events](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--custom-renderer) - Timeline с пользовательским рендерингом событий


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