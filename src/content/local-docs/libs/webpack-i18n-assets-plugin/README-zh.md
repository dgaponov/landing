# 🌍 webpack-i18n-assets-plugin

Webpack 插件，用于将本地化函数（i18n）调用替换为目标文本。

### 功能特性

- 将 i18n 文本内联到捆绑包中（同时将参数替换到最终字符串中）
- 在一次构建中为所有语言环境生成资源
- 该插件仅适用于生产构建！
- 仅支持本地化函数参数中的字面量作为键（不支持模板字符串和变量）

## 📝 如何使用

1. 安装包：

    ```sh
    npm i -D @gravity-ui/webpack-i18n-assets-plugin
    ```

2. 将插件连接到 Webpack（以 `@gravity-ui/app-builder` 为例）：

    Webpack 配置示例（`webpack.config.js`）：

    ```js
    const {I18nAssetsPlugin} = require('@gravity-ui/webpack-i18n-assets-plugin');

    // 示例：读取所有包含本地化文本的文件，并存储到此映射中
    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    module.exports = {
        output: {
            filename: '[name].[locale].js', // 文件名中必须包含 [locale]
        },

        plugins: [
            new I18nAssetsPlugin({
                locales
            })
        ]
    }
    ```

    如果您想为每个语言环境创建资源清单的示例（`webpack.config.js`）：

    ```js
    const {applyPluginToWebpackConfig} = require('@gravity-ui/webpack-i18n-assets-plugin');

    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    // 现有的 Webpack 配置
    const webpackConfig = {
        plugins: [ ... ],
        ...
    };

    // 使用 applyPluginToWebpackConfig 时，还会连接 WebpackAssetsManifest 插件，
    // 该插件将为每个语言环境生成资源清单。
    module.exports = applyPluginToWebpackConfig(webpackConfig, {locales});
    ```

    如果您使用 `@gravity-ui/app-builder` 的示例：

    ```typescript
    import type {ServiceConfig} from '@gravity-ui/app-builder';
    import {applyPluginToWebpackConfig, Options} from '@gravity-ui/webpack-i18n-assets-plugin';

    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    // 使用 applyPluginToWebpackConfig 时，还会连接 WebpackAssetsManifest 插件，
    // 该插件将为每个语言环境生成资源清单。
    const config: ServiceConfig = {
        client: {
            webpack: (originalConfig) => applyPluginToWebpackConfig(originalConfig, {locales}),
        },
    }
    ```

3. 在服务器端配置来自资源清单的动态静态资源（以 `@gravity-ui/app-layout` 为例）：

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

## 🔧 配置选项

默认情况下，该插件配置为与 [`@gravity-ui/i18n`](./frameworks/gravity-i18n.ts) 库配合使用，但您可以自定义处理以适应其他 i18n 库。

### importResolver

