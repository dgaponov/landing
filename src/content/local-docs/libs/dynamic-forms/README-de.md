# @gravity-ui/dynamic-forms · [![npm package](https://img.shields.io/npm/v/@gravity-ui/dynamic-forms)](https://www.npmjs.com/package/@gravity-ui/dynamic-forms) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/dynamic-forms/.github/workflows/ci.yml?label=CI&logo=github)](https://github.com/gravity-ui/dynamic-forms/actions/workflows/ci.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/dynamic-forms/)

Die auf JSON Schema basierende Bibliothek zum Rendern von Formularen und Formularwerten.

## Installation

```shell
npm install --save-dev @gravity-ui/dynamic-forms
```

## Verwendung

```jsx
import {DynamicField, Spec, dynamicConfig} from '@gravity-ui/dynamic-forms';

// Zum Einbetten in ein final-form
<DynamicField name={name} spec={spec} config={config} />;

import {DynamicView, dynamicViewConfig} from '@gravity-ui/dynamic-forms';

// Zum Überblick über die Werte
<DynamicView value={value} spec={spec} config={dynamicViewConfig} />;
```

### I18N

Bestimmte Komponenten enthalten Text-Token (Wörter und Phrasen), die in zwei Sprachen verfügbar sind: `en` (Standard) und `ru`. Um die Sprache zu setzen, verwenden Sie die `configure`-Funktion:

```js
// index.js

import {configure, Lang} from '@gravity-ui/dynamic-forms';

configure({lang: Lang.Ru});
```

## Entwicklung

Um den Entwicklungsserver mit Storybook zu starten, führen Sie den folgenden Befehl aus:

```shell
npm ci
npm run dev
```