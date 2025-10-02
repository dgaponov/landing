# 🌍 webpack-i18n-assets-plugin

Плагин для Webpack, который заменяет вызовы функций локализации (i18n) на целевые тексты.

### Возможности

- Встраивает тексты i18n в бандл (при этом подставляя параметры в итоговую строку)
- Генерирует ассеты для всех локалей в одной сборке
- Плагин работает только для продакшн-сборок!
- Поддерживает только литералы в качестве ключей в аргументах функции локализации (шаблонные строки и переменные не поддерживаются)

## 📝 Как использовать

1. Установите пакет:

    ```sh
    npm i -D @gravity-ui/webpack-i18n-assets-plugin
    ```

2. Подключите плагин к Webpack (пример для `@gravity-ui/app-builder`):

    Пример для конфигурации webpack (`webpack.config.js`):

    ```js
    const {I18nAssetsPlugin} = require('@gravity-ui/webpack-i18n-assets-plugin');

    // Например. Прочитайте все файлы с локализованными текстами и сохраните в этом маппинге
    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    module.exports = {
        output: {
            filename: '[name].[locale].js', // [locale] обязательно в имени файла
        },

        plugins: [
            new I18nAssetsPlugin({
                locales
            })
        ]
    }
    ```

    Пример, если вы хотите создать манифесты ассетов для каждой локали (`webpack.config.js`):

    ```js
    const {applyPluginToWebpackConfig} = require('@gravity-ui/webpack-i18n-assets-plugin');

    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    // Какой-то существующий конфиг webpack
    const webpackConfig = {
        plugins: [ ... ],
        ...
    };

    // При использовании applyPluginToWebpackConfig также подключается плагин WebpackAssetsManifest,
    // который сгенерирует манифесты ассетов для каждой локали.
    module.exports = applyPluginToWebpackConfig(webpackConfig, {locales});
    ```

    Пример, если вы используете `@gravity-ui/app-builder`:

    ```typescript
    import type {ServiceConfig} from '@gravity-ui/app-builder';
    import {applyPluginToWebpackConfig, Options} from '@gravity-ui/webpack-i18n-assets-plugin';

    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    // При использовании applyPluginToWebpackConfig также подключается плагин WebpackAssetsManifest,
    // который сгенерирует манифесты ассетов для каждой локали.
    const config: ServiceConfig = {
        client: {
            webpack: (originalConfig) => applyPluginToWebpackConfig(originalConfig, {locales}),
        },
    }
    ```

3. Настройте динамические статики из манифеста ассетов на сервере (пример с `@gravity-ui/app-layout`):

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

## 🔧 Настройки

По умолчанию плагин настроен для работы с библиотекой [`@gravity-ui/i18n`](./frameworks/gravity-i18n.ts), но вы можете настроить обработку для любой другой библиотеки i18n.

### importResolver

