# @gravity-ui/i18n &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/i18n)](https://www.npmjs.com/package/@gravity-ui/i18n) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/i18n/.github/workflows/ci.yml?branch=main&label=CI&logo=github)](https://github.com/gravity-ui/i18n/actions/workflows/ci.yml?query=branch:main)

## I18N 유틸리티

I18N 패키지의 유틸리티는 Gravity UI 서비스의 국제화(internationalization)를 위해 설계되었습니다.

### 설치

`npm install --save @gravity-ui/i18n`

### API

#### constructor(options)

`options` 객체를 받아들이며, 선택적인 `logger`가 라이브러리 경고를 로깅하는 데 사용됩니다.

##### logger

로거는 다음 시그니처를 가진 명시적인 `log` 메서드를 가져야 합니다:

 * `message` - 로깅될 메시지의 문자열
 * `options` - 로깅 옵션 객체:
   * `level` - 로깅 메시지의 레벨, 항상 `'info'`
   * `logger` - 라이브러리 메시지를 로깅할 위치
   * `extra` - 추가 옵션 객체로, 단일 `type` 문자열을 포함하며 이는 항상 `i18n`입니다

### 사용 예시

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

### 템플릿

이 라이브러리는 템플릿을 지원합니다. 템플릿 변수는 이중 중괄호로 둘러싸여 있으며, 값은 키-값 사전으로 i18n 함수에 전달됩니다:

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

### 복수형(Pluralization)

