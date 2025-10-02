# @gravity-ui/page-constructor · [![npm package](https://img.shields.io/npm/v/@gravity-ui/page-constructor)](https://www.npmjs.com/package/@gravity-ui/page-constructor) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/page-constructor/ci.yml?branch=main&label=CI)](https://github.com/gravity-ui/page-constructor/actions/workflows/ci.yml?query=branch:main) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/page-constructor/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/page-constructor/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/page-constructor/)

## 페이지 생성기

`Page-constructor`는 `JSON` 데이터를 기반으로 웹 페이지나 그 일부를 렌더링하는 라이브러리입니다 (`YAML` 형식 지원은 나중에 추가될 예정입니다).

페이지 생성 시 컴포넌트 기반 접근 방식을 사용합니다: 페이지는 원하는 순서로 배치할 수 있는 미리 만들어진 블록 세트로 구성됩니다. 각 블록은 특정 유형과 입력 데이터 매개변수 세트를 가집니다.

입력 데이터 형식과 사용 가능한 블록 목록은 [문서](https://preview.gravity-ui.com/page-constructor/?path=/docs/documentation-blocks--docs)를 참조하세요.

## 설치

```shell
npm install @gravity-ui/page-constructor
```

## 빠른 시작

먼저 React 프로젝트와 서버가 필요합니다. 예를 들어 Vite를 사용해 React 프로젝트를 만들고 Express 서버를 설정하거나, Next.js 애플리케이션을 생성할 수 있습니다. Next.js는 클라이언트와 서버 측을 모두 제공합니다.

필요한 종속성을 설치하세요:

```shell
npm install @gravity-ui/page-constructor @diplodoc/transform @gravity-ui/uikit
```

페이지를 `Page Constructor`로 삽입하세요. 올바르게 작동하려면 `PageConstructorProvider`로 감싸야 합니다:

```tsx
import {PageConstructor, PageConstructorProvider} from '@gravity-ui/page-constructor';
import '@gravity-ui/page-constructor/styles/styles.scss';

const App = () => {
  const content = {
    blocks: [
      {
        type: 'header-block',
        title: 'Hello world',
        background: {color: '#f0f0f0'},
        description:
          '**Congratulations!** Have you built a [page-constructor](https://github.com/gravity-ui/page-constructor) into your website',
      },
    ],
  };

  return (
    <PageConstructorProvider>
      <PageConstructor content={content} />
    </PageConstructorProvider>
  );
};

export default App;
```

이것은 가장 간단한 연결 예제입니다. YFM 마크업이 작동하려면 서버에서 콘텐츠를 처리하고 클라이언트에서 이를 받아야 합니다.

서버가 별도의 애플리케이션인 경우, page-constructor를 설치하세요:

```shell
npm install @gravity-ui/page-constructor
```

기본 블록의 모든 YFM을 처리하려면 `contentTransformer`를 호출하고 콘텐츠와 옵션을 전달하세요:

```ts
const express = require('express');
const app = express();
const {contentTransformer} = require('@gravity-ui/page-constructor/server');

const content = {
  blocks: [
    {
      type: 'header-block',
      title: 'Hello world',
      background: {color: '#f0f0f0'},
      description:
        '**Congratulations!** Have you built a [page-constructor](https://github.com/gravity-ui/page-constructor) into your website',
    },
  ],
};

app.get('/content', (req, res) => {
  res.send({content: contentTransformer({content, options: {lang: 'en'}})});
});

app.listen(3000);
```

클라이언트에서 엔드포인트 호출을 추가해 콘텐츠를 받으세요:

```tsx
import {PageConstructor, PageConstructorProvider} from '@gravity-ui/page-constructor';
import '@gravity-ui/page-constructor/styles/styles.scss';
import {useEffect, useState} from 'react';

const App = () => {
  const [content, setContent] = useState();

  useEffect(() => {
    (async () => {
      const response = await fetch('http://localhost:3000/content').then((r) => r.json());
      setContent(response.content);
    })();
  }, []);

  return (
    <PageConstructorProvider>
      <PageConstructor content={content} />
    </PageConstructorProvider>
  );
};

export default App;
```

### 미리 만들어진 템플릿

새 프로젝트를 시작하려면 우리가 준비한 [Next.js 기반 미리 만들어진 템플릿](https://github.com/gravity-ui/page-constructor-website-template)을 사용할 수 있습니다.

### 정적 사이트 빌더

[Page Constructor Builder](https://github.com/gravity-ui/page-constructor-builder)는 @gravity-ui/page-constructor를 사용해 YAML 구성으로부터 정적 페이지를 빌드하는 명령줄 유틸리티입니다.

## 문서

### 매개변수

```typescript
interface PageConstructorProps {
  content: PageContent; // JSON 형식의 블록 데이터.
  shouldRenderBlock?: ShouldRenderBlock; // 각 블록을 렌더링할 때 호출되는 함수로, 표시 조건을 설정할 수 있습니다.
  custom?: Custom; // 사용자 지정 블록 ( `Customization` 참조).
  renderMenu?: () => React.ReactNode; // 페이지 메뉴와 네비게이션을 렌더링하는 함수 (기본 메뉴 버전 렌더링을 추가할 계획입니다).
  navigation?: NavigationData; // JSON 형식의 네비게이션 컴포넌트 사용을 위한 네비게이션 데이터
  isBranded?: boolean; // true인 경우 https://gravity-ui.com/로 연결되는 푸터를 추가합니다. 더 많은 사용자 지정을 위해 BrandFooter 컴포넌트를 사용해 보세요.
}

interface PageConstructorProviderProps {
  isMobile?: boolean; // 코드가 모바일 모드에서 실행되는지 나타내는 플래그.
  locale?: LocaleContextProps; // 언어와 도메인 정보 (링크 생성 및 포맷팅 시 사용).
  location?: Location; // 브라우저 또는 라우터 히스토리 API, 페이지 URL.
  analytics?: AnalyticsContextProps; // 분석 이벤트 처리 함수

  ssrConfig?: SSR; // 코드가 서버 측에서 실행되는지 나타내는 플래그.
  theme?: 'light' | 'dark'; // 페이지 렌더링에 사용할 테마.
  mapsContext?: MapsContextType; // 지도 매개변수: apikey, type, scriptSrc, nonce
}

export interface PageContent extends Animatable {
  blocks: Block[];
  menu?: Menu;
  background?: MediaProps;
}

interface Custom {
  blocks?: CustomItems;
  subBlocks?: CustomItems;
  headers?: CustomItems;
  loadable?: LoadableConfig;
}

type ShouldRenderBlock = (block: Block, blockKey: string) => Boolean;

interface Location {
  history?: History;
  search?: string;
  hash?: string;
  pathname?: string;
  hostname?: string;
}

interface Locale {
  lang?: Lang;
  tld?: string;
}

interface SSR {
  isServer?: boolean;
}

interface NavigationData {
  logo: NavigationLogo;
  header: HeaderData;
}

interface NavigationLogo {
  icon: ImageProps;
  text?: string;
  url?: string;
}

interface HeaderData {
  leftItems: NavigationItem[];
  rightItems?: NavigationItem[];
}
```

```typescript
interface NavigationLogo {
  icon: ImageProps;
  text?: string;
  url?: string;
}
```

### 서버 유틸리티

이 패키지는 콘텐츠를 변환하기 위한 서버 유틸리티 세트를 제공합니다.

```ts
const {fullTransform} = require('@gravity-ui/page-constructor/server');

const {html} = fullTransform(content, {
  lang,
  extractTitle: true,
  allowHTML: true,
  path: __dirname,
  plugins,
});
```

내부적으로는 Yandex Flavored Markdown을 HTML로 변환하는 `diplodoc/transform` 패키지를 사용하므로, 이는 peer dependencies에 포함되어 있습니다.

필요한 곳에서 유용한 유틸리티를 사용할 수도 있습니다. 예를 들어, 커스텀 컴포넌트에서 사용할 수 있습니다.

```ts
const {
  typografToText,
  typografToHTML,
  yfmTransformer,
} = require('@gravity-ui/page-constructor/server');

const post = {
  title: typografToText(title, lang),
  content: typografToHTML(content, lang),
  description: yfmTransformer(lang, description, {plugins}),
};
```

더 많은 유틸리티는 [이 섹션](https://github.com/gravity-ui/page-constructor/tree/main/src/text-transform)에서 확인할 수 있습니다.

### 서버 유틸리티 및 변환기 상세 문서

서버 유틸리티 사용에 대한 포괄적인 가이드, 상세 설명 및 고급 사용 사례를 보려면 [서버 유틸리티 사용에 대한 추가 챕터](./docs/data-preparation.md)를 방문하세요.

### 커스텀 블록

페이지 생성기는 앱에서 사용자 정의한 블록을 사용할 수 있게 합니다. 블록은 일반적인 React 컴포넌트입니다.

커스텀 블록을 생성기에 전달하려면:

1. 앱에서 블록을 생성합니다.

2. 코드에서 블록 유형(문자열)을 키로 하고, 가져온 블록 컴포넌트를 값으로 하는 객체를 생성합니다.

3. 생성한 객체를 `PageConstructor` 컴포넌트의 `custom.blocks`, `custom.headers` 또는 `custom.subBlocks` 매개변수에 전달합니다(`custom.headers`는 일반 콘텐츠 위에 별도로 렌더링되는 블록 헤더를 지정합니다).

4. 이제 입력 데이터(`content` 매개변수)에서 생성한 블록의 유형과 데이터를 지정하여 사용할 수 있습니다.

커스텀 블록을 생성할 때 믹스인과 생성기 스타일 변수를 사용하려면 파일에 다음 import를 추가하세요:

```css
@import '~@gravity-ui/page-constructor/styles/styles.scss';
```

기본 폰트를 사용하려면 파일에 다음 import를 추가하세요:

```css
@import '~@gravity-ui/page-constructor/styles/fonts.scss';
```

### 로드 가능한 블록

때때로 블록이 로드된 데이터에 기반하여 스스로를 렌더링해야 하는 경우가 있습니다. 이 경우 로드 가능한 블록을 사용합니다.

커스텀 `loadable` 블록을 추가하려면 `PageConstructor`에 `custom.loadable` 속성을 전달하세요. 여기서 키는 컴포넌트의 데이터 소스 이름(문자열)이고, 값은 객체입니다.

```typescript
export interface LoadableConfigItem {
  fetch: FetchLoadableData; // 데이터 로딩 메서드
  component: React.ComponentType; // 로드된 데이터를 전달할 컴포넌트
}

type FetchLoadableData<TData = any> = (blockKey: string) => Promise<TData>;
```

### 그리드

페이지 생성기는 `bootstrap` 그리드를 사용하며, React 컴포넌트 기반으로 구현되어 있어 프로젝트에서 생성기와 별도로 사용할 수 있습니다.

사용 예시:

```jsx
import {Grid, Row, Col} from '@gravity-ui/page-constructor';

const Page = ({children}: PropsWithChildren<PageProps>) => (
  <Grid>
    <Row>
      <Col sizes={{lg: 4, sm: 6, all: 12}}>{children}</Col>
    </Row>
  </Grid>
);
```

### 네비게이션

페이지 네비게이션도 생성기와 별도로 사용할 수 있습니다:

```jsx
import {Navigation} from '@gravity-ui/page-constructor';

const Page= ({data, logo}: React.PropsWithChildren<PageProps>) => <Navigation data={data} logo={logo} />;
```

### 블록

각 블록은 원자적인 최상위 컴포넌트입니다. 이들은 `src/units/constructor/blocks` 디렉토리에 저장되어 있습니다.

### 서브 블록

서브 블록은 블록의 `children` 속성에서 사용할 수 있는 컴포넌트입니다. 설정에서 서브 블록의 자식 컴포넌트 목록이 지정됩니다. 렌더링되면 이 서브 블록은 블록에 `children`으로 전달됩니다.

### `page-constructor`에 새 블록 추가 방법

1. `src/blocks` 또는 `src/sub-blocks` 디렉토리에 블록 또는 서브 블록 코드를 담은 폴더를 생성합니다.

2. 블록 또는 서브 블록 이름을 enum `BlockType` 또는 `SubBlockType`에 추가하고, `src/models/constructor-items/blocks.ts` 또는 `src/models/constructor-items/sub-blocks.ts` 파일에서 기존 항목과 유사하게 속성을 설명합니다.

3. `src/blocks/index.ts` 파일에 블록을, `src/sub-blocks/index.ts` 파일에 서브 블록을 export합니다.

4. `src/constructor-items.ts`의 매핑에 새 컴포넌트 또는 블록을 추가합니다.

5. 새 블록에 대한 검증기를 추가합니다:

   - 블록 또는 서브 블록 디렉토리에 `schema.ts` 파일을 추가합니다. 이 파일에서 컴포넌트의 매개변수 검증기를 [`json-schema`](http://json-schema.org/) 형식으로 설명합니다.
   - `schema/validators/blocks.ts` 또는 `schema/validators/sub-blocks.ts` 파일에 이를 export합니다.
   - `schema/index.ts` 파일의 `enum` 또는 `selectCases`에 추가합니다.

6. 블록 디렉토리에 입력 매개변수 설명이 포함된 `README.md` 파일을 추가합니다.
7. 블록 디렉토리에 `__stories__` 폴더를 추가하여 storybook 데모를 만듭니다. 스토리의 모든 데모 콘텐츠는 스토리 디렉토리의 `data.json`에 배치해야 합니다. 일반 `Story`는 블록 props의 유형을 받아야 하며, 그렇지 않으면 Storybook에서 잘못된 블록 props가 표시됩니다.
8. 블록 데이터 템플릿을 `src/editor/data/templates/` 폴더에 추가합니다. 파일 이름은 블록 유형과 일치해야 합니다.
9. (선택) 블록 미리보기 아이콘을 `src/editor/data/previews/` 폴더에 추가합니다. 파일 이름은 블록 유형과 일치해야 합니다.

### 테마

`PageConstructor`는 테마를 사용할 수 있게 합니다. 앱에서 선택된 테마에 따라 개별 블록 속성에 다른 값을 설정할 수 있습니다.

블록 속성에 테마를 추가하려면:

1. `models/blocks.ts` 파일에서 해당 블록 속성의 유형을 `ThemeSupporting<T>` 제네릭을 사용하여 정의합니다. 여기서 `T`는 속성의 유형입니다.

2. 블록의 `react` 컴포넌트 파일에서 `getThemedValue`와 `useTheme` 훅을 통해 테마가 적용된 속성 값을 가져옵니다(`MediaBlock.tsx` 블록의 예시를 참조하세요).

3. 속성 검증기에 테마 지원을 추가합니다: 블록의 `schema.ts` 파일에서 이 속성을 `withTheme`으로 감쌉니다.

### i18n

`page-constructor`는 `uikit-based` 라이브러리이며, uikit의 `i18n` 인스턴스를 사용합니다. 국제화를 설정하려면 uikit의 `configure`를 사용하기만 하면 됩니다:

```typescript
import {configure} from '@gravity-ui/uikit';

configure({
  lang: 'ru',
});
```

### 지도

지도 사용을 위해 `PageConstructorProvider`의 `mapContext` 필드에 지도 유형, scriptSrc 및 apiKey를 넣으세요.

프로젝트 루트의 .env.development 파일에서 개발 모드용 환경 변수를 정의할 수 있습니다.
`STORYBOOK_GMAP_API_KEY` - Google Maps용 apiKey

### 분석

#### 초기화

분석을 사용하려면 생성기에 핸들러를 전달하세요. 핸들러는 프로젝트 측에서 생성해야 합니다. 핸들러는 `default`와 `custom` 이벤트 객체를 받습니다. 전달된 핸들러는 버튼, 링크, 네비게이션 및 컨트롤 클릭 시 호출됩니다. 모든 이벤트 처리를 위해 하나의 핸들러를 사용하므로, 핸들러를 생성할 때 다양한 이벤트를 처리하는 방법을 주의 깊게 고려하세요. 복잡한 로직을 구축하는 데 도움이 되는 미리 정의된 필드가 있습니다.

생성기에 `autoEvents: true`를 전달하면 자동으로 구성된 이벤트를 호출할 수 있습니다.

```ts
function sendEvents(events: MyEventType []) {
  ...
}

<PageConstructorProvider
    ...

    analytics={{sendEvents, autoEvents: true}}

    ...
/>
```

이벤트 객체는 하나의 필수 필드인 `name`만 가지고 있습니다. 또한 복잡한 로직을 관리하는 데 도움이 되는 미리 정의된 필드도 있습니다. 예를 들어, 프로젝트에서 여러 분석 시스템을 사용하는 경우 `counter.include`를 사용하여 특정 카운터로 이벤트를 보낼 수 있습니다.

```ts
type AnalyticsEvent<T = {}> = T & {
  name: string;
  type?: string;
  counters?: AnalyticsCounters;
  context?: string;
};
```

프로젝트에 필요한 이벤트 유형을 구성할 수 있습니다.

```ts
type MyEventType = AnalyticsEvent<{
  [key: string]?: string; // only a 'string' type is supported
}>;
```

#### Counter selector

이벤트에 어떤 분석 시스템으로 보낼지 구성할 수 있습니다.

```ts
type AnalyticsCounters = {
  include?: string[]; // array of analytics counter ids that will be applied
  exclude?: string[]; // array of analytics counter ids that will not be applied
};
```

#### context parameter

이벤트가 발생한 프로젝트 내 위치를 정의하기 위해 `context` 값을 전달합니다.

아래 선택기를 사용하거나 프로젝트 요구사항에 맞는 로직을 생성합니다.

```ts
// analyticsHandler.ts
if (isCounterAllowed(counterName, counters)) {
  analyticsCounter.reachGoal(counterName, name, parameters);
}
```

#### Reserved event types

자동 구성된 이벤트를 표시하기 위해 여러 미리 정의된 이벤트 유형이 사용됩니다. 예를 들어, 기본 이벤트를 필터링하는 데 이러한 유형을 사용할 수 있습니다.

```ts
enum PredefinedEventTypes {
  Default = 'default-event', // default events which fire on every button click
  Play = 'play', // React player event
  Stop = 'stop', // React player event
}
```

## Development

```bash
npm ci
npm run dev
```

#### Note about Vite

```ts
import react from '@vitejs/plugin-react-swc';
import dynamicImport from 'vite-plugin-dynamic-import';

export default defineConfig({
  plugins: [
    react(),
    dynamicImport({
      filter: (id) => id.includes('/node_modules/@gravity-ui/page-constructor'),
    }),
  ],
});
```

Vite의 경우 `vite-plugin-dynamic-import` 플러그인을 설치하고 동적 임포트가 작동하도록 설정을 구성해야 합니다.

## Release flow

일반적인 경우 우리는 다음과 같은 유형의 커밋을 사용합니다:

1. fix: fix 유형의 커밋은 코드베이스에서 버그를 수정합니다 (이는 Semantic Versioning의 PATCH와 상응합니다).
2. feat: feat 유형의 커밋은 코드베이스에 새로운 기능을 도입합니다 (이는 Semantic Versioning의 MINOR와 상응합니다).
3. BREAKING CHANGE: BREAKING CHANGE: 푸터가 있거나 유형/스코프 뒤에 !를 붙인 커밋은 중단되는 API 변경을 도입합니다 (이는 Semantic Versioning의 MAJOR와 상응합니다). BREAKING CHANGE는 모든 유형의 커밋에 포함될 수 있습니다.
4. 릴리스 패키지 버전을 수동으로 설정하려면 커밋 메시지에 `Release-As: <version>`을 추가합니다. 예:

```bash
git commit -m 'chore: bump release

Release-As: 1.2.3'
```

모든 정보는 [여기](https://www.conventionalcommits.org/en/v1.0.0/)에서 확인할 수 있습니다.

코드 소유자로부터 풀 리퀘스트 승인을 받고 모든 검사를 통과한 후, 다음 단계를 따르세요:

1. 다른 기여자의 변경 사항이 포함된 로봇의 릴리스 풀 리퀘스트가 있는지 확인하세요 (예: `chore(main): release 0.0.0`). 존재한다면 병합되지 않은 이유를 확인하세요. 기여자가 공유 버전 릴리스를 동의하면 다음 단계를 따르세요. 동의하지 않으면 해당 기여자에게 자신의 버전을 릴리스하도록 요청한 후 다음 단계를 따르세요.
2. PR을 스쿼시하고 병합하세요 (Github-Actions로 새 버전을 릴리스하는 것이 중요합니다).
3. 로봇이 패키지의 새 버전과 CHANGELOG.md에 대한 변경 사항 정보를 포함한 PR을 생성할 때까지 기다리세요. 프로세스는 [Actions 탭](https://github.com/gravity-ui/page-constructor/actions)에서 확인할 수 있습니다.
4. CHANGELOG.md에서 변경 사항을 확인하고 로봇의 PR을 승인하세요.
5. PR을 스쿼시하고 병합하세요. 릴리스 프로세스는 [Actions 탭](https://github.com/gravity-ui/page-constructor/actions)에서 확인할 수 있습니다.

### Alpha versions release

브랜치에서 패키지의 알파 버전을 릴리스하려면 수동으로 수행할 수 있습니다:

1. Actions 탭으로 이동하세요.
2. 왼쪽 페이지에서 "Release alpha version" 워크플로를 선택하세요.
3. 오른쪽에서 "Run workflow" 버튼을 볼 수 있습니다. 여기서 브랜치를 선택할 수 있습니다.
4. 수동 버전 필드를 볼 수 있습니다. 브랜치에서 처음 알파를 릴리스하는 경우 여기에 아무것도 설정하지 마세요. 첫 번째 릴리스 후 브랜치가 곧 만료될 수 있으므로 package.json을 변경하지 않기 때문에 새 버전을 수동으로 설정해야 합니다. 수동 버전에는 `alpha` 접두어를 사용하세요. 그렇지 않으면 오류가 발생합니다.
5. "Run workflow"를 클릭하고 액션이 완료될 때까지 기다리세요. 필요에 따라 여러 버전을 릴리스할 수 있지만 남용하지 말고 정말 필요할 때만 릴리스하세요. 다른 경우에는 [npm pack](https://docs.npmjs.com/cli/v7/commands/npm-pack)을 사용하세요.

### Beta-major versions release

새 주요 버전을 릴리스하려면 안정 버전 전에 베타 버전이 필요할 가능성이 큽니다. 다음을 수행하세요:

1. `beta` 브랜치를 생성하거나 업데이트하세요.
2. 변경 사항을 추가하세요.
3. 새 베타 버전에 준비되면 빈 커밋으로 수동 릴리스를 수행하세요 (또는 마지막 커밋에 푸터가 포함된 이 커밋 메시지를 추가할 수 있습니다):

```bash
git commit -m 'fix: last commit

Release-As: 3.0.0-beta.0' --allow-empty
```

4. 로봇이 업데이트된 CHANGELOG.md와 패키지 버전 증가를 포함한 `beta` 브랜치로 새 PR을 생성할 것입니다.
5. 원하는 만큼 반복할 수 있습니다. 베타 태그 없이 최종 주요 버전을 릴리스할 준비가 되면 `beta` 브랜치에서 `main` 브랜치로 PR을 생성하세요. 패키지 버전에 베타 태그가 포함되는 것은 정상입니다. 로봇이 이를 알고 적절히 변경합니다. `3.0.0-beta.0`은 `3.0.0`이 됩니다.

### Release flow for previous major-versions

main에 커밋한 후 이전 주요 버전에서 새 버전을 릴리스하려면 다음을 수행하세요:

1. 필요한 브랜치를 업데이트하세요. 이전 주요 릴리스 브랜치 이름은 다음과 같습니다:
   1. `version-1.x.x/fixes` - 1.x.x 주요 버전에 대한
   2. `version-2.x.x` - 2.x.x 주요 버전에 대한
2. 이전 주요 릴리스 브랜치에서 새 브랜치를 체크아웃하세요.
3. `main` 브랜치에서 커밋을 체리픽하세요.
4. PR을 생성하고 승인을 받은 후 이전 주요 릴리스 브랜치로 병합하세요.
5. PR을 스쿼시하고 병합하세요 (Github-Actions로 새 버전을 릴리스하는 것이 중요합니다).
6. 로봇이 패키지의 새 버전과 CHANGELOG.md에 대한 변경 사항 정보를 포함한 PR을 생성할 때까지 기다리세요. 프로세스는 [Actions 탭](https://github.com/gravity-ui/page-constructor/actions)에서 확인할 수 있습니다.
7. CHANGELOG.md에서 변경 사항을 확인하고 로봇의 PR을 승인하세요.
8. PR을 스쿼시하고 병합하세요. 릴리스 프로세스는 [Actions 탭](https://github.com/gravity-ui/page-constructor/actions)에서 확인할 수 있습니다.

## Page constructor editor

에디터는 실시간 미리보기를 제공하는 페이지 콘텐츠 관리 사용자 인터페이스를 제공합니다.

사용 방법:

```tsx
import {Editor} from '@gravity-ui/page-constructor/editor';

interface MyAppEditorProps {
  initialContent: PageContent;
  transformContent: ContentTransformer;
  onChange: (content: PageContent) => void;
}

export const MyAppEditor = ({initialContent, onChange, transformContent}: MyAppEditorProps) => (
  <Editor content={initialContent} onChange={onChange} transformContent={transformContent} />
);
```

## Memory Bank

이 프로젝트에는 포괄적인 **Memory Bank**가 포함되어 있습니다. 이는 프로젝트의 아키텍처, 구성 요소, 사용 패턴에 대한 상세한 정보를 제공하는 Markdown 문서 파일 모음입니다. Memory Bank는 AI 에이전트와 작업할 때 특히 유용하며, 다음에 대한 구조화된 정보를 포함하고 있습니다:

- **Project Overview**: 핵심 요구사항, 목표 및 맥락
- **Component Documentation**: 모든 구성 요소에 대한 상세한 사용 가이드
- **System Architecture**: 기술 패턴 및 설계 결정
- **Development Progress**: 현재 상태 및 구현 세부 사항

### Memory Bank 사용 방법

Memory Bank는 `memory-bank/` 디렉터리에 위치하며, 다른 문서와 마찬가지로 읽을 수 있는 일반 Markdown 파일로 구성되어 있습니다:

- `projectbrief.md` - 핵심 요구사항이 포함된 기초 문서
- `productContext.md` - 프로젝트 목적 및 사용자 경험 목표
- `systemPatterns.md` - 아키텍처 및 기술 결정
- `techContext.md` - 기술, 설정 및 제약 조건
- `activeContext.md` - 현재 작업 초점 및 최근 변경 사항
- `progress.md` - 구현 상태 및 알려진 문제
- `usage/` - 구성 요소별 사용 문서
- `storybookComponents.md` - Storybook 통합 세부 사항

### AI 에이전트용

이 프로젝트에서 AI 에이전트와 작업할 때, Memory Bank는 에이전트가 다음을 이해하는 데 도움이 되는 포괄적인 지식 베이스로 작용합니다:

- 프로젝트 구조 및 패턴
- 구성 요소 API 및 사용 예시
- 개발 워크플로 및 모범 사례
- 현재 구현 상태 및 다음 단계

AI 에이전트는 이러한 파일을 읽어 프로젝트 맥락을 빠르게 파악하고, 코드 변경 및 구현에 대해 더 나은 결정을 내릴 수 있습니다.

## Tests

포괄적인 문서는 제공된 [링크](./test-utils/docs/README.md)에서 확인할 수 있습니다.