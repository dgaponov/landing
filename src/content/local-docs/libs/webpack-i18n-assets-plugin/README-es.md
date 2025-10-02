# 🌍 webpack-i18n-assets-plugin

Un plugin para Webpack que reemplaza las llamadas a funciones de localización (i18n) con textos objetivo.

### Características

- Incorpora textos i18n directamente en el bundle (mientras sustituye parámetros en la cadena final)
- Genera assets para todos los locales en una sola compilación
- ¡El plugin solo funciona para builds de producción!
- Solo soporta literales como claves en el argumento de la función de localización (no se permiten cadenas de plantilla ni variables)

## 📝 Cómo usar

1. Instala el paquete:

    ```sh
    npm i -D @gravity-ui/webpack-i18n-assets-plugin
    ```

2. Conecta el plugin a Webpack (ejemplo para `@gravity-ui/app-builder`):

    Ejemplo para la configuración de webpack (`webpack.config.js`):

    ```js
    const {I18nAssetsPlugin} = require('@gravity-ui/webpack-i18n-assets-plugin');

    // Por ejemplo. Lee todos los archivos con textos localizados y guárdalos en este mapeo
    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    module.exports = {
        output: {
            filename: '[name].[locale].js', // [locale] es requerido en el nombre del archivo
        },

        plugins: [
            new I18nAssetsPlugin({
                locales
            })
        ]
    }
    ```

    Ejemplo si quieres crear manifiestos de assets para cada locale (`webpack.config.js`):

    ```js
    const {applyPluginToWebpackConfig} = require('@gravity-ui/webpack-i18n-assets-plugin');

    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    // Alguna configuración de webpack existente
    const webpackConfig = {
        plugins: [ ... ],
        ...
    };

    // Al usar applyPluginToWebpackConfig, también se conectará el plugin WebpackAssetsManifest,
    // que generará manifiestos de assets para cada locale.
    module.exports = applyPluginToWebpackConfig(webpackConfig, {locales});
    ```

    Ejemplo si usas `@gravity-ui/app-builder`:

    ```typescript
    import type {ServiceConfig} from '@gravity-ui/app-builder';
    import {applyPluginToWebpackConfig, Options} from '@gravity-ui/webpack-i18n-assets-plugin';

    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    // Al usar applyPluginToWebpackConfig, también se conectará el plugin WebpackAssetsManifest,
    // que generará manifiestos de assets para cada locale.
    const config: ServiceConfig = {
        client: {
            webpack: (originalConfig) => applyPluginToWebpackConfig(originalConfig, {locales}),
        },
    }
    ```

3. Configura estáticos dinámicos desde el manifiesto de assets en el servidor (ejemplo con `@gravity-ui/app-layout`):

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

## 🔧 Configuraciones

Por defecto, el plugin está configurado para trabajar con la biblioteca [`@gravity-ui/i18n`](./frameworks/gravity-i18n.ts), pero puedes personalizar el procesamiento para cualquier otra biblioteca i18n.

### importResolver