복수형은 숫자 값에 따라 달라지는 키의 로컬라이제이션을 쉽게 처리하는 데 사용할 수 있습니다. 현재 라이브러리는 [CLDR Plural Rules](https://unicode-org.github.io/cldr-staging/charts/latest/supplemental/language_plural_rules.html)를 [Intl.PluralRules API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/PluralRules)를 통해 사용합니다.

브라우저에서 [Intl.PluralRules API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/PluralRules)가 사용 불가능한 경우 [polyfill](https://github.com/eemeli/intl-pluralrules)을 적용해야 할 수 있습니다.

6가지 복수 형태가 있습니다(자세한 내용은 [resolvedOptions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/PluralRules/resolvedOptions) 참조):

- zero (숫자가 0일 때도 사용되며, 언어에서 지원되지 않더라도 적용됨)
- one (단수)
- two (이중)
- few (소수)
- many (분수에 대해 별도의 클래스가 있는 경우에도 사용됨)
- other (모든 언어에 필수적인 형태 — 일반 복수 형태 — 언어가 단일 형태만 가질 때도 사용됨)

#### 복수 키가 포함된 `keysets.json` 예시

```json
{
  "label_seconds": {
    "one": "{{count}} second is left",
    "other":"{{count}} seconds are left",
    "zero": "No time left"
  }
}
```

#### JS에서의 사용

```js
i18n('label_seconds', {count: 1});  // => 1 second
i18n('label_seconds', {count: 3});  // => 3 seconds
i18n('label_seconds', {count: 7});  // => 7 seconds
i18n('label_seconds', {count: 10}); // => 10 seconds
i18n('label_seconds', {count: 0});  // => No time left
```

#### [사용 중단됨] 이전 복수 형식

이전 형식은 v2에서 제거될 예정입니다.

```json
{
  "label_seconds": ["{{count}} second is left", "{{count}} seconds are left", "{{count}} seconds are left", "No time left"]
}
```

복수화된 키는 4개의 값을 포함하며, 각 값은 `PluralForm` 열거형 값에 해당합니다. 열거형 값은 각각 `One`, `Few`, `Many`, `None`입니다. 복수화에 대한 템플릿 변수 이름은 `count`입니다.

#### [사용 중단됨] 사용자 지정 복수화

각 언어는 고유한 복수화 방식을 가지므로, 라이브러리는 선택한 언어에 대한 규칙을 구성할 수 있는 메서드를 제공합니다.

구성 함수는 언어를 키로 하고 복수화 함수를 값으로 하는 객체를 받아들입니다.

복수화 함수는 숫자와 `PluralForm` 열거형을 받아들이며, 제공된 숫자에 따라 열거형 값 중 하나를 반환해야 합니다.

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

#### [사용 중단됨] 제공된 복수화 규칙 세트

기본으로 지원되는 두 언어는 영어와 러시아어입니다.

##### 영어

언어 키: `en`.
* `One`은 1과 -1에 해당합니다.
* `Few`는 사용되지 않습니다.
* `Many`는 0을 제외한 다른 모든 숫자에 해당합니다.
* `None`은 0에 해당합니다.

##### 러시아어

언어 키: `ru`.
* `One`은 ±11을 제외한 1로 끝나는 모든 숫자에 해당합니다.
* `Few`은 ±12, ±13, ±14를 제외한 2, 3, 4로 끝나는 모든 숫자에 해당합니다.
* `Many`는 0을 제외한 다른 모든 숫자에 해당합니다.
* `None`은 0에 해당합니다.

##### 기본값

기본적으로 영어 규칙 세트가 사용되며, 복수형 처리 함수가 설정되지 않은 모든 언어에 적용됩니다.

### 중첩

<!--GITHUB_BLOCK-->
<span style="color:red">
<!--/GITHUB_BLOCK-->

<!--LANDING_BLOCK
<span style={{color: 'red'}}>
LANDING_BLOCK-->

최대 중첩 깊이가 제한되어 있습니다 - 용어집(glossary)을 위한 1단계만 지원됩니다.
</span>

중첩을 통해 번역에서 다른 키를 참조할 수 있습니다. 용어집 용어를 구성하는 데 유용할 수 있습니다.

#### 기본

키

```json
{
  "nesting1": "1 $t{nesting2}",
  "nesting2": "2",
}
```

샘플

```ts
i18n('nesting1'); // -> "1 2"
```

다른 키 세트의 키를 참조하려면 키 세트 이름을 앞에 붙여 사용하세요:
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

### 타입 지정

`i18nInstance.i18n` 함수에 타입을 지정하려면 다음 단계를 따르세요:

#### 준비

타입 지정 절차가 데이터를 가져올 수 있도록 JSON 키 세트 파일을 준비하세요. 키 세트를 가져오는 위치에 추가 `data.json` 파일 생성을 포함하세요. 파일 크기를 줄이고 IDE 파싱 속도를 높이기 위해 모든 값을 `'str'`로 대체할 수 있습니다.

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

#### 연결

`ui/utils/i18n` 디렉토리(여기서 i18n을 구성하고 모든 인터페이스에서 사용할 수 있도록 내보내는 곳)에서 `Keysets`와 함께 타입 지정 함수 `I18NFn`을 가져오세요. i18n 구성이 완료된 후 캐스팅된 함수를 반환하세요.

```ts
import {I18NFn} from '@gravity-ui/i18n';
// 이는 타입이 지정된 가져오기여야 합니다!
import type Keysets from '../../../dist/public/build/i18n/data.json';

const i18nInstance = new I18N();
type TypedI18n = I18NFn<typeof Keysets>;
// ...
export const ci18n = (i18nInstance.i18n as TypedI18n).bind(i18nInstance, 'common');
export const cui18n = (i18nInstance.i18n as TypedI18n).bind(i18nInstance, 'common.units');
export const i18n = i18nInstance.i18n.bind(i18nInstance) as TypedI18n;
```

#### 추가 문제

**타입 지정 로직**

타입 지정에는 여러 사용 사례가 있습니다:

- 문자열 리터럴로 전달된 키로 함수 호출

```ts
i18n('common', 'label_subnet'); // ok
i18n('dcommon', 'label_dsubnet'); // 오류: '"dcommon"' 타입의 인수가 ... 매개변수에 할당할 수 없습니다.
i18n('common', 'label_dsubnet'); // 오류: '"label_dsubnet"' 타입의 인수가 ... 매개변수에 할당할 수 없습니다.
```

- 리터럴로 변환할 수 없는 문자열을 함수에 전달하여 호출 (ts가 문자열 타입을 유추할 수 없으면 오류를 발생시키지 않습니다)

```ts
const someUncomputebleString = `label_random-index-${Math.floor(Math.random() * 4)}`;
i18n('some_service', someUncomputebleString); // ok

for (let i = 0; i < 4; i++) {
    i18n('some_service', `label_random-index-${i}`); // ok
}
```

- 리터럴로 변환할 수 있는 문자열을 함수에 전달하여 호출

```ts
const labelColors = ['red', 'green', 'yelllow', 'white'] as const;
for (let i = 0; i < 4; i++) {
    i18n('some_service', `label_color-${labelColors[i]}`); // ok
}

const labelWrongColors = ['red', 'not-existing', 'yelllow', 'white'] as const;
for (let i = 0; i < 4; i++) {
    i18n('some_service', `label_color-${labelWrongColors[i]}`); // 오류: '"not-existing"' 타입의 인수가 ... 매개변수에 할당할 수 없습니다.
}
```

**클래스를 통한 타입 지정이 지원되지 않는 이유**

이 함수는 일부 i18n 시나리오를 깨뜨리거나 복잡하게 만들 수 있으므로 기능 확장으로 추가되었습니다. 효과적임이 입증되면 내보낸 함수의 캐스팅을 피하기 위해 클래스에 추가할 가능성이 큽니다.

**내장 메서드가 실패할 수 있는 이유**

중첩 구조 탐색과 조건부 타입을 사용한 타입 지정 내장 함수 메서드 구현은 상당히 복잡한 작업입니다. 따라서 직접 함수 호출과 세 번째 인수까지의 `bind` 호출에서만 타입 지정이 작동합니다.

**키 값 타입 캐스팅을 위해 ts 파일을 직접 생성할 수 없는 이유**

`I18NFn`에 결과 타입을 전달하여 그렇게 할 수 있습니다. 그러나 파일 크기가 클 경우 ts가 엄청난 리소스를 소비하며 IDE를 극도로 느리게 만듭니다. 하지만 JSON 파일의 경우 그렇지 않습니다.

**I18N 클래스의 다른 메서드가 타입 지정되지 않은 이유**

타입 지정할 수 있으며, 구현에 도움을 주시면 감사하겠습니다. 다른 메서드는 1%의 경우에만 사용되기 때문입니다.