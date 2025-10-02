# @gravity-ui/i18n &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/i18n)](https://www.npmjs.com/package/@gravity-ui/i18n) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/i18n/.github/workflows/ci.yml?branch=main&label=CI&logo=github)](https://github.com/gravity-ui/i18n/actions/workflows/ci.yml?query=branch:main)

## Утилиты I18N

Утилиты в пакете I18N предназначены для интернационализации сервисов Gravity UI.

### Установка

`npm install --save @gravity-ui/i18n`

### API

#### constructor(options)

Принимает объект `options` с опциональным полем `logger`, который будет использоваться для логирования предупреждений библиотеки.

##### logger

Логгер должен иметь явный метод `log` с следующей сигнатурой:

 * `message` — строка с сообщением, которое будет залогировано
 * `options` — объект опций логирования:
   * `level` — уровень для сообщения, всегда `'info'`
   * `logger` — куда логировать сообщения библиотеки
   * `extra` — дополнительный объект опций с единственной строкой `type`, которая всегда равна `i18n`

### Примеры использования

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

// Keyset позволяет проще получать переводы
const keyset = i18n.keyset('wizard');
console.log(
    keyset('label_error-widget-no-access')
); // -> "No access to the chart"


i18n.setLang('ru');
console.log(
    keyset('label_error-widget-no-access')
); // -> "Нет доступа к чарту"

// Проверка наличия ключа в keyset
if (i18n.has('wizard', 'label_error-widget-no-access')) {
    i18n.i18n('wizard', 'label_error-widget-no-access')
}
```

### Шаблонизация

Библиотека поддерживает шаблонизацию. Переменные шаблона заключаются в двойные фигурные скобки, а значения передаются в функцию i18n в виде словаря ключ-значение:

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

### Плюрализация

Плюрализация позволяет легко локализовать ключи, зависящие от числовых значений. Текущая версия библиотеки использует [правила плюрализации CLDR](https://unicode-org.github.io/cldr-staging/charts/latest/supplemental/language_plural_rules.html) через [API Intl.PluralRules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/PluralRules).

Если API недоступен в браузере, вам может потребоваться [полифилл](https://github.com/eemeli/intl-pluralrules) для [Intl.PluralRules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/PluralRules).

Существует 6 форм плюрализации (см. [resolvedOptions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/PluralRules/resolvedOptions)):

- zero (также используется при count = 0, даже если эта форма не поддерживается в языке)
- one (единственное число)
- two (двойственное число)
- few (множественное число малого количества)
- many (множественное число большого количества; также используется для дробей, если они имеют отдельный класс)
- other (обязательная форма для всех языков — общая форма множественного числа; также используется, если в языке только одна форма)

#### Пример `keysets.json` с ключом плюрализации

```json
{
  "label_seconds": {
    "one": "{{count}} second is left",
    "other":"{{count}} seconds are left",
    "zero": "No time left"
  }
}
```

#### Использование в JS

```js
i18n('label_seconds', {count: 1});  // => 1 second
i18n('label_seconds', {count: 3});  // => 3 seconds
i18n('label_seconds', {count: 7});  // => 7 seconds
i18n('label_seconds', {count: 10}); // => 10 seconds
i18n('label_seconds', {count: 0});  // => No time left
```

#### [Устаревший] Старый формат плюралов

Старый формат будет удалён в версии 2.

```json
{
  "label_seconds": ["{{count}} second is left", "{{count}} seconds are left", "{{count}} seconds are left", "No time left"]
}
```

Ключ с плюрализацией содержит 4 значения, каждое из которых соответствует значению перечисления `PluralForm`. Значения перечисления: `One`, `Few`, `Many` и `None` соответственно. Имя переменной шаблона для плюрализации — `count`.

#### [Устаревший] Пользовательская плюрализация

Поскольку каждый язык имеет свой способ плюрализации, библиотека предоставляет метод для настройки правил для любого выбранного языка.

Функция конфигурации принимает объект, где ключами являются языки, а значениями — функции плюрализации.

Функция плюрализации принимает число и перечисление `PluralForm`, и должна возвращать одно из значений перечисления в зависимости от предоставленного числа.

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

#### [Устаревший] Предоставляемые наборы правил плюрализации

Из коробки поддерживаются два языка: английский и русский.

##### Английский

Ключ языка: `en`.
* `One` соответствует 1 и -1.
* `Few` не используется.
* `Many` соответствует любому другому числу, кроме 0.
* `None` соответствует 0.

##### Русский

Ключ языка: `ru`.
* `One` соответствует любому числу, заканчивающемуся на 1, кроме ±11.
* `Few` соответствует любому числу, заканчивающемуся на 2, 3 или 4, кроме ±12, ±13 и ±14.
* `Many` соответствует любому другому числу, кроме 0.
* `None` соответствует 0.

##### По умолчанию

По умолчанию используется набор правил на английском языке для любого языка, у которого не настроена функция плюрализации.

### Вложенность

<!--GITHUB_BLOCK-->
<span style="color:red">
<!--/GITHUB_BLOCK-->

<!--LANDING_BLOCK
<span style={{color: 'red'}}>
LANDING_BLOCK-->

Максимальная глубина вложенности ограничена — только 1 уровень (для глоссария)
</span>

Вложенность позволяет ссылаться на другие ключи в переводе. Это может быть полезно для создания терминов глоссария.

#### Базовый пример

ключи

```json
{
  "nesting1": "1 $t{nesting2}",
  "nesting2": "2",
}
```

пример

```ts
i18n('nesting1'); // -> "1 2"
```

Вы можете ссылаться на ключи из других наборов, добавляя перед ними имя набора: 
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

### Типизация

Чтобы типизировать функцию `i18nInstance.i18n`, следуйте этим шагам:

#### Подготовка

Подготовьте файл JSON с набором ключей, чтобы процедура типизации могла получить данные. Там, где вы загружаете наборы ключей, добавьте создание дополнительного файла `data.json`. Чтобы уменьшить размер файла и ускорить парсинг в IDE, вы можете заменить все значения на `'str'`.

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

#### Подключение

В директориях `ui/utils/i18n` (где вы настраиваете i18n и экспортируете его для использования во всех интерфейсах) импортируйте функцию типизации `I18NFn` с вашими `Keysets`. После настройки i18n верните приведённую функцию

```ts
import {I18NFn} from '@gravity-ui/i18n';
// Это должен быть типизированный импорт!
import type Keysets from '../../../dist/public/build/i18n/data.json';

