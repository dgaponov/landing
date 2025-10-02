# @gravity-ui/timeline [![npm package](https://img.shields.io/npm/v/@gravity-ui/timeline)](https://www.npmjs.com/package/@gravity-ui/timeline) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/timeline/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/timeline/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/timeline/)

Une bibliothèque basée sur React pour créer des visualisations de chronologie interactives avec un rendu canvas.

## Documentation

Pour plus de détails, consultez la [Documentation](./docs/docs.md).

## Fonctionnalités

- Rendu basé sur Canvas pour des performances élevées
- Chronologie interactive avec capacités de zoom et de déplacement
- Support pour les événements, les marqueurs, les axes et la grille
- Regroupement intelligent des marqueurs avec zoom automatique vers le groupe - Cliquez sur les marqueurs regroupés pour zoomer sur leurs composants individuels
- Rendu virtualisé pour améliorer les performances avec de grands ensembles de données (activé uniquement lorsque le contenu de la chronologie dépasse la zone visible)
- Apparence et comportement personnalisables
- Support TypeScript avec des définitions de types complètes
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
  time: number;           // Timestamp pour la position du marqueur
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

La chronologie regroupe automatiquement les marqueurs qui sont proches les uns des autres et propose une fonctionnalité de zoom :

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
        { time: Date.now(), color: '#ff0000', activeColor: '#ff5252', hoverColor: '#ff1744', label: 'Event 1' },
        { time: Date.now() + 1000, color: '#ff0000', activeColor: '#ff5252', hoverColor: '#ff1744', label: 'Event 2' },
        { time: Date.now() + 2000, color: '#ff0000', activeColor: '#ff5252', hoverColor: '#ff1744', label: 'Event 3' },
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
    console.log('Group zoomed:', data);
  });

  return <TimelineCanvas timeline={timeline} />;
};
```

## Fonctionnement

Le composant de chronologie est construit avec React et offre une façon flexible de créer des visualisations de chronologie interactives. Voici comment il fonctionne :

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

Le composant de chronologie supporte plusieurs événements interactifs :

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
    console.log('Timeline clicked:', data);
  });

  useTimelineEvent(timeline, 'on-select-change', (data) => {
    console.log('Selection changed:', data);
  });

  return <TimelineCanvas timeline={timeline} />;
};
```

### Intégration React

Le composant utilise des hooks personnalisés pour la gestion de la chronologie :

- `useTimeline` : Gère l'instance de la chronologie et son cycle de vie
  - Crée et initialise la chronologie
  - Gère le nettoyage lors du démontage du composant
  - Fournit l'accès à l'instance de la chronologie

- `useTimelineEvent` : Gère les abonnements aux événements et le nettoyage
  - Gère le cycle de vie des écouteurs d'événements
  - Nettoie automatiquement les écouteurs lors du démontage

Le composant gère automatiquement le nettoyage et la destruction de l'instance de la chronologie lors du démontage.

### Structure des événements

Les événements dans la chronologie suivent cette structure :

```typescript
type TimelineEvent = {
  id: string;             // Identifiant unique
  from: number;           // Horodatage de début
  to?: number;            // Horodatage de fin (optionnel pour les événements ponctuels)
  axisId: string;         // ID de l'axe auquel cet événement appartient
  trackIndex: number;     // Index dans la piste de l'axe
  renderer?: AbstractEventRenderer; // Rendu personnalisé optionnel
  color?: string;         // Couleur de l'événement optionnelle
  selectedColor?: string; // Couleur de l'état sélectionné optionnelle
};
```

### Utilisation directe en TypeScript

La classe Timeline peut être utilisée directement en TypeScript sans React. Cela est utile pour l'intégration avec d'autres frameworks ou des applications JavaScript vanilla :

