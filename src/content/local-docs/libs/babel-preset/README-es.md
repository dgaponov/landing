# @gravity-ui/babel-preset

Preset de Babel para proyectos de Gravity UI

## Instalación
```
npm install --save-dev @gravity-ui/babel-preset
```

## Uso

### Vía `.babelrc`

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