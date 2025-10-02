# @gravity-ui/blog-constructor &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/blog-constructor)](https://www.npmjs.com/package/@gravity-ui/blog-constructor) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/blog-constructor/ci.yml?branch=main&label=CI)](https://github.com/gravity-ui/blog-constructor/actions/workflows/ci.yml?query=branch:main) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/blog-constructor/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/blog-constructor/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/blog-constructor/)

## Installation

```shell
npm install @gravity-ui/blog-constructor
```

## Blog-constructor

`Blog-constructor` ist eine Bibliothek, die auf der [Page-constructor](https://github.com/gravity-ui/page-constructor)-Bibliothek basiert und für die Erstellung von Webseiten im Blog-Format entwickelt wurde. Blog-constructor nutzt die [`custom`](https://github.com/gravity-ui/page-constructor#custom-blocks)-Eigenschaft aus page-constructor, um die für den Blog benötigten Komponenten hinzuzufügen.

### Dokumentation - [storybook](https://preview.gravity-ui.com/blog-constructor/)

### Erste Schritte

Blog-constructor umfasst sowohl Client-Komponenten als auch Server-Komponenten für den Import. Die Blog-Seiten werden als React-Komponenten importiert. Um sicherzustellen, dass alles reibungslos läuft, umschließen Sie sie mit `BlogConstructorProvider`:

```jsx
import {BlogPage, BlogConstructorProvider} from '@gravity-ui/blog-constructor';

// Hauptseite des Blogs
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

// Beitragsseite
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

Dokumentation zu [providerProps](./src/constructor/README.md).

Blog-constructor bietet außerdem Server-Komponenten, die Ihnen helfen, Ihre Daten bei Bedarf zu transformieren:

```jsx
import {
  transformPost,
  sanitizeMeta,
  createReadableContent,
  transformPageContent,
} from '@gravity-ui/blog-constructor/server';
```

`Blog-constructor` ist eine `uikit-basierte` Bibliothek, und wir verwenden eine Instanz von `i18n` aus uikit. Für die Einrichtung der Internationalisierung müssen Sie lediglich `configure` aus uikit verwenden:

```typescript
import {configure} from '@gravity-ui/uikit';

configure({
  lang: 'ru',
});
```

## Entwicklung

```bash
npm ci
npm run dev
```