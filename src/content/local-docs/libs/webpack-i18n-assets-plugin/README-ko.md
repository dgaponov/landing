# 🌍 webpack-i18n-assets-plugin

Webpack용 플러그인으로, 지역화 함수(i18n) 호출을 대상 텍스트로 대체합니다.

### 기능

- 번들 내에 i18n 텍스트를 인라인으로 포함시킵니다 (최종 문자열에 매개변수를 대체하면서)
- 한 번의 빌드에서 모든 로케일에 대한 에셋을 생성합니다
- 이 플러그인은 프로덕션 빌드에서만 작동합니다!
- 지역화 함수 인수의 키로 리터럴만 지원합니다 (템플릿 문자열과 변수는 허용되지 않음)

## 📝 사용 방법

1. 패키지 설치:

    ```sh
    npm i -D @gravity-ui/webpack-i18n-assets-plugin
    ```

2. Webpack에 플러그인을 연결합니다 (`@gravity-ui/app-builder` 예시):

    webpack 설정 예시 (`webpack.config.js`):

    ```js
    const {I18nAssetsPlugin} = require('@gravity-ui/webpack-i18n-assets-plugin');

    // 예시. 지역화된 텍스트가 포함된 모든 파일을 읽어 이 매핑에 저장
    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    module.exports = {
        output: {
            filename: '[name].[locale].js', // 파일명에 [locale]가 필요합니다
        },

        plugins: [
            new I18nAssetsPlugin({
                locales
            })
        ]
    }
    ```

    각 로케일에 대한 에셋 매니페스트를 생성하려면 (`webpack.config.js` 예시):

    ```js
    const {applyPluginToWebpackConfig} = require('@gravity-ui/webpack-i18n-assets-plugin');

    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    // 기존 webpack 설정
    const webpackConfig = {
        plugins: [ ... ],
        ...
    };

    // applyPluginToWebpackConfig를 사용할 때 WebpackAssetsManifest 플러그인도 함께 연결되며,
    // 각 로케일에 대한 에셋 매니페스트를 생성합니다.
    module.exports = applyPluginToWebpackConfig(webpackConfig, {locales});
    ```

    `@gravity-ui/app-builder`를 사용하는 경우 예시:

    ```typescript
    import type {ServiceConfig} from '@gravity-ui/app-builder';
    import {applyPluginToWebpackConfig, Options} from '@gravity-ui/webpack-i18n-assets-plugin';

    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    // applyPluginToWebpackConfig를 사용할 때 WebpackAssetsManifest 플러그인도 함께 연결되며,
    // 각 로케일에 대한 에셋 매니페스트를 생성합니다.
    const config: ServiceConfig = {
        client: {
            webpack: (originalConfig) => applyPluginToWebpackConfig(originalConfig, {locales}),
        },
    }
    ```

3. 서버에서 에셋 매니페스트를 사용한 동적 정적 파일 설정 (`@gravity-ui/app-layout` 예시):

    ```typescript
    import {createRenderFunction, createLayoutPlugin} from '@gravity-ui/app-layout';

    const renderLayout = createRenderFunction([
        createLayoutPlugin({
            manifest: ({lang = 'en'}) => {
                return `assets-manifest.${lang}.json`;
            },
            publicPath: '/build/',
        }),
    ]);

    app.get((req, res) => {
        res.send(
            renderLayout({
                title: 'Home page',
                pluginsOptions: {
                    layout: {
                        name: 'home',
                    },
                },
            }),
        );
    });
    ```

## 🔧 설정

기본적으로 이 플러그인은 [`@gravity-ui/i18n`](./frameworks/gravity-i18n.ts) 라이브러리와 함께 작동하도록 구성되어 있지만, 다른 i18n 라이브러리에 대한 처리를 사용자 지정할 수 있습니다.

### importResolver

