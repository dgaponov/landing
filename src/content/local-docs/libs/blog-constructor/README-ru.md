# @gravity-ui/blog-constructor &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/blog-constructor)](https://www.npmjs.com/package/@gravity-ui/blog-constructor) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/blog-constructor/ci.yml?branch=main&label=CI)](https://github.com/gravity-ui/blog-constructor/actions/workflows/ci.yml?query=branch:main) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/blog-constructor/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/blog-constructor/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/blog-constructor/)

## Установка

```shell
npm install @gravity-ui/blog-constructor
```

## Blog-constructor

`Blog-constructor` — это библиотека, основанная на [Page-constructor](https://github.com/gravity-ui/page-constructor), для создания веб-страниц в формате блога. Blog-constructor использует проп [`custom`](https://github.com/gravity-ui/page-constructor#custom-blocks) из page-constructor, чтобы добавить необходимые для блога компоненты.

### Документация - [storybook](https://preview.gravity-ui.com/blog-constructor/)

### Начало работы

Blog-constructor предоставляет как клиентские, так и серверные компоненты для импорта. Страницы блога импортируются как React-компоненты. Чтобы всё работало корректно, оберните их в `BlogConstructorProvider`:

```jsx
import {BlogPage, BlogConstructorProvider} from '@gravity-ui/blog-constructor';

// Главная страница блога
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

// Страница поста
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

Документация о [providerProps](./src/constructor/README.md).

Также blog-constructor включает серверные компоненты, которые помогут преобразовать ваши данные, если это потребуется:

```jsx
import {
  transformPost,
  sanitizeMeta,
  createReadableContent,
  transformPageContent,
} from '@gravity-ui/blog-constructor/server';
```

`Blog-constructor` — это библиотека на базе `uikit`, и мы используем экземпляр `i18n` из uikit. Для настройки интернационализации достаточно использовать `configure` из uikit:

```typescript
import {configure} from '@gravity-ui/uikit';

configure({
  lang: 'ru',
});
```

## Разработка

```bash
npm ci
npm run dev
```