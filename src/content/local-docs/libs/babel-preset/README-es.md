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
        "env": {modules: false}, // predeterminado: {}
        "runtime": {useESModules: true}, // predeterminado: {}
        "typescript": true, // predeterminado: false
        "react": {runtime: "automatic"} // predeterminado: {}
      }
  ]
}
```