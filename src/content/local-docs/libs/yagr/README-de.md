# Ẏagr <img src="https://raw.githubusercontent.com/gravity-ui/yagr/main/docs/assets/yagr.svg" width="24px" height="24px" />

Yagr ist ein hochperformanter HTML5-Canvas-Diagramm-Renderer, der auf [uPlot](https://github.com/leeoniya/uPlot) basiert. Es bietet erweiterte Funktionen für uPlot-Diagramme.

<img src="https://raw.githubusercontent.com/gravity-ui/yagr/main/docs/assets/demo.png" width="800" />

## Funktionen

-   [Linien, Flächen, Säulen und Punkte als Visualisierungstypen. Konfigurierbar pro Serie](https://yagr.tech/en/api/visualization)
-   [Konfigurierbarer Legenden-Tooltip](https://yagr.tech/en/plugins/tooltip)
-   [Achsen mit zusätzlichen Optionen für dezimale Präzision](https://yagr.tech/en/api/axes)
-   [Skalen mit konfigurierbaren Bereichsfunktionen und Transformationen](https://yagr.tech/en/api/scales)
-   [Plot-Linien und -Bänder. Konfigurierbare Zeichnungsebene](https://yagr.tech/en/plugins/plot-lines)
-   [Responsive Diagramme](https://yagr.tech/en/api/settings#adaptivity) (erfordert [ResizeObserver](https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver))
-   [Erweiterte Unterstützung für gestapelte Flächen/Säulen](https://yagr.tech/en/api/scales#stacking)
-   [Konfigurierbare Marker](./docs/api/markers.md)
-   [Helles/Dunkles Theme](https://yagr.tech/en/api/settings#theme)
-   [Daten-Normalisierung](https://yagr.tech/en/api/scales#normalization)
-   [Konfigurierbare Fadenkreuze, Cursor-Marker und Snapping](https://yagr.tech/en/api/cursor)
-   Typescript
-   [Lokalisierung](https://yagr.tech/en/api/settings#localization)
-   [CSS-Variablen in Farbnamen](https://yagr.tech/en/api/css)
-   [Paginierte Inline-Legende](https://yagr.tech/en/plugins/legend)
-   [Fehlerbehandlung und erweiterte Hooks](https://yagr.tech/en/api/lifecycle)
-   [Daten-Ausrichtung und Interpolation für fehlende Daten](https://yagr.tech/en/api/data-processing)
-   [Echtzeit-Updates](https://yagr.tech/en/api/dynamic-updates)

## [Dokumentation](https://yagr.tech)

## Schneller Einstieg

```
npm i @gravity-ui/yagr
```

### NPM-Modul

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

### Script-Tag

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

### Beispiele

Suchen Sie etwas Spezielles? Yagr enthält nützliche Beispiele im Ordner [demo/examples](./demo/examples/). So starten Sie sie mit der aktuellen Version:

1. Klonen Sie das Repository.
2. Installieren Sie die Abhängigkeiten mit `npm i`.
3. Führen Sie `npm run build` aus.
4. Starten Sie `npx http-server .`.
5. Öffnen Sie die Beispiele im Browser gemäß der Ausgabe des http-Servers.