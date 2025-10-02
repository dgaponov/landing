# @gravity-ui/babel-preset

Preset Babel pour les projets Gravity UI

## Installation
```
npm install --save-dev @gravity-ui/babel-preset
```

## Utilisation

### Via `.babelrc`

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