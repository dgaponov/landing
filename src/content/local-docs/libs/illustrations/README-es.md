# @gravity-ui/illustrations · [![npm package](https://img.shields.io/npm/v/@gravity-ui/illustrations)](https://www.npmjs.com/package/@gravity-ui/illustrations) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/illustrations/.github/workflows/ci.yml?label=CI&logo=github)](https://github.com/gravity-ui/illustrations/actions/workflows/ci.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/illustrations/)

## Instalación

```shell
npm install --save-dev @gravity-ui/illustrations
```

## Uso

### React

#### Preparación

Configura el tema de las ilustraciones. Ejecuta cualquiera de los siguientes pasos:

##### Definiendo css-tokens con tu propia paleta de colores

Define los siguientes css-tokens en tu app:

```scss
--gil-color-object-base: rgb(255, 190, 92);
--gil-color-object-accent-heavy: rgb(211, 101, 7);
--gil-color-object-hightlight: rgb(255, 216, 157);
--gil-color-shadow-over-object: rgb(211, 158, 80);
--gil-color-background-lines: rgb(140, 140, 140);
--gil-color-background-shapes: rgb(242, 242, 242);
--gil-color-object-accent-light: rgb(255, 255, 255);
--gil-color-object-danger: rgb(255, 0, 61);
```

##### Usando mixins con el tema gravity predeterminado en scss

Usa los siguientes mixins para estilizar las ilustraciones en diferentes temas:

```scss
@import '@gravity-ui/illustrations/styles/theme.scss';

.g-root {
  &_theme_light {
    @include g-illustrations-colors-light;
  }

  &_theme_light-hc {
    @include g-illustrations-colors-light-hc;
  }

  &_theme_dark {
    @include g-illustrations-colors-dark;
  }

  &_theme_dark-hc {
    @include g-illustrations-colors-dark-hc;
  }
}
```

##### Alternativa para proyectos con tema gravity preinstalado

Alternativamente, si `@gravity-ui/uikit` ya está instalado en el proyecto y se usa el tema predeterminado, puedes simplemente importar `styles.scss` en el archivo raíz de estilos de tu proyecto:

```scss
// definición existente de estilos gravity
import '@gravity-ui/uikit/styles/styles.css';
// solo agrega una importación más abajo
import '@gravity-ui/illustrations/styles/styles.scss';
```

#### Uso de componentes

```js
import NotFound from '@gravity-ui/illustrations/NotFound';
```

o

```js
import {NotFound} from '@gravity-ui/illustrations';
```

### SVG

> Podrías necesitar un loader apropiado para esto

```js
import notFound from '@gravity-ui/illustrations/svgs/not-found-light.svg';
```

### Desarrollo

Para actualizar las ilustraciones según un nuevo diseño, cambia el contenido de los archivos SVG en el tema light (`<raíz-del-repositorio>/svgs/<nombre-de-ilustración>-light.svg`) y luego ejecuta el comando:

```shell
npm run generate
```