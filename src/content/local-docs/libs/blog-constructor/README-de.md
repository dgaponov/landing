# @gravity-ui/blog-constructor &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/blog-constructor)](https://www.npmjs.com/package/@gravity-ui/blog-constructor) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/blog-constructor/ci.yml?branch=main&label=CI)](https://github.com/gravity-ui/blog-constructor/actions/workflows/ci.yml?query=branch:main) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/blog-constructor/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/blog-constructor/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/blog-constructor/)

## Installation

```shell
npm install @gravity-ui/blog-constructor
```

## Blog-Constructor

`Blog-constructor` ist eine Bibliothek, die auf der [Page-constructor](https://github.com/gravity-ui/page-constructor)-Bibliothek basiert und Webseiten im Blog-Format erstellt. Blog-constructor nutzt die [`custom`](https://github.com/gravity-ui/page-constructor#custom-blocks)-Prop aus Page-constructor, um die für den Blog erforderlichen Komponenten hinzuzufügen.

### Dokumentation - [Storybook](https://preview.gravity-ui.com/blog-constructor/)

### Erste Schritte

Der Blog-Constructor umfasst sowohl Client-Komponenten als auch Server-Komponenten, die importiert werden können. Die Blog-Seiten werden als React-Komponente importiert. Um sicherzustellen, dass alles reibungslos läuft, umschließen Sie sie mit `BlogConstructorProvider`:

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

Der Blog-Constructor bietet zudem Server-Komponenten, die Ihnen bei der Transformation Ihrer Daten helfen können, falls Sie diese benötigen:

```jsx
import {
  transformPost,
  sanitizeMeta,
  createReadableContent,
  transformPageContent,
} from '@gravity-ui/blog-constructor/server';
```

Der `blog-constructor` ist eine auf `uikit` basierende Bibliothek, und wir verwenden eine Instanz von `i18n` aus UIKit. Für die Einrichtung der Internationalisierung müssen Sie lediglich `configure` aus UIKit verwenden:

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