# @gravity-ui/timeline [![npm package](https://img.shields.io/npm/v/@gravity-ui/timeline)](https://www.npmjs.com/package/@gravity-ui/timeline) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/timeline/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/timeline/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/timeline/)

Une bibliothèque basée sur React pour créer des visualisations de chronologie interactives avec un rendu canvas.

## Documentation

Pour plus de détails, consultez la [Documentation](./docs/docs.md).

## Fonctionnalités

- Rendu basé sur canvas pour des performances élevées
- Chronologie interactive avec capacités de zoom et de déplacement
- Prise en charge des événements, marqueurs, axes et grille
- Regroupement intelligent des marqueurs avec zoom automatique vers le groupe - Cliquez sur les marqueurs regroupés pour zoomer sur leurs composants individuels
- Rendu virtualisé pour améliorer les performances avec de grands ensembles de données (activé uniquement lorsque le contenu de la chronologie dépasse la zone visible)
- Apparence et comportement personnalisables
- Prise en charge de TypeScript avec des définitions de types complètes
- Intégration React avec des hooks personnalisés

## Installation

```bash
npm install @gravity-ui/timeline
```

## Utilisation

Le composant de chronologie peut être utilisé dans des applications React avec la configuration de base suivante :

```tsx
import { TimelineCanvas, useTimeline } from '@gravity-ui/timeline/react';

const MyTimelineComponent = () => {
  const { timeline, api, start, stop } = useTimeline({
    settings: {
      start: Date.now(),
      end: Date.now() + 3600000, // 1 heure à partir de maintenant
      axes: [],
      events: [],
      markers: []
    },
    viewConfiguration: {
      // Configuration de vue optionnelle
    }
  });

  // timeline - Instance de Timeline
  // api - Instance de CanvasApi (identique à timeline.api)
  // start - Fonction pour initialiser la chronologie avec le canvas
  // stop - Fonction pour détruire la chronologie

  return (
    <div style={{ width: '100%', height: '100%' }}>
      <TimelineCanvas timeline={timeline} />
    </div>
  );
};
```

### Structure des marqueurs

Chaque marqueur nécessite la structure suivante :

```typescript
type TimelineMarker = {
  time: number;           // Horodatage pour la position du marqueur
  color: string;          // Couleur de la ligne du marqueur
  activeColor: string;    // Couleur lorsque le marqueur est sélectionné (obligatoire)
  hoverColor: string;     // Couleur lorsque le marqueur est survolé (obligatoire)
  lineWidth?: number;     // Largeur optionnelle de la ligne du marqueur
  label?: string;         // Texte du libellé optionnel
  labelColor?: string;    // Couleur du libellé optionnelle
  renderer?: AbstractMarkerRenderer; // Rendu personnalisé optionnel
  nonSelectable?: boolean;// Si le marqueur peut être sélectionné
  group?: boolean;        // Si le marqueur représente un groupe
};
```

### Regroupement des marqueurs et zoom

La chronologie regroupe automatiquement les marqueurs proches les uns des autres et propose une fonctionnalité de zoom :

```tsx
const MyTimelineComponent = () => {
  const { timeline } = useTimeline({
    settings: {
      start: Date.now(),
      end: Date.now() + 3600000,
      axes: [],
      events: [],
      markers: [
        // Ces marqueurs seront regroupés ensemble
        { time: Date.now(), color: '#ff0000', activeColor: '#ff5252', hoverColor: '#ff1744', label: 'Événement 1' },
        { time: Date.now() + 1000, color: '#ff0000', activeColor: '#ff5252', hoverColor: '#ff1744', label: 'Événement 2' },
        { time: Date.now() + 2000, color: '#ff0000', activeColor: '#ff5252', hoverColor: '#ff1744', label: 'Événement 3' },
      ]
    },
    viewConfiguration: {
      markers: {
        collapseMinDistance: 8,        // Regrouper les marqueurs dans un rayon de 8 pixels
        groupZoomEnabled: true,        // Activer le zoom au clic sur le groupe
        groupZoomPadding: 0.3,        // Marge de 30 % autour du groupe
        groupZoomMaxFactor: 0.3,      // Facteur de zoom maximal
      }
    }
  });

  // Écouter les événements de zoom sur groupe
  useTimelineEvent(timeline, 'on-group-marker-click', (data) => {
    console.log('Groupe zoomé :', data);
  });

  return <TimelineCanvas timeline={timeline} />;
};
```

