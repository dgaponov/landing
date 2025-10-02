# @gravity-ui/i18n &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/i18n)](https://www.npmjs.com/package/@gravity-ui/i18n) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/i18n/.github/workflows/ci.yml?branch=main&label=CI&logo=github)](https://github.com/gravity-ui/i18n/actions/workflows/ci.yml?query=branch:main)

## I18N-Hilfsfunktionen

Die Hilfsfunktionen im I18N-Paket sind für die Internationalisierung von Gravity UI-Diensten konzipiert.

### Installation

`npm install --save @gravity-ui/i18n`

### API

#### constructor(options)

Akzeptiert ein `options`-Objekt mit einem optionalen `logger`, der für das Protokollieren von Warnungen der Bibliothek verwendet wird.

##### logger

Der Logger sollte eine explizite `log`-Methode mit folgender Signatur besitzen:

 * `message` – String der zu protokollierenden Nachricht
 * `options` – Objekt mit Protokollierungsoptionen:
   * `level` – Stufe für die Protokollnachricht, immer `'info'`
   * `logger` – Ziel für das Protokollieren der Bibliotheksnachrichten
   * `extra` – Zusätzliches Optionsobjekt mit einem einzelnen `type`-String, der immer `i18n` ist

### Verwendungsbeispiele

#### `keysets/en.json`

```json
{
  "wizard": {
    "label_error-widget-no-access": "No access to the chart"
  }
}
```

#### `keysets/ru.json`

```json
{
  "wizard": {
    "label_error-widget-no-access": "Нет доступа к чарту"
  }
}
```

#### `index.js`

```js
const ru = require('./keysets/ru.json');
const en = require('./keysets/en.json');

const {I18N} = require('@gravity-ui/i18n');

const i18n = new I18N();
i18n.registerKeysets('ru', ru);
i18n.registerKeysets('en', en);

i18n.setLang('ru');
console.log(
    i18n.i18n('wizard', 'label_error-widget-no-access')
); // -> "Нет доступа к чарту"

i18n.setLang('en');
console.log(
    i18n.i18n('wizard', 'label_error-widget-no-access')
); // -> "No access to the chart

// Keyset allows for a simpler translations retrieval
const keyset = i18n.keyset('wizard');
console.log(
    keyset('label_error-widget-no-access')
); // -> "No access to the chart"


i18n.setLang('ru');
console.log(
    keyset('label_error-widget-no-access')
); // -> "Нет доступа к чарту"

// Checking if keyset has a key
if (i18n.has('wizard', 'label_error-widget-no-access')) {
    i18n.i18n('wizard', 'label_error-widget-no-access')
}
```

### Vorlagen (Templating)

Die Bibliothek unterstützt Vorlagen. Vorlagenvariablen werden in doppelten geschweiften Klammern eingeschlossen, und die Werte werden als Schlüssel-Wert-Dictionary an die i18n-Funktion übergeben:

#### `keysets.json`

```json
{
  "label_template": "No matches found for '{{inputValue}}' in '{{folderName}}'"
}
```

#### `index.js`

```js
i18n('label_template', {inputValue: 'something', folderName: 'somewhere'});  // => No matches found for "something" in "somewhere"
```

### Pluralisierung

