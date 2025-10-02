# @gravity-ui/babel-preset

Gravity UI 项目的 Babel 预设

## 安装
```
npm install --save-dev @gravity-ui/babel-preset
```

## 使用

### 通过 `.babelrc`

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