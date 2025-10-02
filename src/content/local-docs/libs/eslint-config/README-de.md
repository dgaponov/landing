# @gravity-ui/eslint-config

## Installation

```
npm install --save-dev eslint @gravity-ui/eslint-config
```

## Verwendung

Erstellen Sie eine Datei `eslint.config.js` in Ihrem Projekt mit folgendem Inhalt:

```js
import baseConfig from '@gravity-ui/eslint-config';

export default [
  ...baseConfig,
  {
    // ...weitere Konfiguration
  },
];
```

Die Basis-Konfiguration umfasst auch TypeScript-Regeln.

### Prettier

Falls Sie Prettier verwenden, fügen Sie die entsprechende Konfiguration hinzu:

```js
import baseConfig from '@gravity-ui/eslint-config';
import prettierConfig from '@gravity-ui/eslint-config/prettier';

export default [
  ...baseConfig,
  ...prettierConfig,
  {
    // ...weitere Konfiguration
  },
];
```

### a11y

Falls Sie Barrierefreiheitsprobleme erkennen möchten, fügen Sie die entsprechende Konfiguration hinzu:

```js
import baseConfig from '@gravity-ui/eslint-config';
import a11yConfig from '@gravity-ui/eslint-config/a11y';

export default [
  ...baseConfig,
  ...a11yConfig,
  {
    // ...weitere Konfiguration
  },
];
```

### Import-Reihenfolge

Falls Sie eine Konvention für die Reihenfolge von Modul-Imports durchsetzen möchten, fügen Sie die entsprechende Konfiguration hinzu:

```js
import baseConfig from '@gravity-ui/eslint-config';
import importOrderConfig from '@gravity-ui/eslint-config/import-order';

export default [
  ...baseConfig,
  ...importOrderConfig,
  {
    // ...weitere Konfiguration
  },
];
```