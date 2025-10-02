# @gravity-ui/blog-constructor &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/blog-constructor)](https://www.npmjs.com/package/@gravity-ui/blog-constructor) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/blog-constructor/ci.yml?branch=main&label=CI)](https://github.com/gravity-ui/blog-constructor/actions/workflows/ci.yml?query=branch:main) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/blog-constructor/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/blog-constructor/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/blog-constructor/)

## 설치

```shell
npm install @gravity-ui/blog-constructor
```

## Blog-constructor

`Blog-constructor`는 [Page-constructor](https://github.com/gravity-ui/page-constructor) 라이브러리를 기반으로 한 라이브러리로, 블로그 형식의 웹 페이지를 생성하는 데 사용됩니다. Blog-constructor는 page-constructor의 [`custom`](https://github.com/gravity-ui/page-constructor#custom-blocks) 속성을 활용하여 블로그에 필요한 컴포넌트를 추가합니다.

### 문서 - [storybook](https://preview.gravity-ui.com/blog-constructor/)

### 시작하기

Blog-constructor는 클라이언트 컴포넌트와 서버 컴포넌트를 모두 제공합니다. 블로그 페이지는 React 컴포넌트로 가져옵니다. 제대로 작동하도록 하려면 `BlogConstructorProvider`로 감싸야 합니다:

```jsx
import {BlogPage, BlogConstructorProvider} from '@gravity-ui/blog-constructor';

// 메인 블로그 페이지
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

// 포스트 페이지
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

[providerProps](./src/constructor/README.md)에 대한 문서.

또한 blog-constructor는 데이터 변환을 돕기 위해 서버 컴포넌트도 제공합니다:

```jsx
import {
  transformPost,
  sanitizeMeta,
  createReadableContent,
  transformPageContent,
} from '@gravity-ui/blog-constructor/server';
```

`blog-constructor`는 `uikit-based` 라이브러리이며, uikit의 `i18n` 인스턴스를 사용합니다. 국제화를 설정하려면 uikit의 `configure`를 사용하면 됩니다:

```typescript
import {configure} from '@gravity-ui/uikit';

configure({
  lang: 'ru',
});
```

## 개발

```bash
npm ci
npm run dev
```