# @gravity-ui/timeline [![npm package](https://img.shields.io/npm/v/@gravity-ui/timeline)](https://www.npmjs.com/package/@gravity-ui/timeline) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/timeline/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/timeline/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/timeline/)

Eine auf React basierende Bibliothek zum Erstellen interaktiver Timeline-Visualisierungen mit Canvas-Rendering.

## Dokumentation

Für Details siehe [Dokumentation](./docs/docs.md).

## Features

- Canvas-basiertes Rendering für hohe Performance
- Interaktive Timeline mit Zoom- und Pan-Fähigkeiten
- Unterstützung für Events, Marker, Achsen und Gitter
- Intelligente Marker-Gruppierung mit automatischer Zoom-Funktion zur Gruppe – Klicken Sie auf gruppierte Marker, um in ihre individuellen Komponenten zu zoomen
- Virtualisiertes Rendering für bessere Performance bei großen Datensätzen (nur aktiv, wenn der Timeline-Inhalt das Sichtfenster überschreitet)
- Anpassbares Aussehen und Verhalten
- TypeScript-Unterstützung mit vollständigen Typdefinitionen
- React-Integration mit benutzerdefinierten Hooks

## Installation

```bash
npm install @gravity-ui/timeline
```

## Nutzung

Die Timeline-Komponente kann in React-Anwendungen mit der folgenden grundlegenden Einrichtung verwendet werden:

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

### Marker-Struktur

Jeder Marker erfordert die folgende Struktur:

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

### Marker-Gruppierung und Zoom

Die Timeline gruppiert Marker automatisch, die nah beieinander liegen, und bietet Zoom-Funktionalität:

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

## So funktioniert es

Die Timeline-Komponente ist mit React aufgebaut und bietet einen flexiblen Weg, interaktive Timeline-Visualisierungen zu erstellen. Hier ist, wie sie funktioniert:

### Komponentenarchitektur

Die Timeline wird als React-Komponente implementiert, die über zwei Hauptobjekte konfiguriert werden kann:

1. **TimelineSettings**: Steuert das Kernverhalten und Aussehen der Timeline
   - `start`: Startzeit der Timeline
   - `end`: Endzeit der Timeline
   - `axes`: Array von Achsenkonfigurationen
   - `events`: Array von Event-Konfigurationen
   - `markers`: Array von Marker-Konfigurationen

2. **ViewConfiguration**: Verwaltet die visuelle Darstellung und Interaktionseinstellungen
   - Steuert Aussehen, Zoom-Level und Interaktionsverhalten
   - Kann angepasst oder Standardwerte verwendet werden

### Event-Behandlung

Die Timeline-Komponente unterstützt mehrere interaktive Events:

- `on-click`: Wird ausgelöst, wenn auf die Timeline geklickt wird
- `on-context-click`: Wird bei Rechtsklick/Kontextmenü ausgelöst
- `on-select-change`: Wird ausgelöst, wenn die Auswahl geändert wird
- `on-hover`: Wird ausgelöst, wenn über Timeline-Elemente gehovert wird
- `on-leave`: Wird ausgelöst, wenn die Maus Timeline-Elemente verlässt

Beispiel für Event-Behandlung:

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

### React-Integration

Die Komponente verwendet benutzerdefinierte Hooks für das Timeline-Management:

- `useTimeline`: Verwaltet die Timeline-Instanz und ihren Lebenszyklus
  - Erstellt und initialisiert die Timeline
  - Handhabt Cleanup beim Unmount der Komponente
  - Bietet Zugriff auf die Timeline-Instanz

- `useTimelineEvent`: Handhabt Event-Abonnements und Cleanup
  - Verwaltet den Lebenszyklus von Event-Listenern
  - Räumt Listener automatisch beim Unmount auf

Die Komponente handhabt automatisch das Cleanup und die Zerstörung der Timeline-Instanz beim Unmount.

### Event-Struktur

Events in der Timeline folgen dieser Struktur:

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

### Direkte TypeScript-Nutzung

Die Timeline-Klasse kann direkt in TypeScript ohne React verwendet werden. Das ist nützlich für die Integration in andere Frameworks oder reine JavaScript-Anwendungen:

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

Die Timeline-Klasse bietet eine umfangreiche API zur Verwaltung der Timeline:

- **Event-Verwaltung**:
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

- **Timeline-Steuerung**:
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

## Live-Beispiele

Erkunden Sie interaktive Beispiele in unserem [Storybook](https://preview.gravity-ui.com/timeline/):

- [Basic Timeline](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--basic) - Einfache Timeline mit Events und Achsen
- [Endless Timeline](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--endless-timelines) - Endlose Timeline mit Events und Achsen
- [Markers](https://preview.gravity-ui.com/timeline/?path=/story/timeline-markers--basic) - Timeline mit vertikalen Markern und Beschriftungen
- [Custom Events](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--custom-renderer) - Timeline mit benutzerdefinierter Event-Darstellung


## Entwicklung

### Storybook

Dieses Projekt enthält Storybook für die Komponentenentwicklung und Dokumentation.

Um Storybook zu starten:

```bash
npm run storybook
```

Dadurch wird der Storybook-Entwicklungsserver auf Port 6006 gestartet. Sie können ihn unter http://localhost:6006 aufrufen.

Um eine statische Version von Storybook für die Bereitstellung zu erstellen:

```bash
npm run build-storybook
```

## Lizenz

MIT