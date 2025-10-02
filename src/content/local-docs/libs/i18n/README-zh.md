# @gravity-ui/i18n &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/i18n)](https://www.npmjs.com/package/@gravity-ui/i18n) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/i18n/.github/workflows/ci.yml?branch=main&label=CI&logo=github)](https://github.com/gravity-ui/i18n/actions/workflows/ci.yml?query=branch:main)

## I18N 实用工具

I18N 包中的实用工具旨在为 Gravity UI 服务提供国际化支持。

### 安装

`npm install --save @gravity-ui/i18n`

### API

#### constructor(options)

接受一个 `options` 对象，其中可选的 `logger` 用于记录库的警告日志。

##### logger

Logger 应该有一个明确的 `log` 方法，其签名如下：

 * `message` - 要记录的消息字符串
 * `options` - 日志选项对象：
   * `level` - 日志消息的级别，始终为 `'info'`
   * `logger` - 记录库消息的位置
   * `extra` - 附加选项对象，只有一个 `type` 字符串，始终为 `i18n`

### 使用示例

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

### 模板化

该库支持模板化。模板变量用双大括号括起来，值作为键值对字典传递给 i18n 函数：

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

### 复数形式

复数形式可用于轻松本地化依赖数值的键。当前库通过 [Intl.PluralRules API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/PluralRules) 使用 [CLDR Plural Rules](https://unicode-org.github.io/cldr-staging/charts/latest/supplemental/language_plural_rules.html)。

如果浏览器中不可用，您可能需要为 [Intl.PluralRules API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/PluralRules) 添加 [polyfill](https://github.com/eemeli/intl-pluralrules)。

有 6 种复数形式（参见 [resolvedOptions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/PluralRules/resolvedOptions)）：

- zero（即使语言不支持该形式，当 count = 0 时也会使用）
- one（单数）
- two（双数）
- few（少量）
- many（多量，也用于分数如果它们有单独的类别）
- other（所有语言的必需形式 — 通用复数形式 — 如果语言只有一个形式，也会使用）

#### 带有复数键的 `keysets.json` 示例

```json
{
  "label_seconds": {
    "one": "{{count}} second is left",
    "other":"{{count}} seconds are left",
    "zero": "No time left"
  }
}
```

#### JS 中的用法

```js
i18n('label_seconds', {count: 1});  // => 1 second
i18n('label_seconds', {count: 3});  // => 3 seconds
i18n('label_seconds', {count: 7});  // => 7 seconds
i18n('label_seconds', {count: 10}); // => 10 seconds
i18n('label_seconds', {count: 0});  // => No time left
```

#### [已弃用] 旧复数格式

旧格式将在 v2 中移除。

```json
{
  "label_seconds": ["{{count}} second is left", "{{count}} seconds are left", "{{count}} seconds are left", "No time left"]
}
```

复数化键包含 4 个值，每个对应一个 `PluralForm` 枚举值。枚举值分别是：`One`、`Few`、`Many` 和 `None`。复数化的模板变量名为 `count`。

#### [已弃用] 自定义复数化

由于每种语言都有自己的复数化方式，该库提供了一种方法来为任何选择的语言配置规则。

配置函数接受一个对象，其中键为语言，值为复数化函数。

复数化函数接受一个数字和 `PluralForm` 枚举，并根据提供的数字返回枚举值之一。

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

#### [已弃用] 提供的复数化规则集

开箱即用的两种语言是英语和俄语。

##### 英语

语言键：`en`。
* `One` 对应 1 和 -1。
* `Few` 未使用。
* `Many` 对应除 0 外的任何其他数字。
* `None` 对应 0。

##### 俄语

语言键：`ru`。
* `One` 对应以 1 结尾的任何数字，除 ±11 外。
* `Few` 对应以 2、3 或 4 结尾的任何数字，除 ±12、±13 和 ±14 外。
* `Many` 对应除 0 外的任何其他数字。
* `None` 对应 0。

##### 默认

默认使用英文规则集，对于任何没有配置复数化函数的语言。

### 嵌套

<!--GITHUB_BLOCK-->
<span style="color:red">
<!--/GITHUB_BLOCK-->

<!--LANDING_BLOCK
<span style={{color: 'red'}}>
LANDING_BLOCK-->

最大嵌套深度限制 - 仅支持 1 级（用于术语表）
</span>

嵌套允许你在翻译中引用其他键。这对于构建术语表非常有用。

#### 基础用法

键

```json
{
  "nesting1": "1 $t{nesting2}",
  "nesting2": "2",
}
```

示例

```ts
i18n('nesting1'); // -> "1 2"
```

你可以通过在键名前添加 keysetName 来引用其他 keyset 中的键：
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

### 类型化

要为 `i18nInstance.i18n` 函数添加类型，请按照以下步骤操作：

#### 准备工作

准备一个 JSON keyset 文件，以便类型化过程能够获取数据。在你从哪里获取 keyset 的地方，添加创建额外的 `data.json` 文件的逻辑。为了减少文件大小并加速 IDE 解析，你可以将所有值替换为 `'str'`。

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

#### 连接

在你的 `ui/utils/i18n` 目录（那里配置 i18n 并导出以供所有接口使用）中，导入类型化函数 `I18NFn` 并带上你的 `Keysets`。在 i18n 配置完成后，返回类型转换后的函数。

```ts
import {I18NFn} from '@gravity-ui/i18n';
// 这必须是一个类型化的导入！
import type Keysets from '../../../dist/public/build/i18n/data.json';

const i18nInstance = new I18N();
type TypedI18n = I18NFn<typeof Keysets>;
// ...
export const ci18n = (i18nInstance.i18n as TypedI18n).bind(i18nInstance, 'common');
export const cui18n = (i18nInstance.i18n as TypedI18n).bind(i18nInstance, 'common.units');
export const i18n = i18nInstance.i18n.bind(i18nInstance) as TypedI18n;
```

#### 其他问题

**类型化逻辑**

有几种类型化用例：

- 使用字符串字面量作为键调用函数

```ts
i18n('common', 'label_subnet'); // ok
i18n('dcommon', 'label_dsubnet'); // error: Argument of type '"dcommon"' is not assignable to parameter of type ...
i18n('common', 'label_dsubnet'); // error: Argument of type '"label_dsubnet"' is not assignable to parameter of type ...
```

- 调用函数时传入无法转换为字面量的字符串（如果 ts 无法推导字符串类型，它不会抛出错误）

```ts
const someUncomputebleString = `label_random-index-${Math.floor(Math.random() * 4)}`;
i18n('some_service', someUncomputebleString); // ok

for (let i = 0; i < 4; i++) {
    i18n('some_service', `label_random-index-${i}`); // ok
}
```

- 调用函数时传入可以转换为字面量的字符串

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

**为什么不支持通过类进行类型化**

此功能可能会破坏或复杂化某些 i18n 场景，因此它被添加为函数式扩展。如果证明有效，我们可能会将其添加到类中，以避免导出函数的类型转换。

**为什么内置方法可能失效**

实现嵌套结构遍历和使用类型化内置函数方法的条件类型是一个足够复杂任务。因此，类型化仅在使用直接函数调用和 `bind` 调用（最多到第三个参数）时有效。

**为什么不能直接生成 ts 文件来类型化键值？**

你可以将结果类型传递给 I18NFn 来实现这一点。然而，对于大文件大小，ts 会消耗大量资源，导致 IDE 严重变慢，但使用 JSON 文件则不会出现这种情况。

**为什么 I18N 类的其他方法没有类型化？**

它们可以被类型化，我们很欣赏你帮忙实现。问题是其他方法仅在 1% 的情况下使用。