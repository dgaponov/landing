# @gravity-ui/babel-preset

Babel-Preset für Gravity UI-Projekte

## Installation
```
npm install --save-dev @gravity-ui/babel-preset
```

## Verwendung

### Über `.babelrc`

```json5
{
  "presets": [
      "@gravity-ui/babel-preset",
      {
        "env": {modules: false}, // Standard: {}
        "runtime": {useESModules: true}, // Standard: {}
        "typescript": true, // Standard: false
        "react": {runtime: "automatic"} // Standard: {}
      }
  ]
}
```