const i18nInstance = new I18N();
type TypedI18n = I18NFn<typeof Keysets>;
// ...
export const ci18n = (i18nInstance.i18n as TypedI18n).bind(i18nInstance, 'common');
export const cui18n = (i18nInstance.i18n as TypedI18n).bind(i18nInstance, 'common.units');
export const i18n = i18nInstance.i18n.bind(i18nInstance) as TypedI18n;
```

#### Дополнительные вопросы

**Логика типизации**

Существует несколько случаев использования типизации:

- Вызов функции с ключами, переданными как строковые литералы

```ts
i18n('common', 'label_subnet'); // ok
i18n('dcommon', 'label_dsubnet'); // error: Argument of type '"dcommon"' is not assignable to parameter of type ...
i18n('common', 'label_dsubnet'); // error: Argument of type '"label_dsubnet"' is not assignable to parameter of type ...
```

- Вызов функции с передачей строк, которые нельзя преобразовать в литералы (если ts не может вывести тип строки, ошибка не возникает)

```ts
const someUncomputebleString = `label_random-index-${Math.floor(Math.random() * 4)}`;
i18n('some_service', someUncomputebleString); // ok

for (let i = 0; i < 4; i++) {
    i18n('some_service', `label_random-index-${i}`); // ok
}
```

- Вызов функции с передачей строк, которые можно преобразовать в литералы

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

**Почему типизация через класс не поддерживается**

Эта функция может нарушить или усложнить некоторые сценарии i18n, поэтому она добавлена как функциональное расширение. Если она окажется эффективной, мы, вероятно, добавим её в класс, чтобы избежать приведения экспортируемых функций.

**Почему встроенные методы могут не работать**

Реализация обхода вложенных структур и условных типов с использованием типизированных встроенных методов функций — достаточно сложная задача. Поэтому типизация работает только при прямом вызове функции и вызове `bind` до третьего аргумента.

**Почему нельзя просто сгенерировать ts-файл для приведения типов ключей?**

Вы можете это сделать, передав результирующий тип в I18NFn. Однако при больших размерах файлов ts начинает потреблять огромные ресурсы, сильно замедляя IDE, но с JSON-файлом этого не происходит.

**Почему другие методы класса I18N не типизированы?**

Их можно типизировать, мы будем признательны за помощь в реализации. Дело в том, что другие методы используются в 1% случаев.