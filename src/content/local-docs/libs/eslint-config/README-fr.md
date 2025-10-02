# @gravity-ui/eslint-config

## Installation

```
npm install --save-dev eslint @gravity-ui/eslint-config
```

## Utilisation

Ajoutez un fichier `eslint.config.js` dans votre projet avec le contenu suivant :

```js
import baseConfig from '@gravity-ui/eslint-config';

export default [
  ...baseConfig,
  {
    // ...autres configurations
  },
];
```

La configuration de base inclut également les règles TypeScript.

### Prettier

Si vous utilisez Prettier, ajoutez la configuration correspondante :

```js
import baseConfig from '@gravity-ui/eslint-config';
import prettierConfig from '@gravity-ui/eslint-config/prettier';

export default [
  ...baseConfig,
  ...prettierConfig,
  {
    // ...autres configurations
  },
];
```

### a11y

Si vous souhaitez détecter les problèmes d'accessibilité, ajoutez la configuration correspondante :

```js
import baseConfig from '@gravity-ui/eslint-config';
import a11yConfig from '@gravity-ui/eslint-config/a11y';

export default [
  ...baseConfig,
  ...a11yConfig,
  {
    // ...autres configurations
  },
];
```

### Ordre

Si vous souhaitez imposer une convention pour l'ordre des imports de modules, ajoutez la configuration correspondante :

```js
import baseConfig from '@gravity-ui/eslint-config';
import importOrderConfig from '@gravity-ui/eslint-config/import-order';

export default [
  ...baseConfig,
  ...importOrderConfig,
  {
    // ...autres configurations
  },
];
```