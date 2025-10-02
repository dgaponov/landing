# @gravity-ui/i18n &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/i18n)](https://www.npmjs.com/package/@gravity-ui/i18n) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/i18n/.github/workflows/ci

##### Predeterminado

El conjunto de reglas en inglés se utiliza por defecto para cualquier idioma sin una función de pluralización configurada.

### Anidamiento

<!--GITHUB_BLOCK-->
<span style="color:red">
<!--/GITHUB_BLOCK-->

<!--LANDING_BLOCK
<span style={{color: 'red'}}>
LANDING_BLOCK-->

La profundidad máxima de anidamiento está limitada: solo 1 nivel (para el glosario)
</span>

El anidamiento te permite hacer referencia a otras claves en una traducción. Puede ser útil para construir términos de glosario.

#### Básico

claves

```json
{
  "nesting1": "1 $t{nesting2}",
  "nesting2": "2",
}
```

ejemplo

```ts
i18n('nesting1'); // -> "1 2"
```

Puedes hacer referencia a claves de otros conjuntos de claves preponiendo el nombre del conjunto: 
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

### Tipado

Para tipar la función `i18nInstance.i18n`, sigue estos pasos:

#### Preparación

Prepara un archivo de conjunto de claves JSON para que el procedimiento de tipado pueda obtener los datos. Donde obtengas los conjuntos de claves, agrega la creación de un archivo adicional `data.json`. Para reducir el tamaño del archivo y acelerar el análisis del IDE, puedes reemplazar todos los valores por `'str'`.

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

#### Conexión

En tus directorios `ui/utils/i18n` (donde configuras i18n y lo exportas para que sea utilizado por todas las interfaces), importa la función de tipado `I18NFn` con tus `Keysets`. Después de que i18n haya sido configurado, devuelve la función casteada

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

#### Problemas adicionales

**Lógica de tipado**

Hay varios casos de uso para el tipado:

- Llamar a una función con claves pasadas como literales de cadena

```ts
i18n('common', 'label_subnet'); // ok
i18n('dcommon', 'label_dsubnet'); // error: Argument of type '"dcommon"' is not assignable to parameter of type ...
i18n('common', 'label_dsubnet'); // error: Argument of type '"label_dsubnet"' is not assignable to parameter of type ...
```

- Llamar a una función, pasándole cadenas que no se pueden convertir en literales (si ts no puede derivar el tipo de cadena, no genera un error)

```ts
const someUncomputebleString = `label_random-index-${Math.floor(Math.random() * 4)}`;
i18n('some_service', someUncomputebleString); // ok

for (let i = 0; i < 4; i++) {
    i18n('some_service', `label_random-index-${i}`); // ok
}
```

- Llamar a una función, pasándole cadenas que se pueden convertir en literales

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

**Por qué no se soporta el tipado a través de una clase**

Esta función puede romper o complicar algunos escenarios de i18n, por lo que se agregó como una extensión funcional. Si resulta efectiva, probablemente la agreguemos a una clase para evitar el casteo de las funciones exportadas.

**Por qué los métodos integrados podrían fallar**

Implementar el recorrido de estructuras anidadas y tipos condicionales utilizando métodos de funciones integradas tipadas es una tarea lo suficientemente compleja. Por eso, el tipado funciona solo al usar una llamada directa a la función y una llamada a `bind` hasta el tercer argumento.

**Por qué no puedo generar un archivo ts directamente para castear también los valores de las claves?**

Puedes hacerlo pasando el tipo de resultado a I18NFn. Sin embargo, con tamaños de archivo grandes, ts comienza a consumir cantidades enormes de recursos, ralentizando drásticamente el IDE, pero con un archivo JSON esto no ocurre.

**Por qué otros métodos de la clase I18N no han sido tipados?**

Se pueden tipar, apreciamos si nos ayudas a implementarlo. El caso es que otros métodos se usan en el 1% de los casos.