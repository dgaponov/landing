# @gravity-ui/stylelint-config

Конфигурация Stylelint для проектов Gravity UI.

## Требования

- Node.js >= 20.x
- Stylelint 16.18.0
- PostCSS 8.x

## Установка

```
npm install --save-dev stylelint postcss @gravity-ui/stylelint-config
```

## Использование

Добавьте файл `.stylelintrc` в корень проекта со следующим содержимым:

```json
{
  "extends": "@gravity-ui/stylelint-config"
}
```

### Prettier

Если вы используете Prettier, расширьте базовую конфигурацию дополнительными правилами:

```json
{
  "extends": ["@gravity-ui/stylelint-config", "@gravity-ui/stylelint-config/prettier"]
}
```

### Order

Если вы хотите упорядочивать свойства в ваших CSS-файлах, расширьте базовую конфигурацию дополнительными правилами:

```json
{
  "extends": ["@gravity-ui/stylelint-config", "@gravity-ui/stylelint-config/order"]
}
```