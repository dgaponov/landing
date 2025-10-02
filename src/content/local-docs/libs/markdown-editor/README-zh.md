![Markdown Editor](https://github.com/user-attachments/assets/0b4e5f65-54cf-475f-9c68-557a4e9edb46)

# @gravity-ui/markdown-editor &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/markdown-editor)](https://www.npmjs.com/package/@gravity-ui/markdown-editor) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/markdown-editor/ci.yml?branch=main&label=CI)](https://github.com/gravity-ui/markdown-editor/actions/workflows/ci.yml?query=branch:main) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/markdown-editor/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/markdown-editor/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/md-editor/)

## Markdown WYSIWYG 和标记编辑器

MarkdownEditor 是一个强大的 Markdown 处理工具，它结合了 WYSIWYG（所见即所得）和标记模式。这意味着您可以以便捷的可视化模式创建和编辑内容，同时还能完全掌控标记代码。

### 🔧 主要特性

- 支持基本的 Markdown 和 [YFM](https://ydocs.tech) 语法。
- 通过 ProseMirror 和 CodeMirror 引擎实现可扩展性。
- 支持 WYSIWYG 和标记模式，以提供最大灵活性。

## 安装

```shell
npm install @gravity-ui/markdown-editor
```

### 必需依赖

请注意，要开始使用该包，您的项目还需要安装以下内容：`@diplodoc/transform`、`react`、`react-dom`、`@gravity-ui/uikit`、`@gravity-ui/components` 等。请查看 `package.json` 中的 `peerDependencies` 部分以获取准确信息。

## 快速开始

Markdown 编辑器以 React Hook 的形式提供，用于创建编辑器实例，以及一个用于渲染视图的组件。\
要设置样式和主题，请参阅 [UIKit 文档](https://github.com/gravity-ui/uikit?tab=readme-ov-file#styles)。

```tsx
import React from 'react';
import {useMarkdownEditor, MarkdownEditorView} from '@gravity-ui/markdown-editor';

function Editor({onSubmit}) {
  const editor = useMarkdownEditor({allowHTML: false});

  React.useEffect(() => {
    function submitHandler() {
      // Serialize current content to markdown markup
      const value = editor.getValue();
      onSubmit(value);
    }

    editor.on('submit', submitHandler);
    return () => {
      editor.off('submit', submitHandler);
    };
  }, [onSubmit]);

  return <MarkdownEditorView stickyToolbar autofocus editor={editor} />;
}
```

阅读更多：
- [如何在 Create React App 中连接编辑器](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-getting-started-create-react-app--docs)
- [如何为标记模式添加预览](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-getting-started-preview--docs)
- [如何添加 HTML 扩展](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-extensions-html-block--docs)
- [如何添加 Latex 扩展](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-extensions-latex-extension--docs)
- [如何添加 Mermaid 扩展](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-extensions-mermaid-extension--docs)
- [如何编写扩展](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-develop-extension-creation--docs)
- [如何添加 GPT 扩展](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-extensions-gpt--docs)
- [如何在 Markdown 中添加文本绑定扩展](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-develop-extension-with-popup--docs)

### 开发

启动开发 Storybook

```shell
npm start
```

### i18n

要设置国际化，只需使用 `configure`：

```typescript
import {configure} from '@gravity-ui/markdown-editor';

configure({
  lang: 'ru',
});
```

不要忘记从 [UIKit](https://github.com/gravity-ui/uikit?tab=readme-ov-file#i18n) 和其他 UI 库中调用 `configure()`。

### 贡献

- [贡献指南](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-contributing--docs)