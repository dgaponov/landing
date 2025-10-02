# 🌍 webpack-i18n-assets-plugin

Un plugin pour Webpack qui remplace les appels aux fonctions de localisation (i18n) par des textes cibles.

### Fonctionnalités

- Intègre les textes i18n directement dans le bundle (tout en substituant les paramètres dans la chaîne finale)
- Génère des assets pour toutes les locales en une seule build
- Le plugin ne fonctionne que pour les builds de production !
- Ne supporte que les littéraux comme clés dans l'argument de la fonction de localisation (les template strings et les variables ne sont pas autorisés)

## 📝 Comment l'utiliser

1. Installez le package :

    ```sh
    npm i -D @gravity-ui/webpack-i18n-assets-plugin
    ```

2. Connectez le plugin à Webpack (exemple pour `@gravity-ui/app-builder`) :

    Exemple pour la configuration webpack (`webpack.config.js`) :

    ```js
    const {I18nAssetsPlugin} = require('@gravity-ui/webpack-i18n-assets-plugin');

    // For example. Read all files with localized texts and store in this mapping
    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    module.exports = {
        output: {
            filename: '[name].[locale].js', // [locale] is required in filename
        },

        plugins: [
            new I18nAssetsPlugin({
                locales
            })
        ]
    }
    ```

    Exemple si vous souhaitez créer des manifestes d'assets pour chaque locale (`webpack.config.js`) :

    ```js
    const {applyPluginToWebpackConfig} = require('@gravity-ui/webpack-i18n-assets-plugin');

    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    // Some exist webpack config
    const webpackConfig = {
        plugins: [ ... ],
        ...
    };

    // When using applyPluginToWebpackConfig, the WebpackAssetsManifest plugin will also be connected,
    // which will generate assets manifests for each locale.
    module.exports = applyPluginToWebpackConfig(webpackConfig, {locales});
    ```

    Exemple si vous utilisez `@gravity-ui/app-builder` :

    ```typescript
    import type {ServiceConfig} from '@gravity-ui/app-builder';
    import {applyPluginToWebpackConfig, Options} from '@gravity-ui/webpack-i18n-assets-plugin';

    const locales = {
        en: {},
        ru: {},
        tr: {},
    };

    // When using applyPluginToWebpackConfig, the WebpackAssetsManifest plugin will also be connected,
    // which will generate assets manifests for each locale.
    const config: ServiceConfig = {
        client: {
            webpack: (originalConfig) => applyPluginToWebpackConfig(originalConfig, {locales}),
        },
    }
    ```

3. Configurez les statiques dynamiques à partir du manifeste d'assets sur le serveur (exemple avec `@gravity-ui/app-layout`) :

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

## 🔧 Paramètres

Par défaut, le plugin est configuré pour fonctionner avec la bibliothèque [`@gravity-ui/i18n`](./frameworks/gravity-i18n.ts), mais vous pouvez personnaliser le traitement pour toute autre bibliothèque i18n.

### importResolver

Type : [`ImportResolver`](./src/types.ts#18)

La fonction qui traite les imports et marque lesquels des imports doivent être considérés comme des fonctions de localisation (par la suite, les appels aux identifiants marqués sont traités par le remplaçant).

La signature est similaire à l'original [importSpecifier](https://webpack.js.org/api/parser/#importspecifier) de webpack.

Exemple :

```typescript
const importResolver = (source: string, exportName: string, _identifierName: string, module: string) => {
    // If you need to ignore processing modules based on specific paths, you can handle such a case this way.
    if (module.startsWith('src/units/compute')) {
        return undefined;
    }

    // Processing the default import of a global function
    // import i18n from 'ui/utils/i18n'
    if (source === 'ui/utils/i18n' && exportName === 'default') {
        return {
            resolved: true,
            keyset: undefined,
        };
    }

    // Processing the import of a helper function and specifying that it belongs to the common keyset (namespace).
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

Type : [`DeclarationResolver`](./src/types.ts#30)

La fonction qui traite les déclarations de variables et marque quelles variables doivent être considérées comme des fonctions de localisation (par la suite, les appels aux identifiants marqués sont traités par la fonction de remplacement).

Exemple :

```typescript
import type {VariableDeclarator} from 'estree';

const declarationResolver = (declarator: VariableDeclarator, module: string) => {
    // If you need to ignore processing modules based on specific paths, you can handle such a case this way.
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

Une fonction qui traite les appels de fonctions de localisation et retourne un remplacement sous forme de chaîne de caractères.

Exemple :

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

Type : [`Boolean`] (par défaut - false)

Active le mode de collecte des clés inutilisées dans le projet. Après la construction, cela crée un fichier nommé `unused-keys.json`.

Pour assurer un fonctionnement correct, il est toujours nécessaire de retourner un format détaillé dans la fonction `Replacer`. Cela est important car, lors du remplacement, il existe une possibilité de modifier les clés et les keysets déterminés automatiquement.

## Paramètres des frameworks

### Gravity i18n

Fonctions pour gérer les appels de fonctions de localisation de la bibliothèque [`@gravity-ui/i18n`](https://github.com/gravity-ui/i18n).

Les fonctions prêtes à l'emploi se trouvent [`ici`](./src/frameworks/gravity-i18n.ts).

Un exemple de code avec lequel ces fonctions fonctionneront :

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

Le Replacer effectue en outre les opérations suivantes :

1. Intègre les paramètres directement dans une chaîne de caractères. Par exemple, si la valeur de la clé est la suivante :

    ```typescript
    const keyset = {
        some_key: 'string value with {{param}}'
    };

    i18nK('some_key', {param: getSomeParam()})
    // After the replacements, we will get:
    `string value with ${getSomeParam()}`
    ```

2. Substitue une fonction auto-invoquée pour les clés pluriel :

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

### Comment ce plugin se compare-t-il à [webpack-localize-assets-plugin](https://github.com/privatenumber/webpack-localize-assets-plugin) ?

Pour implémenter ce plugin, une idée du package webpack-localize-assets-plugin a été utilisée (un grand merci au créateur du package !).

Les différences sont les suivantes :

- Une API plus pratique qui vous permet de travailler avec n’importe quel type de fonctions d’internationalisation (y compris les helpers de namespaces comme useTranslation d’i18next, les fonctions importées d’autres modules, etc.)
- Une génération correcte des source maps relatives au code source
- Le support est limité à webpack 5. Le support de Webpack 4 a été supprimé.