## Fonctionnement

Le composant de chronologie est construit avec React et offre un moyen flexible de créer des visualisations de chronologie interactives. Voici comment il fonctionne :

### Architecture des composants

La chronologie est implémentée comme un composant React qui peut être configuré via deux objets principaux :

1. **TimelineSettings** : Contrôle le comportement et l'apparence principaux de la chronologie
   - `start` : Heure de début de la chronologie
   - `end` : Heure de fin de la chronologie
   - `axes` : Tableau de configurations d'axes
   - `events` : Tableau de configurations d'événements
   - `markers` : Tableau de configurations de marqueurs

2. **ViewConfiguration** : Gère la représentation visuelle et les paramètres d'interaction
   - Contrôle l'apparence, les niveaux de zoom et le comportement d'interaction
   - Peut être personnalisé ou utiliser les valeurs par défaut

### Gestion des événements

Le composant de chronologie prend en charge plusieurs événements interactifs :

- `on-click` : Déclenché lors d'un clic sur la chronologie
- `on-context-click` : Déclenché lors d'un clic droit/menu contextuel
- `on-select-change` : Déclenché lorsque la sélection change
- `on-hover` : Déclenché lors du survol des éléments de la chronologie
- `on-leave` : Déclenché lorsque la souris quitte les éléments de la chronologie

Exemple de gestion d'événements :

```tsx
import { useTimelineEvent } from '@gravity-ui/timeline/react';

const MyTimelineComponent = () => {
  const { timeline } = useTimeline({ /* ... */ });

  useTimelineEvent(timeline, 'on-click', (data) => {
    console.log('Chronologie cliquée :', data);
  });

  useTimelineEvent(timeline, 'on-select-change', (data) => {
    console.log('Sélection modifiée :', data);
  });

  return <TimelineCanvas timeline={timeline} />;
};
```

### Intégration React

Le composant utilise des hooks personnalisés pour la gestion de la chronologie :

- `useTimeline` : Gère l'instance de chronologie et son cycle de vie
  - Crée et initialise la chronologie
  - Gère le nettoyage lors du démontage du composant
  - Fournit l'accès à l'instance de chronologie

- `useTimelineEvent` : Gère les abonnements aux événements et le nettoyage
  - Gère le cycle de vie des écouteurs d'événements
  - Nettoie automatiquement les écouteurs lors du démontage

Le composant gère automatiquement le nettoyage et la destruction de l'instance de chronologie lors du démontage.

### Structure des événements

Les événements dans la chronologie suivent cette structure :

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

### Utilisation directe en TypeScript

La classe Timeline peut être utilisée directement en TypeScript sans React. Cela est utile pour l'intégration avec d'autres frameworks ou des applications JavaScript vanilla :

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

La classe Timeline fournit une API riche pour gérer la timeline :

- **Gestion des événements** :
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

- **Contrôle de la timeline** :
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

## Exemples en direct

Explorez des exemples interactifs dans notre [Storybook](https://preview.gravity-ui.com/timeline/) :

- [Timeline de base](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--basic) - Timeline simple avec événements et axes
- [Timeline infinie](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--endless-timelines) - Timeline infinie avec événements et axes
- [Marqueurs](https://preview.gravity-ui.com/timeline/?path=/story/timeline-markers--basic) - Timeline avec marqueurs verticaux et étiquettes
- [Événements personnalisés](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--custom-renderer) - Timeline avec rendu d'événements personnalisé


## Développement

### Storybook

Ce projet inclut Storybook pour le développement et la documentation des composants.

Pour lancer Storybook :

```bash
npm run storybook
```

Cela démarrera le serveur de développement Storybook sur le port 6006. Vous pouvez y accéder à http://localhost:6006.

Pour construire une version statique de Storybook pour le déploiement :

```bash
npm run build-storybook
```

## Licence

MIT