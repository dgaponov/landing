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
        "env": {modules: false}, // Standardmäßig {}
        "runtime": {useESModules: true}, // Standardmäßig {}
        "typescript": true, // Standardmäßig false
        "react": {runtime: "automatic"} // Standardmäßig {}
      }
  ]
}
```