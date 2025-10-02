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
        "env": {modules: false}, // valor predeterminado {}
        "runtime": {useESModules: true}, // valor predeterminado {}
        "typescript": true, // valor predeterminado false
        "react": {runtime: "automatic"} // valor predeterminado {}
      }
  ]
}
```