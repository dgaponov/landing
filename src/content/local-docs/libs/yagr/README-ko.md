# Yagr <img src="https://raw.githubusercontent.com/gravity-ui/yagr/main/docs/assets/yagr.svg" width="24px" height="24px" />

Yagr은 [uPlot](https://github.com/leeoniya/uPlot)을 기반으로 한 고성능 HTML5 캔버스 차트 렌더러입니다. uPlot 차트에 대한 고급 기능을 제공합니다.

<img src="https://raw.githubusercontent.com/gravity-ui/yagr/main/docs/assets/demo.png" width="800" />

## 기능

-   [선, 영역, 열, 점을 시각화 유형으로 지원. 시리즈별로 구성 가능](https://yagr.tech/en/api/visualization)
-   [구성 가능한 범례 툴팁](https://yagr.tech/en/plugins/tooltip)
-   [소수점 정밀도를 위한 추가 옵션이 있는 축](https://yagr.tech/en/api/axes)
-   [범위 함수와 변환을 구성할 수 있는 스케일](https://yagr.tech/en/api/scales)
-   [플롯 선과 밴드. 구성 가능한 그리기 레이어](https://yagr.tech/en/plugins/plot-lines)
-   [반응형 차트](https://yagr.tech/en/api/settings#adaptivity) ([ResizeObserver](https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver) 필요)
-   [스택 영역/열에 대한 고급 지원](https://yagr.tech/en/api/scales#stacking)
-   [구성 가능한 마커](./docs/api/markers.md)
-   [라이트/다크 테마](https://yagr.tech/en/api/settings#theme)
-   [데이터 정규화](https://yagr.tech/en/api/scales#normalization)
-   [구성 가능한 크로스헤어, 커서 마커 및 스냅](https://yagr.tech/en/api/cursor)
-   Typescript
-   [지역화](https://yagr.tech/en/api/settings#localization)
-   [색상 이름에 CSS 변수 사용](https://yagr.tech/en/api/css)
-   [페이지네이션된 인라인 범례](https://yagr.tech/en/plugins/legend)
-   [오류 처리 및 확장된 훅](https://yagr.tech/en/api/lifecycle)
-   [누락된 데이터에 대한 데이터 정렬 및 보간](https://yagr.tech/en/api/data-processing)
-   [실시간 업데이트](https://yagr.tech/en/api/dynamic-updates)

## [문서](https://yagr.tech)

## 빠른 시작

```
npm i @gravity-ui/yagr
```

### NPM 모듈

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

### 스크립트 태그

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

### 예제

특정 기능이 필요하신가요? Yagr은 [demo/examples](./demo/examples/) 폴더에 유용한 예제를 제공합니다. 현재 버전으로 실행하는 방법:

1. 저장소를 클론합니다.
2. 종속성을 설치합니다: `npm i`.
3. `npm run build`를 실행합니다.
4. `npx http-server .`를 실행합니다.
5. http-server 출력에 따라 브라우저에서 예제를 엽니다.