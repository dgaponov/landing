# 🌍 webpack-i18n-assets-plugin

Ein Plugin für Webpack, das Aufrufe von Lokalisierungsfunktionen (i18n) durch Zieltexte ersetzt.

### Features

- Inline i18n-Texte in das Bundle (während Parameter in den finalen String eingefügt werden)
- Generiert Assets für alle Sprachen in einem Build
- Das Plugin funktioniert nur für Produktionsbuilds!
- Unterstützt nur Literale als Schlüssel im Argument der Lokalisierungsfunktion (Template-Strings und Variablen sind nicht erlaubt)

## 📝 So verwenden

1. Installieren Sie das Paket:

    ```sh
    npm i -D @gravity-ui/webpack-i18n-assets-plugin
    ```

2. Verbinden Sie das Plugin mit Webpack (Beispiel für `@gravity-ui/app-builder`):

    Beispiel für die Webpack-Konfiguration (`webpack.config.js`):

    ```js
    const {I18nAssetsPlugin} = require('@gravity-ui/webpack-i18n-assets-plugin');

    // Zum Beispiel. Lesen Sie alle Dateien mit lokalen Texten und speichern Sie sie in dieser Zuordnung
    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    module.exports = {
        output: {
            filename: '[name].[locale].js', // [locale] ist im Dateinamen erforderlich
        },

        plugins: [
            new I18nAssetsPlugin({
                locales
            })
        ]
    }
    ```

    Beispiel, wenn Sie Assets-Manifeste für jede Sprache erstellen möchten (`webpack.config.js`):

    ```js
    const {applyPluginToWebpackConfig} = require('@gravity-ui/webpack-i18n-assets-plugin');

    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    // Eine bestehende Webpack-Konfiguration
    const webpackConfig = {
        plugins: [ ... ],
        ...
    };

    // Beim Verwenden von applyPluginToWebpackConfig wird auch das WebpackAssetsManifest-Plugin verbunden,
    // das Assets-Manifeste für jede Sprache generiert.
    module.exports = applyPluginToWebpackConfig(webpackConfig, {locales});
    ```

    Beispiel, wenn Sie `@gravity-ui/app-builder` verwenden:

    ```typescript
    import type {ServiceConfig} from '@gravity-ui/app-builder';
    import {applyPluginToWebpackConfig, Options} from '@gravity-ui/webpack-i18n-assets-plugin';

    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    // Beim Verwenden von applyPluginToWebpackConfig wird auch das WebpackAssetsManifest-Plugin verbunden,
    // das Assets-Manifeste für jede Sprache generiert.
    const config: ServiceConfig = {
        client: {
            webpack: (originalConfig) => applyPluginToWebpackConfig(originalConfig, {locales}),
        },
    }
    ```

3. Konfigurieren Sie dynamische Statiken aus dem Asset-Manifest auf dem Server (Beispiel mit `@gravity-ui/app-layout`):

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

## 🔧 Einstellungen

Standardmäßig ist das Plugin so konfiguriert, dass es mit der [`@gravity-ui/i18n`](./frameworks/gravity-i18n.ts)-Bibliothek funktioniert, aber Sie können die Verarbeitung für jede andere i18n-Bibliothek anpassen.

### importResolver

Typ: [`ImportResolver`](./src/types.ts#18)

Die Funktion, die Imports verarbeitet und markiert, welche der Imports als Lokalisierungsfunktionen betrachtet werden sollen (anschließend werden Aufrufe der markierten Identifier vom Replacer verarbeitet).

Die Signatur ist ähnlich wie der originale [importSpecifier](https://webpack.js.org/api/parser/#importspecifier) aus Webpack.

Beispiel:

```typescript
const importResolver = (source: string, exportName: string, _identifierName: string, module: string) => {
    // Wenn Sie Module basierend auf spezifischen Pfaden ignorieren möchten, können Sie das so handhaben.
    if (module.startsWith('src/units/compute')) {
        return undefined;
    }

    // Verarbeitung des Default-Imports einer globalen Funktion
    // import i18n from 'ui/utils/i18n'
    if (source === 'ui/utils/i18n' && exportName === 'default') {
        return {
            resolved: true,
            keyset: undefined,
        };
    }

    // Verarbeitung des Imports einer Hilfsfunktion und Angabe, dass sie zum gemeinsamen Keyset (Namespace) gehört.
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

Typ: [`DeclarationResolver`](./src/types.ts#30)

Die Funktion, die Variablendeklarationen verarbeitet und markiert, welche Variablen als Lokalisierungsfunktionen betrachtet werden sollen (anschließend werden Aufrufe der markierten Identifier von der Replacer-Funktion verarbeitet).

Beispiel:

```typescript
import type {VariableDeclarator} from 'estree';

const declarationResolver = (declarator: VariableDeclarator, module: string) => {
    // Wenn Sie Module basierend auf spezifischen Pfaden ignorieren möchten, können Sie das so handhaben.
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

Typ: [`Replacer`](./src/types.ts#55)

Eine Funktion, die Aufrufe von Lokalisierungsfunktionen verarbeitet und eine Ersetzung als String zurückgibt.

Beispiel:

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

Typ: [`Boolean`] (Standard – false)

Aktiviert den Modus zur Sammlung ungenutzter Schlüssel im Projekt. Nach dem Build wird eine Datei namens `unused-keys.json` erstellt.

Um eine ordnungsgemäße Funktionalität zu gewährleisten, muss immer ein detailliertes Format in der `Replacer`-Funktion zurückgegeben werden. Dies ist wichtig, da bei der Ersetzung die Möglichkeit besteht, automatisch bestimmte Schlüssel und Keysets zu modifizieren.

## Framework-Einstellungen

### Gravity i18n

Funktionen zur Handhabung von Aufrufen von Lokalisierungsfunktionen aus der Bibliothek [`@gravity-ui/i18n`](https://github.com/gravity-ui/i18n).

Die einsatzbereiten Funktionen finden Sie [`hier`](./src/frameworks/gravity-i18n.ts).

Ein Beispiel für Code, mit dem die Funktionen arbeiten:

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

Der Replacer führt zusätzlich Folgendes aus:

1. Inline-Integration der Parameter in einen String. Zum Beispiel, wenn der Schlüsselwert wie folgt lautet:

    ```typescript
    const keyset = {
        some_key: 'string value with {{param}}'
    };

    i18nK('some_key', {param: getSomeParam()})
    // After the replacements, we will get:
    `string value with ${getSomeParam()}`
    ```

2. Ersetzung durch eine selbstaufrufende Funktion für Plural-Schlüssel:

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

```

### Wie unterscheidet sich das von [webpack-localize-assets-plugin](https://github.com/privatenumber/webpack-localize-assets-plugin)?

Um dieses Plugin zu implementieren, wurde eine Idee aus dem webpack-localize-assets-plugin-Paket übernommen (vielen Dank an den Paket-Ersteller!).

Die Unterschiede sind wie folgt:

- Eine bequemere API, die es ermöglicht, mit allen Arten von Internationalisierungsfunktionen zu arbeiten (einschließlich Namespace-Helfern wie useTranslation aus i18next, importierten Funktionen aus anderen Modulen usw.)
- Korrekte Generierung von Source Maps relativ zum Quellcode
- Es gibt nur Unterstützung für webpack 5. Die Unterstützung für Webpack 4 wurde entfernt.