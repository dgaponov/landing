# @gravity-ui/stylelint-config

Gravity UI 프로젝트를 위한 Stylelint 구성입니다.

## 요구 사항

- Node.js >= 20.x
- Stylelint 16.18.0
- PostCSS 8.x

## 설치

```
npm install --save-dev stylelint postcss @gravity-ui/stylelint-config
```

## 사용법

프로젝트 루트에 다음 내용을 포함한 `.stylelintrc` 파일을 추가하세요:

```json
{
  "extends": "@gravity-ui/stylelint-config"
}
```

### Prettier

Prettier를 사용 중이라면, 루트 구성에 추가 규칙을 확장하세요:

```json
{
  "extends": ["@gravity-ui/stylelint-config", "@gravity-ui/stylelint-config/prettier"]
}
```

### Order

CSS 파일의 속성을 정렬하고 싶다면, 루트 구성에 추가 규칙을 확장하세요:

```json
{
  "extends": ["@gravity-ui/stylelint-config", "@gravity-ui/stylelint-config/order"]
}
```