```typescript
import { Timeline } from '@gravity-ui/timeline';

const timestamp = Date.now();

// Créer une instance de timeline
const timeline = new Timeline({
  settings: {
    start: timestamp,
    end: timestamp + 3600000, // 1 heure à partir de maintenant
    axes: [
      {
        id: 'main',
        label: 'Axe Principal',
        color: '#000000'
      }
    ],
    events: [
      {
        id: 'event1',
        from: timestamp + 1800000, // 30 minutes à partir de maintenant
        to: timestamp + 2400000,   // 40 minutes à partir de maintenant
        label: 'Événement Exemple',
        axisId: 'main'
      }
    ],
    markers: [
      {
        id: 'marker1',
        time: timestamp + 1200000, // 20 minutes à partir de maintenant
        label: 'Point Important',
        color: '#ff0000',
        activeColor: '#ff5252',
        hoverColor: '#ff1744'
      }
    ]
  },
  viewConfiguration: {
    // Optionnel : personnaliser les paramètres d'affichage
    zoomLevels: [1, 2, 4, 8, 16],
    hideRuler: false,
    showGrid: true
  }
});

// Initialiser avec un élément canvas
const canvas = document.querySelector('canvas');
if (canvas instanceof HTMLCanvasElement) {
  timeline.init(canvas);
}

// Ajouter des écouteurs d'événements
timeline.on('on-click', (detail) => {
  console.log('Timeline cliquée :', detail);
});

timeline.on('on-select-change', (detail) => {
  console.log('Sélection modifiée :', detail);
});

// Nettoyer à la fin
timeline.destroy();
```

La classe Timeline fournit une API riche pour gérer la timeline :

- **Gestion des événements** :
  ```typescript
  // Ajouter un écouteur d'événement
  timeline.on('eventClick', (detail) => {
    console.log('Événement cliqué :', detail);
  });

  // Supprimer un écouteur d'événement
  const handler = (detail) => console.log(detail);
  timeline.on('eventClick', handler);
  timeline.off('eventClick', handler);

  // Émettre des événements personnalisés
  timeline.emit('customEvent', { data: 'données personnalisées' });
  ```

- **Contrôle de la timeline** :
  ```typescript
  // Mettre à jour les données de la timeline
  timeline.api.setEvents([
    {
      id: 'newEvent',
      from: Date.now(),
      to: Date.now() + 3600000,
      label: 'Nouvel Événement',
      axisId: 'main',
      trackIndex: 0
    }
  ]);

  // Mettre à jour les axes
  timeline.api.setAxes([
    {
      id: 'newAxis',
      label: 'Nouvel Axe',
      color: '#0000ff'
    }
  ]);

  // Mettre à jour les marqueurs
  timeline.api.setMarkers([
    {
      id: 'newMarker',
      time: Date.now(),
      label: 'Nouveau Marqueur',
      color: '#00ff00',
      activeColor: '#4caf50',
      hoverColor: '#2e7d32'
    }
  ]);
  ```

## Exemples en direct

Explorez des exemples interactifs dans notre [Storybook](https://preview.gravity-ui.com/timeline/):

- [Timeline de base](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--basic) - Timeline simple avec événements et axes
- [Timeline infinie](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--endless-timelines) - Timeline infinie avec événements et axes
- [Marqueurs](https://preview.gravity-ui.com/timeline/?path=/story/timeline-markers--basic) - Timeline avec marqueurs verticaux et étiquettes
- [Événements personnalisés](https://preview.gravity-ui.com/timeline/?path=/story/timeline-events--custom-renderer) - Timeline avec rendu personnalisé d'événements


## Développement

### Storybook

Ce projet inclut Storybook pour le développement et la documentation des composants.

Pour lancer Storybook :

```bash
npm run storybook
```

Cela démarrera le serveur de développement Storybook sur le port 6006. Vous pouvez y accéder à l'adresse http://localhost:6006.

Pour générer une version statique de Storybook pour le déploiement :

```bash
npm run build-storybook
```

## Licence

MIT