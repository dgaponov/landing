# @gravity-ui/timeline [![npm package](https://img.shields.io/npm/v/@gravity-ui/timeline)](https://www.npmjs.com/package/@gravity-ui/timeline) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/timeline/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/timeline/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/timeline/)

Una biblioteca basada en React para construir visualizaciones interactivas de líneas de tiempo con renderizado en canvas.

## Documentación

Para más detalles, consulta la [Documentación](./docs/docs.md).

## Características

- Renderizado basado en canvas para un alto rendimiento
- Línea de tiempo interactiva con capacidades de zoom y desplazamiento (pan)
- Soporte para eventos, marcadores, ejes y cuadrícula
- Agrupación inteligente de marcadores con zoom automático al grupo: haz clic en marcadores agrupados para hacer zoom en sus componentes individuales
- Renderizado virtualizado para mejorar el rendimiento con grandes conjuntos de datos (solo se activa cuando el contenido de la línea de tiempo excede el viewport)
- Apariencia y comportamiento personalizables
- Soporte para TypeScript con definiciones de tipos completas
- Integración con React mediante hooks personalizados

## Instalación

```bash
npm install @gravity-ui/timeline
```

## Uso

El componente de línea de tiempo se puede usar en aplicaciones React con la siguiente configuración básica:

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

### Estructura de Marcadores

Cada marcador requiere la siguiente estructura:

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

### Agrupación de Marcadores y Zoom

La línea de tiempo agrupa automáticamente los marcadores que están cerca uno del otro y proporciona funcionalidad de zoom:

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

## Cómo Funciona

El componente de línea de tiempo está construido usando React y proporciona una forma flexible de crear visualizaciones interactivas de líneas de tiempo. Aquí te explicamos cómo funciona:

### Arquitectura del Componente

La línea de tiempo se implementa como un componente de React que se puede configurar a través de dos objetos principales:

1. **TimelineSettings**: Controla el comportamiento y la apariencia principal de la línea de tiempo
   - `start`: Tiempo de inicio de la línea de tiempo
   - `end`: Tiempo de finalización de la línea de tiempo
   - `axes`: Arreglo de configuraciones de ejes
   - `events`: Arreglo de configuraciones de eventos
   - `markers`: Arreglo de configuraciones de marcadores

2. **ViewConfiguration**: Gestiona la representación visual y las configuraciones de interacción
   - Controla la apariencia, niveles de zoom y comportamiento de interacción
   - Se puede personalizar o usar valores predeterminados

### Manejo de Eventos

El componente de línea de tiempo soporta varios eventos interactivos:

- `on-click`: Se activa al hacer clic en la línea de tiempo
- `on-context-click`: Se activa con clic derecho/menú contextual
- `on-select-change`: Se dispara cuando cambia la selección
- `on-hover`: Se activa al pasar el cursor sobre elementos de la línea de tiempo
- `on-leave`: Se dispara cuando el cursor sale de los elementos de la línea de tiempo

Ejemplo de manejo de eventos:

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

### Integración con React

El componente utiliza hooks personalizados para la gestión de la línea de tiempo:

- `useTimeline`: Gestiona la instancia de la línea de tiempo y su ciclo de vida
  - Crea e inicializa la línea de tiempo
  - Maneja la limpieza al desmontar el componente
  - Proporciona acceso a la instancia de la línea de tiempo

- `useTimelineEvent`: Maneja las suscripciones a eventos y la limpieza
  - Gestiona el ciclo de vida de los oyentes de eventos
  - Limpia automáticamente los oyentes al desmontar

El componente maneja automáticamente la limpieza y destrucción de la instancia de la línea de tiempo al desmontarse.

### Estructura de Eventos

Los eventos en la línea de tiempo siguen esta estructura:

```typescript
type TimelineEvent = {
  id: string;             // Identificador único
  from: number;           // Marca de tiempo de inicio
  to?: number;            // Marca de tiempo de fin (opcional para eventos puntuales)
  axisId: string;         // ID del eje al que pertenece este evento
  trackIndex: number;     // Índice en la pista del eje
  renderer?: AbstractEventRenderer; // Renderizador personalizado opcional
  color?: string;         // Color del evento opcional
  selectedColor?: string; // Color del estado seleccionado opcional
};
```

