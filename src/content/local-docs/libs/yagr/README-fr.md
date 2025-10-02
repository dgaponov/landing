# Ẏagr <img src="https://raw.githubusercontent.com/gravity-ui/yagr/main/docs/assets/yagr.svg" width="24px" height="24px" />

Yagr est un moteur de rendu de graphiques HTML5 canvas haute performance basé sur [uPlot](https://github.com/leeoniya/uPlot). Il fournit des fonctionnalités de haut niveau pour les graphiques uPlot.

<img src="https://raw.githubusercontent.com/gravity-ui/yagr/main/docs/assets/demo.png" width="800" />

## Fonctionnalités

-   [Lignes, aires, colonnes et points comme types de visualisation. Configurable par série](https://yagr.tech/en/api/visualization)
-   [Infobulle de légende configurable](https://yagr.tech/en/plugins/tooltip)
-   [Axes avec des options supplémentaires pour une précision au niveau décimal](https://yagr.tech/en/api/axes)
-   [Échelles avec des fonctions de plage configurables et des transformations](https://yagr.tech/en/api/scales)
-   [Lignes de tracé et bandes. Couche de dessin configurable](https://yagr.tech/en/plugins/plot-lines)
-   [Graphiques adaptatifs](https://yagr.tech/en/api/settings#adaptivity) (nécessite [ResizeObserver](https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver))
-   [Support de haut niveau pour les aires/colonnes empilées](https://yagr.tech/en/api/scales#stacking)
-   [Marqueurs configurables](./docs/api/markers.md)
-   [Thème clair/sombre](https://yagr.tech/en/api/settings#theme)
-   [Normalisation des données](https://yagr.tech/en/api/scales#normalization)
-   [Réticules configurables, marqueurs de curseur et accrochage](https://yagr.tech/en/api/cursor)
-   Typescript
-   [Localisation](https://yagr.tech/en/api/settings#localization)
-   [Variables CSS dans les noms de couleurs](https://yagr.tech/en/api/css)
-   [Légende en ligne paginée](https://yagr.tech/en/plugins/legend)
-   [Gestion des erreurs et hooks étendus](https://yagr.tech/en/api/lifecycle)
-   [Alignement des données et interpolation pour les données manquantes](https://yagr.tech/en/api/data-processing)
-   [Mises à jour en temps réel](https://yagr.tech/en/api/dynamic-updates)

## [Documentation](https://yagr.tech)

## Démarrage rapide

```
npm i @gravity-ui/yagr
```

### Module NPM

```typescript
import Yagr from '@gravity-ui/yagr';

new Yagr(document.body, {
    timeline: [1, 2, 3, 4, 5],
    series: [
        {
            data: [1, 2, 3, 4, 5],
            color: 'red',
        },
        {
            data: [2, 3, 1, 4, 5],
            color: 'green',
        },
    ],
});
```

### Balise Script

```html
<script src="https://unpkg.com/@gravity-ui/yagr/dist/yagr.iife.min.js"></script>
<script>
    new Yagr(document.body, {
        timeline: [1, 2, 3, 4, 5],
        series: [
            {
                data: [1, 2, 3, 4, 5],
                color: 'red',
            },
            {
                data: [2, 3, 1, 4, 5],
                color: 'green',
            },
        ],
    });
</script>
```

### Exemples

Vous avez besoin de quelque chose de spécifique ? Yagr propose quelques exemples utiles dans le dossier [demo/examples](./demo/examples/). Voici comment les démarrer avec la version actuelle :

1. Clonez le dépôt.
2. Installez les dépendances `npm i`.
3. Exécutez `npm run build`.
4. Lancez `npx http-server .`.
5. Ouvrez les exemples dans le navigateur en suivant la sortie de http-server.