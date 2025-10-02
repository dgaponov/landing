# Ẏagr <img src="https://raw.githubusercontent.com/gravity-ui/yagr/main/docs/assets/yagr.svg" width="24px" height="24px" />

Yagr — это высокопроизводительный рендерер графиков на HTML5 canvas, основанный на [uPlot](https://github.com/leeoniya/uPlot). Он предоставляет высокоуровневые возможности для графиков uPlot.

<img src="https://raw.githubusercontent.com/gravity-ui/yagr/main/docs/assets/demo.png" width="800" />

## Возможности

-   [Линии, области, столбцы и точки как типы визуализации. Настраивается для каждой серии](https://yagr.tech/en/api/visualization)
-   [Настраиваемая подсказка легенды](https://yagr.tech/en/plugins/tooltip)
-   [Оси с дополнительными опциями для точности на уровне десятичных знаков](https://yagr.tech/en/api/axes)
-   [Шкалы с настраиваемыми функциями диапазона и преобразованиями](https://yagr.tech/en/api/scales)
-   [Линии и полосы графика. Настраиваемый слой отрисовки](https://yagr.tech/en/plugins/plot-lines)
-   [Адаптивные графики](https://yagr.tech/en/api/settings#adaptivity) (требуется [ResizeObserver](https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver))
-   [Высокоуровневая поддержка стековых областей/столбцов](https://yagr.tech/en/api/scales#stacking)
-   [Настраиваемые маркеры](./docs/api/markers.md)
-   [Светлая/тёмная тема](https://yagr.tech/en/api/settings#theme)
-   [Нормализация данных](https://yagr.tech/en/api/scales#normalization)
-   [Настраиваемые прицелы, маркеры курсора и привязка](https://yagr.tech/en/api/cursor)
-   Typescript
-   [Локализация](https://yagr.tech/en/api/settings#localization)
-   [CSS-переменные в именах цветов](https://yagr.tech/en/api/css)
-   [Пагинированная встроенная легенда](https://yagr.tech/en/plugins/legend)
-   [Обработка ошибок и расширенные хуки](https://yagr.tech/en/api/lifecycle)
-   [Выравнивание данных и интерполяция для пропущенных значений](https://yagr.tech/en/api/data-processing)
-   [Обновления в реальном времени](https://yagr.tech/en/api/dynamic-updates)

## [Документация](https://yagr.tech)

## Быстрый старт

```
npm i @gravity-ui/yagr
```

### NPM-модуль

```typescript
import Yagr from '@gravity-ui/yagr';

new Yagr(document.body, {
    timeline: [1, 2, 3, 4, 5],
    series: [
        {
            data: [1, 2, 3, 4, 5],
            color: 'red',
        },
        {
            data: [2, 3, 1, 4, 5],
            color: 'green',
        },
    ],
});
```

### Тег Script

```html
<script src="https://unpkg.com/@gravity-ui/yagr/dist/yagr.iife.min.js"></script>
<script>
    new Yagr(document.body, {
        timeline: [1, 2, 3, 4, 5],
        series: [
            {
                data: [1, 2, 3, 4, 5],
                color: 'red',
            },
            {
                data: [2, 3, 1, 4, 5],
                color: 'green',
            },
        ],
    });
</script>
```

### Примеры

Нужен конкретный пример? Yagr предлагает несколько полезных примеров в папке [demo/examples](./demo/examples/). Как запустить их с текущей версией:

1. Клонируйте репозиторий.
2. Установите зависимости: `npm i`.
3. Выполните `npm run build`.
4. Запустите `npx http-server .`.
5. Откройте примеры в браузере, следуя выводу http-server.