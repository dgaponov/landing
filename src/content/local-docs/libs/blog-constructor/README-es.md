# @gravity-ui/blog-constructor &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/blog-constructor)](https://www.npmjs.com/package/@gravity-ui/blog-constructor) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/blog-constructor/ci.yml?branch=main&label=CI)](https://github.com/gravity-ui/blog-constructor/actions/workflows/ci.yml?query=branch:main) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/blog-constructor/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/blog-constructor/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/blog-constructor/)

## Instalación

```shell
npm install @gravity-ui/blog-constructor
```

## Blog-constructor

`Blog-constructor` es una biblioteca basada en la biblioteca [Page-constructor](https://github.com/gravity-ui/page-constructor) para crear páginas web en formato de blog. Blog-constructor utiliza la prop [`custom`](https://github.com/gravity-ui/page-constructor#custom-blocks) de page-constructor para agregar los componentes necesarios para el blog.

### Documentación - [storybook](https://preview.gravity-ui.com/blog-constructor/)

### Primeros pasos

Blog-constructor incluye componentes tanto del lado del cliente como del servidor para importar. La página de blog se importa como un componente de React. Para asegurarte de que funcione correctamente, envuélvela en `BlogConstructorProvider`:

```jsx
import {BlogPage, BlogConstructorProvider} from '@gravity-ui/blog-constructor';

// Página principal del blog
<BlogConstructorProvider {...providerProps}>
    <BlogPage
        content={content}
        posts={posts}
        tags={tags}
        getPosts={handleGetPosts}
        settings={settings}
    />
</BlogConstructorProvider>

---

import {BlogPostPage, BlogConstructorProvider} from '@gravity-ui/blog-constructor';

// Página de publicación
<BlogConstructorProvider {...providerProps}>
    <BlogPostPage
        content={content}
        post={post}
        suggestedPosts={suggestedPosts}
        settings={settings}
        shareOptions={shareOptions}
    />
</BlogConstructorProvider>

```

Documentación sobre [providerProps](./src/constructor/README.md).

Además, blog-constructor incluye componentes del servidor para ayudarte a transformar tus datos si lo necesitas:

```jsx
import {
  transformPost,
  sanitizeMeta,
  createReadableContent,
  transformPageContent,
} from '@gravity-ui/blog-constructor/server';
```

La biblioteca `blog-constructor` está basada en `uikit`, y utilizamos una instancia de `i18n` de uikit. Para configurar la internacionalización, solo necesitas usar `configure` de uikit:

```typescript
import {configure} from '@gravity-ui/uikit';

configure({
  lang: 'ru',
});
```

## Desarrollo

```bash
npm ci
npm run dev
```