### Uso directo de TypeScript

La clase Timeline se puede usar directamente en TypeScript sin React. Esto es útil para integrarla con otros frameworks o aplicaciones de JavaScript vanilla:

```typescript
import { Timeline } from '@gravity-ui/timeline';

const timestamp = Date.now();

// Crear una instancia de timeline
const timeline = new Timeline({
  settings: {
    start: timestamp,
    end: timestamp + 3600000, // 1 hora a partir de ahora
    axes: [
      {
        id: 'main',
        label: 'Eje Principal',
        color: '#000000'
      }
    ],
    events: [
      {
        id: 'event1',
        from: timestamp + 1800000, // 30 minutos a partir de ahora
        to: timestamp + 2400000,   // 40 minutos a partir de ahora
        label: 'Evento de Ejemplo',
        axisId: 'main'
      }
    ],
    markers: [
      {
        id: 'marker1',
        time: timestamp + 1200000, // 20 minutos a partir de ahora
        label: 'Punto Importante',
        color: '#ff0000',
        activeColor: '#ff5252',
        hoverColor: '#ff1744'
      }
    ]
  },
  viewConfiguration: {
    // Opcional: personalizar configuraciones de vista
    zoomLevels: [1, 2, 4, 8, 16],
    hideRuler: false,
    showGrid: true
  }
});

// Inicializar con un elemento canvas
const canvas = document.querySelector('canvas');
if (canvas instanceof HTMLCanvasElement) {
  timeline.init(canvas);
}

// Agregar escuchadores de eventos
timeline.on('on-click', (detail) => {
  console.log('Timeline clicked:', detail);
});

timeline.on('on-select-change', (detail) => {
  console.log('Selection changed:', detail);
});

// Limpiar cuando se termine
timeline.destroy();
```

La clase Timeline proporciona una API rica para gestionar la línea de tiempo:

- **Gestión de eventos**:
  ```typescript
  // Agregar escuchador de eventos
  timeline.on('eventClick', (detail) => {
    console.log('Event clicked:', detail);
  });

  // Eliminar escuchador de eventos
  const handler = (detail) => console.log(detail);
  timeline.on('eventClick', handler);
  timeline.off('eventClick', handler);

  // Emitir eventos personalizados
  timeline.emit('customEvent', { data: 'custom data' });
  ```

- **Control de la línea de tiempo**:
  ```typescript
  // Actualizar datos de la línea de tiempo
  timeline.api.setEvents([
    {
      id: 'newEvent',
      from: Date.now(),
      to: Date.now() + 3600000,
      label: 'Nuevo Evento',
      axisId: 'main',
      trackIndex: 0
    }
  ]);

  // Actualizar ejes
  timeline.api.setAxes([
    {
      id: 'newAxis',
      label: 'Nuevo Eje',
      color: '#0000ff'
    }
  ]);

  // Actualizar marcadores
  timeline.api.setMarkers([
    {
      id: 'newMarker',
      time: Date.now(),
      label: 'Nuevo Marcador',
      color: '#00ff00',
      activeColor: '#4caf50',
      hoverColor: '#2e7d32'
    }
  ]);
  ```

## Ejemplos en vivo

Explora ejemplos interactivos en nuestro [Storybook](https://preview.gravity-ui.com/timeline/):

- [Línea de tiempo básica](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--basic) - Línea de tiempo simple con eventos y ejes
- [Línea de tiempo infinita](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--endless-timelines) - Línea de tiempo infinita con eventos y ejes
- [Marcadores](https://preview.gravity-ui.com/timeline/?path=/story/timeline-markers--basic) - Línea de tiempo con marcadores verticales y etiquetas
- [Eventos personalizados](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--custom-renderer) - Línea de tiempo con renderizado personalizado de eventos


## Desarrollo

### Storybook

Este proyecto incluye Storybook para el desarrollo y documentación de componentes.

Para ejecutar Storybook:

```bash
npm run storybook
```

Esto iniciará el servidor de desarrollo de Storybook en el puerto 6006. Puedes acceder a él en http://localhost:6006.

Para compilar una versión estática de Storybook para despliegue:

```bash
npm run build-storybook
```

## Licencia

MIT