유형: [`ImportResolver`](./src/types.ts#18)

임포트를 처리하고 어떤 임포트가 지역화 함수로 간주되어야 하는지 표시하는 함수입니다 (이후 표시된 식별자에 대한 호출은 replacer에 의해 처리됩니다).

시그니처는 webpack의 원본 [importSpecifier](https://webpack.js.org/api/parser/#importspecifier)와 유사합니다.

예시:

```typescript
const importResolver = (source: string, exportName: string, _identifierName: string, module: string) => {
    // 특정 경로에 기반한 모듈 처리를 무시해야 하는 경우 이렇게 처리할 수 있습니다.
    if (module.startsWith('src/units/compute')) {
        return undefined;
    }

    // 전역 함수의 기본 임포트 처리
    // import i18n from 'ui/utils/i18n'
    if (source === 'ui/utils/i18n' && exportName === 'default') {
        return {
            resolved: true,
            keyset: undefined,
        };
    }

    // 헬퍼 함수의 임포트 처리 및 공통 keyset(네임스페이스)에 속한다고 지정
    // import {ci18n} from 'ui/utils/i18n'
    if (source === 'ui/utils/i18n' && exportName === 'ci18n') {
        return {
            resolved: true,
            keyset: 'common',
        };
    }

    return undefined;
};

```

### declarationResolver

유형: [`DeclarationResolver`](./src/types.ts#30)

변수 선언을 처리하고 어떤 변수가 지역화 함수로 간주되어야 하는지 표시하는 함수입니다 (이후 표시된 식별자에 대한 호출은 replacer 함수에 의해 처리됩니다).

예시:

```typescript
import type {VariableDeclarator} from 'estree';

const declarationResolver = (declarator: VariableDeclarator, module: string) => {
    // 특정 경로에 기반한 모듈 처리를 무시해야 하는 경우 이렇게 처리할 수 있습니다.
    if (module.startsWith('src/units/compute')) {
        return undefined;
    }

```

```typescript
// Processing function declarations like const i18nK = i18n.bind(null, 'keyset');
if (
    declarator.id.type === 'Identifier' &&
    declarator.id.name.startsWith('i18n') &&
    declarator.init &&
    isI18nBind(declarator.init)
) {
    return {
        functionName: declarator.id.name,
        keyset: getKeysetFromBind(declarator.init),
    };
}

return undefined;
};
```

### replacer

Type: [`Replacer`](./src/types.ts#55)

로컬라이제이션 함수 호출을 처리하고 문자열 형태의 대체 값을 반환하는 함수입니다.

예시:

```typescript
import type {VariableDeclarator} from 'estree';
import type {ReplacerArgs, ReplacerContext} from '@gravity-ui/webpack-i18n-assets-plugin';

function replacer(
    this: ReplacerContext,
    {callNode, key: parsedKey, keyset: parsedKeyset, localeName}: ReplacerArgs,
) => {
    let key = parsedKey;
    let keyset = parsedKeyset;
    let params: Expression | SpreadElement | undefined;

    const getStringValue = (node: Expression | SpreadElement) => {
        if (node.type === 'Literal' && typeof node.value === 'string') {
            return node.value;
        }

        throw new Error('Incorrect argument type in localizer call');
    };

    // Processing a call with one argument i18nK('key')
    if (callNode.arguments.length === 1) {
        key = getStringValue(callNode.arguments[0]);
    } else if (callNode.arguments.length === 2) {
        // Processing i18n('keyset', 'key') or i18nK('key', {params})
        const [firstArg, secondArg] = callNode.arguments;

        // Call i18n('keyset', 'key')
        if (secondArg.type === 'Literal') {
            keyset = getStringValue(firstArg);
            key = getStringValue(secondArg);
        } else {
            // Call i18nK('key', {params})
            key = getStringValue(firstArg);
            params = secondArg;
        }
    } else if (callNode.arguments.length === 3) {
        // Call i18n(namespace, key, params)
        const [firstArg, secondArg, thirdArg] = callNode.arguments;
        keyset = getStringValue(firstArg);
        key = getStringValue(secondArg);
        params = thirdArg;
    } else {
        throw new Error('Incorrect count of arguments in localizer call');
    }

    // Be sure to process the key obtained from the function call argument.
    // If the function is related to a keyset, after modifying the code, the keyset can be inserted into the key (this is a plugin feature).
    // If you use the key from ReplacerArgs, it comes without the keyset and does not need to be processed.
    const keyParts = key.split('::');
    if (keyParts.length === 2) {
        key = keyParts[1];
    }

    const value = this.resolveKey(key, keyset);

    // Implement replacement options based on your needs here.
    // For example, if the key is plural, return a function call, etc.

    return JSON.stringify(value);
};
```

### collectUnusedKeys

Type: [`Boolean`] (default - false)

프로젝트에서 사용되지 않는 키를 수집하는 모드를 활성화합니다. 빌드 후 `unused-keys.json`이라는 파일을 생성합니다.

올바른 동작을 위해 `Replacer` 함수에서 항상 상세한 형식을 반환해야 합니다. 이는 대체 과정에서 자동으로 결정된 키와 키셋을 수정할 가능성이 있기 때문입니다.

## 프레임워크 설정

### Gravity i18n

[`@gravity-ui/i18n`](https://github.com/gravity-ui/i18n) 라이브러리의 로컬라이제이션 함수 호출을 처리하는 함수들입니다.

사용 준비된 함수들은 [`여기`](./src/frameworks/gravity-i18n.ts)에 있습니다.

함수들이 작동할 코드 예시:

```typescript
// The importResolver only considers the default import at the path ui/utils/i18n.
import i18n from 'ui/utils/i18n';

// The declarationResolver handles variables whose value is a call to i18n.bind.
const i18nK = i18n.bind(null, 'component.navigation');

// The replacer handles calls to identifiers found by the importResolver and declarationResolver
// This means the following calls will be processed:
i18nK('some_key');
i18nK('some_plural_key', { count: 123 });
i18nK('some_key_with_param', { someParam: 'hello' });
i18n('component.navigation', 'some_key');
i18n('component.navigation', 'some_plural_key', { count: 123 });
i18n('component.navigation', 'some_key_with_param', { someParam: 'hello' });
```

Replacer는 추가로 다음을 수행합니다:

1. 매개변수를 문자열에 인라인으로 삽입합니다. 예를 들어, 키 값이 다음과 같을 때:

    ```typescript
    const keyset = {
        some_key: 'string value with {{param}}'
    };

    i18nK('some_key', {param: getSomeParam()})
    // After the replacements, we will get:
    `string value with ${getSomeParam()}`
    ```

2. 복수 키에 대해 자체 호출 함수를 대체합니다:

    ```typescript
    const keyset = {
        pural_key: [
            'one_form {{count}}',
            'few_form {{count}}',
            'many_form {{count}}',
            'other_form {{count}}',
        ],
    };

    i18nK('pural_key', {count: getSomeCount()})

    // After the replacements, we will get:
    (function(f,c){
        const v=f[!c ? "zero" : new Intl.PluralRules("${locale}").select(c)];
        return v && v.replaceAll("{{count}}",c);
    })({
        "one": "one_form {{count}}",
        "few": "few_form {{count}}",
        "many": "many_form {{count}}",
        "other": "other_form {{count}}"
    }, getSomeCount())
    ```

## ℹ️ FAQ

### [webpack-localize-assets-plugin](https://github.com/privatenumber/webpack-localize-assets-plugin)과 비교하면 어떻게 다를까요?

이 플러그인을 구현하는 데 webpack-localize-assets-plugins 패키지의 아이디어를 사용했습니다 (패키지 제작자에게 큰 감사를 드립니다!).

차이점은 다음과 같습니다:

- 모든 종류의 국제화 함수와 함께 작업할 수 있는 더 편리한 API (i18next의 useTranslation 같은 네임스페이스 헬퍼, 다른 모듈에서 가져온 함수 등 포함)
- 소스 코드에 상대적인 소스 맵의 올바른 생성
- webpack 5만 지원합니다. webpack 4 지원은 제거되었습니다.