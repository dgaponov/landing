# @gravity-ui/stylelint-config

Configuration Stylelint pour les projets Gravity UI.

## Exigences

- Node.js >= 20.x
- Stylelint 16.18.0
- PostCSS 8.x

## Installation

```
npm install --save-dev stylelint postcss @gravity-ui/stylelint-config
```

## Utilisation

Ajoutez un fichier `.stylelintrc` à la racine de votre projet avec le contenu suivant :

```json
{
  "extends": "@gravity-ui/stylelint-config"
}
```

### Prettier

Si vous utilisez Prettier, étendez la configuration racine avec les règles supplémentaires suivantes :

```json
{
  "extends": ["@gravity-ui/stylelint-config", "@gravity-ui/stylelint-config/prettier"]
}
```

### Order

Si vous souhaitez ordonner les propriétés dans vos fichiers CSS, étendez la configuration racine avec les règles supplémentaires suivantes :

```json
{
  "extends": ["@gravity-ui/stylelint-config", "@gravity-ui/stylelint-config/order"]
}
```