Тип: [`ImportResolver`](./src/types.ts#18)

Функция, которая обрабатывает импорты и помечает, какие из импортов следует считать функциями локализации (после этого вызовы помеченных идентификаторов обрабатываются функцией замены).

Сигнатура похожа на оригинальный [importSpecifier](https://webpack.js.org/api/parser/#importspecifier) из webpack.

Пример:

```typescript
const importResolver = (source: string, exportName: string, _identifierName: string, module: string) => {
    // Если нужно игнорировать обработку модулей на основе конкретных путей, можно обработать такой случай таким образом.
    if (module.startsWith('src/units/compute')) {
        return undefined;
    }

    // Обработка дефолтного импорта глобальной функции
    // import i18n from 'ui/utils/i18n'
    if (source === 'ui/utils/i18n' && exportName === 'default') {
        return {
            resolved: true,
            keyset: undefined,
        };
    }

    // Обработка импорта вспомогательной функции и указание, что она относится к общему keyset (пространству имен).
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

Тип: [`DeclarationResolver`](./src/types.ts#30)

Функция, которая обрабатывает объявления переменных и помечает, какие переменные следует считать функциями локализации (после этого вызовы помеченных идентификаторов обрабатываются функцией замены).

Пример:

```typescript
import type {VariableDeclarator} from 'estree';

const declarationResolver = (declarator: VariableDeclarator, module: string) => {
    // Если нужно игнорировать обработку модулей на основе конкретных путей, можно обработать такой случай таким образом.
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

Тип: [`Replacer`](./src/types.ts#55)

Функция, которая обрабатывает вызовы функций локализации и возвращает замену в виде строки.

Пример:

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

    // Обработка вызова с одним аргументом i18nK('key')
    if (callNode.arguments.length === 1) {
        key = getStringValue(callNode.arguments[0]);
    } else if (callNode.arguments.length === 2) {
        // Обработка i18n('keyset', 'key') или i18nK('key', {params})
        const [firstArg, secondArg] = callNode.arguments;

        // Вызов i18n('keyset', 'key')
        if (secondArg.type === 'Literal') {
            keyset = getStringValue(firstArg);
            key = getStringValue(secondArg);
        } else {
            // Вызов i18nK('key', {params})
            key = getStringValue(firstArg);
            params = secondArg;
        }
    } else if (callNode.arguments.length === 3) {
        // Вызов i18n(namespace, key, params)
        const [firstArg, secondArg, thirdArg] = callNode.arguments;
        keyset = getStringValue(firstArg);
        key = getStringValue(secondArg);
        params = thirdArg;
    } else {
        throw new Error('Incorrect count of arguments in localizer call');
    }

    // Обязательно обработайте ключ, полученный из аргумента вызова функции.
    // Если функция связана с набором ключей, после изменения кода набор ключей может быть вставлен в ключ (это функция плагина).
    // Если вы используете ключ из ReplacerArgs, он приходит без набора ключей и не требует обработки.
    const keyParts = key.split('::');
    if (keyParts.length === 2) {
        key = keyParts[1];
    }

    const value = this.resolveKey(key, keyset);

    // Здесь реализуйте варианты замены в зависимости от ваших нужд.
    // Например, если ключ множественный, верните вызов функции и т.д.

    return JSON.stringify(value);
};
```

### collectUnusedKeys

Тип: [`Boolean`] (по умолчанию — false)

Включает режим сбора неиспользуемых ключей в проекте. После сборки создаёт файл с именем `unused-keys.json`.

Для корректной работы всегда необходимо возвращать подробный формат в функции `Replacer`. Это важно, поскольку во время замены есть вероятность изменения автоматически определённых ключей и наборов ключей.

## Настройки фреймворков

### Gravity i18n

Функции для обработки вызовов функций локализации из библиотеки [`@gravity-ui/i18n`](https://github.com/gravity-ui/i18n).

Готовые к использованию функции находятся [`здесь`](./src/frameworks/gravity-i18n.ts).

Пример кода, с которым работают функции:

```typescript
// importResolver учитывает только дефолтный импорт по пути ui/utils/i18n.
import i18n from 'ui/utils/i18n';

// declarationResolver обрабатывает переменные, значение которых — вызов i18n.bind.
const i18nK = i18n.bind(null, 'component.navigation');

// replacer обрабатывает вызовы идентификаторов, найденных importResolver и declarationResolver.
// Это значит, что следующие вызовы будут обработаны:
i18nK('some_key');
i18nK('some_plural_key', { count: 123 });
i18nK('some_key_with_param', { someParam: 'hello' });
i18n('component.navigation', 'some_key');
i18n('component.navigation', 'some_plural_key', { count: 123 });
i18n('component.navigation', 'some_key_with_param', { someParam: 'hello' });
```

Replacer дополнительно выполняет следующее:

1. Встраивает параметры в строку. Например, если значение ключа выглядит так:

    ```typescript
    const keyset = {
        some_key: 'string value with {{param}}'
    };

    i18nK('some_key', {param: getSomeParam()})
    // После замены мы получим:
    `string value with ${getSomeParam()}`
    ```

2. Подставляет самовызывающуюся функцию для множественных ключей:

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

    // После замены мы получим:
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

### Чем это отличается от [webpack-localize-assets-plugin](https://github.com/privatenumber/webpack-localize-assets-plugin)?

Для реализации этого плагина была использована идея из пакета webpack-localize-assets-plugins (большое спасибо создателю пакета!).

Различия следующие:

- Более удобный API, который позволяет работать с любыми функциями интернационализации (включая helpers с пространствами имён, такие как useTranslation из i18next, импортированные функции из других модулей и т. д.)
- Корректная генерация source maps относительно исходного кода
- Поддержка только webpack 5. Поддержка webpack 4 удалена.