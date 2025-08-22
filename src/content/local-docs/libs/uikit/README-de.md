# UIKit &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/uikit)](https://www.npmjs.com/package/@gravity-ui/uikit) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/uikit/.github/workflows/ci.yml?branch=main&label=CI&logo=github)](https://github.com/gravity-ui/uikit/actions/workflows/ci.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/uikit/)

<p align="center">
  <a href="README.md">English</a> |
  <a href="README.zh-CN.md">简体中文</a> |
  <span>Deutsch</span> |
  <a href="README.fr-FR.md">Français</a>
</p>

Eine Sammlung flexibler, hochpraktischer und effizienter React-Komponenten zur Erstellung anspruchsvoller Webanwendungen.

<!--GITHUB_BLOCK-->

![Cover-Bild](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/uikit_cover.png)

## Ressourcen

### ![Globus-Logo Hell](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/globe_light.svg#gh-light-mode-only) ![Globus-Logo Dunkel](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/globe_dark.svg#gh-dark-mode-only) [Website](https://gravity-ui.com)

### ![Dokumentations-Logo Hell](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/book-open_light.svg#gh-light-mode-only) ![Dokumentations-Logo Dunkel](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/book-open_dark.svg#gh-dark-mode-only) [Dokumentation](https://gravity-ui.com/components/uikit/alert)

### ![Figma-Logo Hell](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/figma_light.svg#gh-light-mode-only) ![Figma-Logo Dunkel](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/figma_dark.svg#gh-dark-mode-only) [Figma](https://www.figma.com/community/file/1271150067798118027/Gravity-UI-Design-System-(Beta))

### ![Themer-Logo Hell](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/bucket-paint_light.svg#gh-light-mode-only) ![Themer-Logo Dunkel](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/bucket-paint_dark.svg#gh-dark-mode-only) [Themer](https://gravity-ui.com/themer)

### ![Storybook-Logo Hell](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/storybook_light.svg#gh-light-mode-only) ![Storybook-Logo Dunkel](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/storybook_dark.svg#gh-dark-mode-only) [Storybook](https://preview.gravity-ui.com/uikit/)

### ![Community-Logo Hell](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/telegram_light.svg#gh-light-mode-only) ![Community-Logo Dunkel](https://raw.githubusercontent.com/gravity-ui/uikit/main/docs/assets/telegram_dark.svg#gh-dark-mode-only) [Community](https://t.me/gravity_ui)

<!--/GITHUB_BLOCK-->

## Installation

```shell
npm install --save-dev @gravity-ui/uikit
```

## Verwendung

```jsx
import {Button} from '@gravity-ui/uikit';

const SubmitButton = <Button view="action" size="l" />;
```

### Styling

UIKit wird mit einem grundlegenden Stil und Theme ausgeliefert. Füge dies oben in deiner Hauptdatei ein, damit alles korrekt angezeigt wird:

```js
// index.js

import '@gravity-ui/uikit/styles/fonts.css';
import '@gravity-ui/uikit/styles/styles.css';

// ...
```

UIKit unterstützt verschiedene Themes: Hell, Dunkel und deren Kontrastvarianten. Deine Anwendung muss innerhalb von `ThemeProvider` gerendert werden:

```js
import {createRoot} from 'react-dom/client';
import {ThemeProvider} from '@gravity-ui/uikit';

const root = createRoot(document.getElementById('root'));
root.render(
  <ThemeProvider theme="light">
    <App />
  </ThemeProvider>,
);
```

Für SSR können anfängliche CSS-Root-Klassen generiert werden, um Theme-Flackern zu vermeiden:

```js
import {getRootClassName} from '@gravity-ui/uikit/server';

const theme = 'dark';
const rootClassName = getRootClassName({theme});

const html = `
<html>
  <body>
    <div id="root" class="${rootClassName}"></div>
  </body>
</html>
`;
```

Zusätzlich steht eine SCSS-[Mixins](styles/mixins.scss)-Datei mit nützlichen Helfern für deine Anwendung zur Verfügung.

### I18N

Einige Komponenten enthalten Textsymbole (Wörter und Phrasen). Diese sind in zwei Sprachen verfügbar: `en` (Standard) und `ru`. 
Konfiguriere die Sprache mit der `configure`-Funktion:

```js
// index.js

import {configure} from '@gravity-ui/uikit';

configure({
  lang: 'ru',
});
```

## Entwicklung

Starte den Entwicklungsserver mit Storybook durch folgenden Befehl:

```shell
npm start
```