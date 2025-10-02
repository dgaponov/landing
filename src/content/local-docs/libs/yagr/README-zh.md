# Ẏagr <img src="https://raw.githubusercontent.com/gravity-ui/yagr/main/docs/assets/yagr.svg" width="24px" height="24px" />

Yagr 是一个基于 [uPlot](https://github.com/leeoniya/uPlot) 的高性能 HTML5 Canvas 图表渲染器。它为 uPlot 图表提供了高级功能。

<img src="https://raw.githubusercontent.com/gravity-ui/yagr/main/docs/assets/demo.png" width="800" />

## 功能特性

-   [线条、区域、柱状图和点作为可视化类型。可按系列配置](https://yagr.tech/en/api/visualization)
-   [可配置的图例提示](https://yagr.tech/en/plugins/tooltip)
-   [轴线支持小数级精度的额外选项](https://yagr.tech/en/api/axes)
-   [可配置范围函数和变换的刻度](https://yagr.tech/en/api/scales)
-   [绘图线和带状区域。可配置绘制层](https://yagr.tech/en/plugins/plot-lines)
-   [响应式图表](https://yagr.tech/en/api/settings#adaptivity)（需要 [ResizeObserver](https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver)）
-   [对堆叠区域/柱状图的高级支持](https://yagr.tech/en/api/scales#stacking)
-   [可配置标记](./docs/api/markers.md)
-   [浅色/深色主题](https://yagr.tech/en/api/settings#theme)
-   [数据归一化](https://yagr.tech/en/api/scales#normalization)
-   [可配置十字准线、光标标记和吸附](https://yagr.tech/en/api/cursor)
-   Typescript
-   [本地化](https://yagr.tech/en/api/settings#localization)
-   [颜色名称中的 CSS 变量](https://yagr.tech/en/api/css)
-   [分页内联图例](https://yagr.tech/en/plugins/legend)
-   [错误处理和扩展钩子](https://yagr.tech/en/api/lifecycle)
-   [数据对齐和缺失数据的插值](https://yagr.tech/en/api/data-processing)
-   [实时更新](https://yagr.tech/en/api/dynamic-updates)

## [文档](https://yagr.tech)

## 快速开始

```
npm i @gravity-ui/yagr
```

### NPM 模块

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

### Script 标签

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

### 示例

需要特定功能？Yagr 在 [demo/examples](./demo/examples/) 文件夹中提供了一些有用的示例。如何使用当前版本启动它们：

1. 克隆仓库。
2. 安装依赖 `npm i`。
3. 运行 `npm run build`。
4. 运行 `npx http-server .`。
5. 根据 http-server 输出在浏览器中打开示例。