Pluralisierung kann für eine einfache Lokalisierung von Schlüsseln verwendet werden, die von numerischen Werten abhängen. Die aktuelle Bibliothek verwendet [CLDR Plural Rules](https://unicode-org.github.io/cldr-staging/charts/latest/supplemental/language_plural_rules.html) über die [Intl.PluralRules API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/PluralRules).

Falls die [Intl.PluralRules API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/PluralRules) im Browser nicht verfügbar ist, müssen Sie sie möglicherweise mit einem [Polyfill](https://github.com/eemeli/intl-pluralrules) ergänzen.

Es gibt 6 Pluralformen (siehe [resolvedOptions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/PluralRules/resolvedOptions)):

- zero (wird auch verwendet, wenn count = 0 ist, auch wenn die Form in der Sprache nicht unterstützt wird)
- one (Einzahl)
- two (Dual)
- few (Pauckal)
- many (wird auch für Brüche verwendet, falls sie eine separate Klasse haben)
- other (erforderliche Form für alle Sprachen – allgemeine Pluralform – wird auch verwendet, wenn die Sprache nur eine einzige Form hat)

#### Beispiel für `keysets.json` mit Plural-Schlüssel

```json
{
  "label_seconds": {
    "one": "{{count}} second is left",
    "other":"{{count}} seconds are left",
    "zero": "No time left"
  }
}
```

#### Verwendung in JS

```js
i18n('label_seconds', {count: 1});  // => 1 second
i18n('label_seconds', {count: 3});  // => 3 seconds
i18n('label_seconds', {count: 7});  // => 7 seconds
i18n('label_seconds', {count: 10}); // => 10 seconds
i18n('label_seconds', {count: 0});  // => No time left
```

#### [Veraltet] Altes Pluralisierungsformat

Das alte Format wird in v2 entfernt.

```json
{
  "label_seconds": ["{{count}} second is left", "{{count}} seconds are left", "{{count}} seconds are left", "No time left"]
}
```

Ein pluralisierter Schlüssel enthält 4 Werte, die jeweils einem `PluralForm`-Enum-Wert entsprechen. Die Enum-Werte sind: `One`, `Few`, `Many` und `None`, in dieser Reihenfolge. Der Name der Template-Variable für die Pluralisierung ist `count`.

#### [Veraltet] Benutzerdefinierte Pluralisierung

Da jede Sprache ihre eigene Art der Pluralisierung hat, bietet die Bibliothek eine Methode, um die Regeln für jede gewählte Sprache zu konfigurieren.

Die Konfigurationsfunktion akzeptiert ein Objekt mit Sprachen als Schlüsseln und Pluralisierungsfunktionen als Werten.

Eine Pluralisierungsfunktion nimmt eine Zahl und das `PluralForm`-Enum entgegen und soll einen der Enum-Werte je nach angegebener Zahl zurückgeben.

```js
const {I18N} = require('@gravity-ui/i18n');

const i18n = new I18N();

i18n.configurePluralization({
  en: (count, pluralForms) => {
    if (!count) return pluralForms.None;
    if (count === 1) return pluralForms.One;
    return pluralForms.Many;
  },
});
```

#### [Veraltet] Bereitgestellte Pluralisierungsregeln

Die zwei out-of-the-box unterstützten Sprachen sind Englisch und Russisch.

##### Englisch

Sprachschnitt: `en`.
* `One` entspricht 1 und -1.
* `Few` wird nicht verwendet.
* `Many` entspricht allen anderen Zahlen außer 0.
* `None` entspricht 0.

##### Russisch

Sprachschnitt: `ru`.
* `One` entspricht jeder Zahl, die auf 1 endet, außer ±11.
* `Few` entspricht jeder Zahl, die auf 2, 3 oder 4 endet, außer ±12, ±13 und ±14.
* `Many` entspricht allen anderen Zahlen außer 0.
* `None` entspricht 0.

##### Standard

Der englische Pluralisierungs-Regelsatz wird standardmäßig verwendet, für jede Sprache ohne konfigurierte Pluralisierungsfunktion.

### Verschachtelung

<!--GITHUB_BLOCK-->
<span style="color:red">
<!--/GITHUB_BLOCK-->

<!--LANDING_BLOCK
<span style={{color: 'red'}}>
LANDING_BLOCK-->

Maximale Verschachtelungstiefe begrenzt – nur 1 Ebene (für Glossar)
</span>

Die Verschachtelung ermöglicht es, auf andere Schlüssel in einer Übersetzung zu verweisen. Das kann nützlich sein, um Glossar-Begriffe zu erstellen.

#### Grundlegend

Schlüssel

```json
{
  "nesting1": "1 $t{nesting2}",
  "nesting2": "2",
}
```

Beispiel

```ts
i18n('nesting1'); // -> "1 2"
```

Sie können Schlüssel aus anderen Schlüssel-Sets referenzieren, indem Sie dem Schlüssel-Sets-Namen ein Präfix voranstellen: 
```json
// global/en.json
{
  "app": "App"
}

// service/en.json
{
  "app-service": "$t{global::app} service"
}
```

### Typisierung

Um die Funktion `i18nInstance.i18n` zu typisieren, folgen Sie diesen Schritten:

#### Vorbereitung

Bereiten Sie eine JSON-Schlüssel-Set-Datei vor, damit das Typisierungsverfahren Daten abrufen kann. Wo Sie Schlüssel-Sets laden, fügen Sie die Erstellung einer zusätzlichen `data.json`-Datei hinzu. Um die Dateigröße zu verringern und das Parsen in der IDE zu beschleunigen, können Sie alle Werte durch `'str'` ersetzen.

```ts
async function createFiles(keysets: Record<Lang, LangKeysets>) {
    await mkdirp(DEST_PATH);

    const createFilePromises = Object.keys(keysets).map((lang) => {
        const keysetsJSON = JSON.stringify(keysets[lang as Lang], null, 4);
        const content = umdTemplate(keysetsJSON);
        const hash = getContentHash(content);
        const filePath = path.resolve(DEST_PATH, `${lang}.${hash.slice(0, 8)}.js`);

        // <New lines>
        let typesPromise;

        if (lang === 'ru') {
            const keyset = keysets[lang as Lang];
            Object.keys(keyset).forEach((keysetName) => {
                const keyPhrases = keyset[keysetName];
                Object.keys(keyPhrases).forEach((keyName) => {
                    // mutate object!
                    keyPhrases[keyName] = 'str';
                });
            });

            const JSONForTypes = JSON.stringify(keyset, null, 4);
            typesPromise = writeFile(path.resolve(DEST_PATH, `data.json`), JSONForTypes, 'utf-8');
        }
        // </New lines>

        return Promise.all([typesPromise, writeFile(filePath, content, 'utf-8')]);
    });

    await Promise.all(createFilePromises);
}
```

#### Verbindung

In Ihren Verzeichnissen `ui/utils/i18n` (wo Sie i18n konfigurieren und es für alle Schnittstellen exportieren), importieren Sie die Typisierungsfunktion `I18NFn` mit Ihren `Keysets`. Nachdem Ihre i18n konfiguriert wurde, geben Sie die gecastete Funktion zurück

```ts
import {I18NFn} from '@gravity-ui/i18n';
// This must be a typed import!
import type Keysets from '../../../dist/public/build/i18n/data.json';

const i18nInstance = new I18N();
type TypedI18n = I18NFn<typeof Keysets>;
// ...
export const ci18n = (i18nInstance.i18n as TypedI18n).bind(i18nInstance, 'common');
export const cui18n = (i18nInstance.i18n as TypedI18n).bind(i18nInstance, 'common.units');
export const i18n = i18nInstance.i18n.bind(i18nInstance) as TypedI18n;
```

#### Zusätzliche Probleme

**Typisierungslogik**

Es gibt mehrere Typisierungs-Use-Cases:

- Aufruf einer Funktion mit Schlüsseln, die als String-Literale übergeben werden

```ts
i18n('common', 'label_subnet'); // ok
i18n('dcommon', 'label_dsubnet'); // error: Argument of type '"dcommon"' is not assignable to parameter of type ...
i18n('common', 'label_dsubnet'); // error: Argument of type '"label_dsubnet"' is not assignable to parameter of type ...
```

- Aufruf einer Funktion, bei dem Strings übergeben werden, die nicht in Literale umgewandelt werden können (wenn ts den String-Typ nicht ableiten kann, wirft es keinen Fehler)

```ts
const someUncomputebleString = `label_random-index-${Math.floor(Math.random() * 4)}`;
i18n('some_service', someUncomputebleString); // ok

for (let i = 0; i < 4; i++) {
    i18n('some_service', `label_random-index-${i}`); // ok
}
```

- Aufruf einer Funktion, bei dem Strings übergeben werden, die in Literale umgewandelt werden können

```ts
const labelColors = ['red', 'green', 'yelllow', 'white'] as const;
for (let i = 0; i < 4; i++) {
    i18n('some_service', `label_color-${labelColors[i]}`); // ok
}

const labelWrongColors = ['red', 'not-existing', 'yelllow', 'white'] as const;
for (let i = 0; i < 4; i++) {
    i18n('some_service', `label_color-${labelWrongColors[i]}`); // error: Argument of type '"not-existing"' is not assignable to parameter of type ...
}
```

**Warum die Typisierung über eine Klasse nicht unterstützt wird**

Diese Funktion kann einige i18n-Szenarien stören oder komplizieren, daher wurde sie als funktionale Erweiterung hinzugefügt. Wenn sie sich als effektiv erweist, würden wir sie wahrscheinlich in eine Klasse integrieren, um das Casten exportierter Funktionen zu vermeiden.

**Warum eingebaute Methoden fehlschlagen könnten**

Die Implementierung der Traversierung verschachtelter Strukturen und bedingter Typen unter Verwendung typisierter eingegebener Funktionsmethoden ist eine komplexe Aufgabe. Deshalb funktioniert die Typisierung nur bei direkten Funktionsaufrufen und `bind`-Aufrufen bis zum dritten Argument.

**Warum kann ich nicht direkt eine ts-Datei generieren, um Schlüsselwerte ebenfalls zu typisieren?**

Das können Sie tun, indem Sie den Ergebnistyp an I18NFn übergeben. Bei großen Dateigrößen verbraucht ts jedoch enorme Ressourcen und verlangsamt die IDE dramatisch, was bei JSON-Dateien nicht der Fall ist.

**Warum sind andere Methoden der I18N-Klasse nicht typisiert?**

Sie können typisiert werden, wir schätzen es, wenn Sie bei der Implementierung helfen. Der Grund ist, dass andere Methoden nur in 1 % der Fälle verwendet werden.