Tipo: [`ImportResolver`](./src/types.ts#18)

La función que procesa las importaciones y marca cuáles de ellas deben considerarse como funciones de localización (posteriormente, las llamadas a los identificadores marcados se procesan con el reemplazador).

La firma es similar a la del [importSpecifier](https://webpack.js.org/api/parser/#importspecifier) original de webpack.

Ejemplo:

```typescript
const importResolver = (source: string, exportName: string, _identifierName: string, module: string) => {
    // Si necesitas ignorar el procesamiento de módulos basados en rutas específicas, puedes manejar ese caso de esta manera.
    if (module.startsWith('src/units/compute')) {
        return undefined;
    }

    // Procesamiento de la importación por defecto de una función global
    // import i18n from 'ui/utils/i18n'
    if (source === 'ui/utils/i18n' && exportName === 'default') {
        return {
            resolved: true,
            keyset: undefined,
        };
    }

    // Procesamiento de la importación de una función auxiliar y especificación de que pertenece al keyset común (namespace).
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

Tipo: [`DeclarationResolver`](./src/types.ts#30)

La función que procesa las declaraciones de variables y marca cuáles variables deben considerarse como funciones de localización (posteriormente, las llamadas a los identificadores marcados se procesan con la función de reemplazo).

Ejemplo:

```typescript
import type {VariableDeclarator} from 'estree';

const declarationResolver = (declarator: VariableDeclarator, module: string) => {
    // Si necesitas ignorar el procesamiento de módulos basados en rutas específicas, puedes manejar ese caso de esta manera.
    if (module.startsWith('src/units/compute')) {
        return undefined;
    }

```

# webpack-i18n-assets-plugin

[![npm version](https://img.shields.io/npm/v/@gravity-ui/webpack-i18n-assets-plugin.svg)](https://www.npmjs.com/package/@gravity-ui/webpack-i18n-assets-plugin)
[![Downloads](https://img.shields.io/npm/dm/@gravity-ui/webpack-i18n-assets-plugin.svg)](https://www.npmjs.com/package/@gravity-ui/webpack-i18n-assets-plugin)
[![Bundlephobia](https://img.shields.io/bundlephobia/minzip/@gravity-ui/webpack-i18n-assets-plugin.svg)](https://bundlephobia.com/package/@gravity-ui/webpack-i18n-assets-plugin)
[![License](https://img.shields.io/github/license/gravity-ui/webpack-i18n-assets-plugin.svg)](https://github.com/gravity-ui/webpack-i18n-assets-plugin)

Un plugin para Webpack que extrae las cadenas de texto de los archivos de origen y las reemplaza con llamadas a funciones de localización durante la compilación. Esto permite cargar solo las traducciones necesarias para el locale actual.

## 🚀 Características principales

- Soporte para llamadas a funciones de localización con argumentos dinámicos.
- Extracción de claves de localización a archivos JSON separados.
- Reemplazo de llamadas a funciones de localización con cadenas de texto traducidas.
- Soporte para pluralización y parámetros en las traducciones.
- Integración con marcos de trabajo populares como React, Vue y Angular.
- Generación de informes sobre claves no utilizadas.

## 📦 Instalación

```bash
npm install --save-dev @gravity-ui/webpack-i18n-assets-plugin
```

## 🔧 Configuración básica

Agrega el plugin a tu configuración de Webpack:

```javascript
const I18nAssetsPlugin = require('@gravity-ui/webpack-i18n-assets-plugin');

module.exports = {
    plugins: [
        new I18nAssetsPlugin({
            // Opciones de configuración
            locales: ['en', 'es', 'fr'], // Idiomas soportados
            keysetsPath: './src/locales', // Ruta a los archivos de keysets
            outputPath: './dist/locales', // Ruta de salida para los archivos traducidos
        }),
    ],
};
```

## ⚙️ Opciones de configuración

### locales

**Tipo:** `Array<string>` (predeterminado - `[]`)

Lista de códigos de idioma para los que se generarán los archivos de localización. El plugin procesará solo las llamadas a funciones de localización para estos idiomas.

**Ejemplo:**

```javascript
locales: ['en', 'ru', 'de'],
```

### keysetsPath

**Tipo:** `string` (predeterminado - `'src/locales'`)

Ruta relativa desde la raíz del proyecto donde se encuentran los archivos de keysets (archivos JSON con traducciones).

**Ejemplo:**

```javascript
keysetsPath: './i18n/keysets',
```

### outputPath

**Tipo:** `string` (predeterminado - `'dist/locales'`)

Ruta de salida para los archivos de localización generados. Se creará un subdirectorio para cada idioma.

**Ejemplo:**

```javascript
outputPath: './build/i18n',
```

### importResolver

**Tipo:** [`ImportResolver`](./src/types.ts#12)

Una función que identifica las importaciones de funciones de localización en el código fuente.

**Ejemplo:**

```typescript
import type {ImportDeclaration} from 'estree';
import type {ImportResolverArgs} from '@gravity-ui/webpack-i18n-assets-plugin';

function importResolver({node}: ImportResolverArgs) {
    // Busca importaciones de funciones de localización conocidas
    if (
        node.type === 'ImportDeclaration' &&
        node.source.value === '@gravity-ui/i18n'
    ) {
        return {
            functionName: 'i18n',
            sourcePath: node.source.value,
        };
    }

    return undefined;
}
```

### declarationResolver

**Tipo:** [`DeclarationResolver`](./src/types.ts#28)

Una función que procesa las declaraciones de variables que contienen funciones de localización vinculadas (por ejemplo, `i18n.bind(null, 'keyset')`).

**Ejemplo:**

```typescript
import type {VariableDeclarator} from 'estree';
import type {DeclarationResolverArgs} from '@gravity-ui/webpack-i18n-assets-plugin';

function isI18nBind(node: any) {
    return (
        node.type === 'CallExpression' &&
        node.callee.type === 'MemberExpression' &&
        node.callee.property.name === 'bind' &&
        node.arguments.length >= 1
    );
}

function getKeysetFromBind(node: any) {
    const arg = node.arguments[0];
    if (arg.type === 'Literal' && typeof arg.value === 'string') {
        return arg.value;
    }
    return undefined;
}

function declarationResolver({declarator}: DeclarationResolverArgs) {
    // Procesamiento de declaraciones de funciones como const i18nK = i18n.bind(null, 'keyset');
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

**Tipo:** [`Replacer`](./src/types.ts#55)

Una función que procesa las llamadas a funciones de localización y devuelve un reemplazo como una cadena.

**Ejemplo:**

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

        throw new Error('Tipo de argumento incorrecto en la llamada al localizador');
    };

    // Procesamiento de una llamada con un argumento i18nK('key')
    if (callNode.arguments.length === 1) {
        key = getStringValue(callNode.arguments[0]);
    } else if (callNode.arguments.length === 2) {
        // Procesamiento de i18n('keyset', 'key') o i18nK('key', {params})
        const [firstArg, secondArg] = callNode.arguments;

        // Llamada i18n('keyset', 'key')
        if (secondArg.type === 'Literal') {
            keyset = getStringValue(firstArg);
            key = getStringValue(secondArg);
        } else {
            // Llamada i18nK('key', {params})
            key = getStringValue(firstArg);
            params = secondArg;
        }
    } else if (callNode.arguments.length === 3) {
        // Llamada i18n(namespace, key, params)
        const [firstArg, secondArg, thirdArg] = callNode.arguments;
        keyset = getStringValue(firstArg);
        key = getStringValue(secondArg);
        params = thirdArg;
    } else {
        throw new Error('Número incorrecto de argumentos en la llamada al localizador');
    }

    // Asegúrate de procesar la clave obtenida del argumento de la llamada a la función.
    // Si la función está relacionada con un keyset, después de modificar el código, el keyset puede insertarse en la clave (esta es una característica del plugin).
    // Si usas la clave de ReplacerArgs, viene sin el keyset y no necesita procesarse.
    const keyParts = key.split('::');
    if (keyParts.length === 2) {
        key = keyParts[1];
    }

    const value = this.resolveKey(key, keyset);

    // Implementa opciones de reemplazo según tus necesidades aquí.
    // Por ejemplo, si la clave es plural, devuelve una llamada a función, etc.

    return JSON.stringify(value);
};
```

### collectUnusedKeys

**Tipo:** [`Boolean`] (predeterminado - false)

Habilita el modo para recopilar claves no utilizadas en el proyecto. Después de la compilación, crea un archivo llamado `unused-keys.json`.

Para garantizar un funcionamiento adecuado, siempre es necesario devolver un formato detallado en la función `Replacer`. Esto es importante porque durante el reemplazo, existe la posibilidad de modificar claves y keysets determinados automáticamente.

## Configuraciones de marcos de trabajo

### Gravity i18n

Funciones para manejar llamadas a funciones de localización de la biblioteca [`@gravity-ui/i18n`](https://github.com/gravity-ui/i18n).

Las funciones listas para usar se encuentran [`aquí`](./src/frameworks/gravity-i18n.ts).

Un ejemplo del código con el que funcionarán las funciones:

```typescript
// El importResolver solo considera la importación predeterminada en la ruta ui/utils/i18n.
import i18n from 'ui/utils/i18n';

// El declarationResolver maneja variables cuyo valor es una llamada a i18n.bind.
const i18nK = i18n.bind(null, 'component.navigation');

// El replacer maneja llamadas a identificadores encontrados por el importResolver y declarationResolver
// Esto significa que las siguientes llamadas se procesarán:
i18nK('some_key');
i18nK('some_plural_key', { count: 123 });
i18nK('some_key_with_param', { someParam: 'hello' });
i18n('component.navigation', 'some_key');
i18n('component.navigation', 'some_plural_key', { count: 123 });
i18n('component.navigation', 'some_key_with_param', { someParam: 'hello' });
```

El Replacer realiza adicionalmente lo siguiente:

1. Inserta los parámetros en una cadena. Por ejemplo, si el valor de la clave es el siguiente:

    ```typescript
    const keyset = {
        some_key: 'string value with {{param}}'
    };

    i18nK('some_key', {param: getSomeParam()})
    // Después de los reemplazos, obtendremos:
    `string value with ${getSomeParam()}`
    ```

2. Sustituye una función autoejecutable para claves plurales:

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

    // Después de los reemplazos, obtendremos:
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

## ℹ️ Preguntas frecuentes

<!-- Aquí iría el HTML snippet con opciones de idioma si estuviera presente en el archivo fuente, pero no se muestra en el texto proporcionado. Mantener en su lugar original. -->

### ¿Cómo se compara esto con [webpack-localize-assets-plugin](https://github.com/privatenumber/webpack-localize-assets-plugin)?

Para implementar este plugin, se utilizó una idea del paquete webpack-localize-assets-plugins (¡muchas gracias al creador del paquete!).

Las diferencias son las siguientes:

- Una API más conveniente que permite trabajar con cualquier tipo de funciones de internacionalización (incluyendo helpers de namespaces como useTranslation de i18next, funciones importadas de otros módulos, etc.)
- Generación correcta de source maps relativa al código fuente
- Solo hay soporte para webpack 5. Se ha eliminado el soporte para Webpack 4.