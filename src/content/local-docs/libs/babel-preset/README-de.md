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
        "env": {modules: false}, // defaults to {}
        "runtime": {useESModules: true}, // defaults to {}
        "typescript": true, // defaults to false
        "react": {runtime: "automatic"} // defaults to {}
      }
  ]
}
```