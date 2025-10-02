# @gravity-ui/stylelint-config

Stylelint-Konfiguration für Gravity UI-Projekte.

## Voraussetzungen

- Node.js >= 20.x
- Stylelint 16.18.0
- PostCSS 8.x

## Installation

```
npm install --save-dev stylelint postcss @gravity-ui/stylelint-config
```

## Verwendung

Fügen Sie eine `.stylelintrc`-Datei im Projektstammverzeichnis mit folgendem Inhalt hinzu:

```json
{
  "extends": "@gravity-ui/stylelint-config"
}
```

### Prettier

Falls Sie Prettier verwenden, erweitern Sie die Root-Konfiguration um die zusätzlichen Regeln:

```json
{
  "extends": ["@gravity-ui/stylelint-config", "@gravity-ui/stylelint-config/prettier"]
}
```

### Order

Falls Sie Eigenschaften in Ihren CSS-Dateien sortieren möchten, erweitern Sie die Root-Konfiguration um die zusätzlichen Regeln:

```json
{
  "extends": ["@gravity-ui/stylelint-config", "@gravity-ui/stylelint-config/order"]
}
```