类型：[`ImportResolver`](./src/types.ts#18)

用于处理导入并标记哪些导入应被视为本地化函数的函数（随后，对标记的标识符的调用将由替换器处理）。

其签名类似于 Webpack 原生的 [importSpecifier](https://webpack.js.org/api/parser/#importspecifier)。

示例：

```typescript
const importResolver = (source: string, exportName: string, _identifierName: string, module: string) => {
    // 如果需要基于特定路径忽略处理模块，可以这样处理。
    if (module.startsWith('src/units/compute')) {
        return undefined;
    }

    // 处理全局函数的默认导入
    // import i18n from 'ui/utils/i18n'
    if (source === 'ui/utils/i18n' && exportName === 'default') {
        return {
            resolved: true,
            keyset: undefined,
        };
    }

    // 处理辅助函数的导入，并指定它属于公共 keyset（命名空间）。
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

类型：[`DeclarationResolver`](./src/types.ts#30)

用于处理变量声明并标记哪些变量应被视为本地化函数的函数（随后，对标记的标识符的调用将由替换器函数处理）。

示例：

```typescript
import type {VariableDeclarator} from 'estree';

const declarationResolver = (declarator: VariableDeclarator, module: string) => {
    // 如果需要基于特定路径忽略处理模块，可以这样处理。
    if (module.startsWith('src/units/compute')) {
        return undefined;
    }

```

### importResolver

类型：[`ImportResolver`](./src/types.ts#55)

一个用于处理导入语句的函数，返回一个字符串，该字符串是导入的标识符名称。

示例：

```typescript
import type {ImportDeclaration} from 'estree';
import type {ImportResolverArgs, ImportResolverContext} from '@gravity-ui/webpack-i18n-assets-plugin';

function importResolver(
    this: ImportResolverContext,
    {node}: ImportResolverArgs,
) {
    // 处理默认导入
    if (node.specifiers.length === 1 && node.specifiers[0].type === 'ImportDefaultSpecifier') {
        return node.specifiers[0].local.name;
    }

    // 处理命名导入
    if (node.specifiers.length === 1 && node.specifiers[0].type === 'ImportSpecifier') {
        return node.specifiers[0].imported.name;
    }

    return undefined;
}
```

### declarationResolver

类型：[`DeclarationResolver`](./src/types.ts#55)

一个用于处理变量声明的函数，返回一个对象，该对象包含函数名称和 keyset。

示例：

```typescript
import type {VariableDeclarator} from 'estree';
import type {DeclarationResolverArgs} from '@gravity-ui/webpack-i18n-assets-plugin';

function isI18nBind(node: any) {
    return (
        node.type === 'CallExpression' &&
        node.callee.type === 'MemberExpression' &&
        node.callee.object.name === 'i18n' &&
        node.callee.property.name === 'bind' &&
        node.arguments.length === 2 &&
        node.arguments[0].type === 'Literal' &&
        node.arguments[0].value === null
    );
}

function getKeysetFromBind(node: any) {
    return node.arguments[1].value;
}

function declarationResolver({declarator}: DeclarationResolverArgs) {
    // 处理像 const i18nK = i18n.bind(null, 'keyset'); 这样的函数声明
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
}
```

### replacer

类型：[`Replacer`](./src/types.ts#55)

一个处理本地化函数调用的函数，并返回一个字符串作为替换内容。

示例：

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

    // 处理只有一个参数的调用 i18nK('key')
    if (callNode.arguments.length === 1) {
        key = getStringValue(callNode.arguments[0]);
    } else if (callNode.arguments.length === 2) {
        // 处理 i18n('keyset', 'key') 或 i18nK('key', {params})
        const [firstArg, secondArg] = callNode.arguments;

        // 调用 i18n('keyset', 'key')
        if (secondArg.type === 'Literal') {
            keyset = getStringValue(firstArg);
            key = getStringValue(secondArg);
        } else {
            // 调用 i18nK('key', {params})
            key = getStringValue(firstArg);
            params = secondArg;
        }
    } else if (callNode.arguments.length === 3) {
        // 调用 i18n(namespace, key, params)
        const [firstArg, secondArg, thirdArg] = callNode.arguments;
        keyset = getStringValue(firstArg);
        key = getStringValue(secondArg);
        params = thirdArg;
    } else {
        throw new Error('Incorrect count of arguments in localizer call');
    }

    // 确保处理从函数调用参数中获取的 key。
    // 如果函数与 keyset 相关，在修改代码后，可以将 keyset 插入到 key 中（这是插件的功能）。
    // 如果使用 ReplacerArgs 中的 key，它不包含 keyset，因此无需处理。
    const keyParts = key.split('::');
    if (keyParts.length === 2) {
        key = keyParts[1];
    }

    const value = this.resolveKey(key, keyset);

    // 根据需求在这里实现替换选项。
    // 例如，如果 key 是复数形式，则返回一个函数调用等。

    return JSON.stringify(value);
}
```

### collectUnusedKeys

类型：[`Boolean`]（默认值 - false）

启用项目中收集未使用 key 的模式。构建后，会创建一个名为 `unused-keys.json` 的文件。

为了确保正常工作，在 `Replacer` 函数中始终需要返回详细格式。这很重要，因为在替换过程中，可能需要修改自动确定的 key 和 keyset。

## 框架设置

### Gravity i18n

用于处理来自库 [`@gravity-ui/i18n`](https://github.com/gravity-ui/i18n) 的本地化函数调用的函数。

现成的函数位于 [`这里`](./src/frameworks/gravity-i18n.ts)。

函数处理的代码示例：

```typescript
// importResolver 只考虑路径 ui/utils/i18n 的默认导入
import i18n from 'ui/utils/i18n';

// declarationResolver 处理值为 i18n.bind 调用的变量
const i18nK = i18n.bind(null, 'component.navigation');

// replacer 处理 importResolver 和 declarationResolver 找到的标识符调用
// 这意味着以下调用将被处理：
i18nK('some_key');
i18nK('some_plural_key', { count: 123 });
i18nK('some_key_with_param', { someParam: 'hello' });
i18n('component.navigation', 'some_key');
i18n('component.navigation', 'some_plural_key', { count: 123 });
i18n('component.navigation', 'some_key_with_param', { someParam: 'hello' });
```

Replacer 还会额外执行以下操作：

1. 将参数内联到字符串中。例如，如果 key 的值如下：

    ```typescript
    const keyset = {
        some_key: 'string value with {{param}}'
    };

    i18nK('some_key', {param: getSomeParam()})
    // 替换后，我们将得到：
    `string value with ${getSomeParam()}`
    ```

2. 为复数 key 替换为自调用函数：

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

    // 替换后，我们将得到：
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

## ℹ️ 常见问题解答

### 这与 [webpack-localize-assets-plugin](https://github.com/privatenumber/webpack-localize-assets-plugin) 相比如何？

在实现这个插件时，我们借鉴了 webpack-localize-assets-plugin 包中的一个想法（在此感谢该包的创建者！）。

差异如下：

- 更便捷的 API，可以处理任何类型的国际化函数（包括像 i18next 中的 useTranslation 这样的命名空间助手、从其他模块导入的函数等）
- 正确生成相对于源代码的源映射
- 仅支持 webpack 5，已移除对 webpack 4 的支持