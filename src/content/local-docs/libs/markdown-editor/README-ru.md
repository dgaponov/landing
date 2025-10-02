![Редактор Markdown](https://github.com/user-attachments/assets/0b4e5f65-54cf-475f-9c68-557a4e9edb46)

# @gravity-ui/markdown-editor &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/markdown-editor)](https://www.npmjs.com/package/@gravity-ui/markdown-editor) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/markdown-editor/ci.yml?branch=main&label=CI)](https://github.com/gravity-ui/markdown-editor/actions/workflows/ci.yml?query=branch:main) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/markdown-editor/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/markdown-editor/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/md-editor/)

## Редактор Markdown в режимах WYSIWYG и разметки

MarkdownEditor — это мощный инструмент для работы с Markdown, который сочетает в себе режимы WYSIWYG и разметки. Это значит, что вы можете создавать и редактировать контент в удобном визуальном режиме, а также иметь полный контроль над разметкой.

### 🔧 Основные возможности

- Поддержка базового синтаксиса Markdown и [YFM](https://ydocs.tech).
- Расширяемость благодаря использованию движков ProseMirror и CodeMirror.
- Возможность работы в режимах WYSIWYG и разметки для максимальной гибкости.

## Установка

```shell
npm install @gravity-ui/markdown-editor
```

### Необходимые зависимости

Обратите внимание, что для начала использования пакета в вашем проекте также должны быть установлены следующие зависимости: `@diplodoc/transform`, `react`, `react-dom`, `@gravity-ui/uikit`, `@gravity-ui/components` и некоторые другие. Точную информацию смотрите в разделе `peerDependencies` файла `package.json`.

## Начало работы

Редактор Markdown поставляется в виде React-хука для создания экземпляра редактора и компонента для рендеринга представления.\
Для настройки стилей и темы обратитесь к [документации UIKit](https://github.com/gravity-ui/uikit?tab=readme-ov-file#styles).

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
Подробнее:
- [Как подключить редактор в Create React App](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-getting-started-create-react-app--docs)
- [Как добавить предпросмотр для режима разметки](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-getting-started-preview--docs)
- [Как добавить расширение для HTML](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-extensions-html-block--docs)
- [Как добавить расширение для LaTeX](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-extensions-latex-extension--docs)
- [Как добавить расширение для Mermaid](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-extensions-mermaid-extension--docs)
- [Как написать расширение](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-develop-extension-creation--docs)
- [Как добавить расширение для GPT](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-extensions-gpt--docs)
- [Как добавить расширение с привязкой текста в Markdown](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-develop-extension-with-popup--docs)

### Разработка
Чтобы запустить dev-версию Storybook:

```shell
npm start
```

### i18n

Для настройки интернационализации достаточно использовать функцию `configure`:

```typescript
import {configure} from '@gravity-ui/markdown-editor';

configure({
  lang: 'ru',
});
```

Не забудьте вызвать `configure()` из [UIKit](https://github.com/gravity-ui/uikit?tab=readme-ov-file#i18n) и других UI-библиотек.

### Вклад в проект

- [Правила для контрибьюторов](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-contributing--docs)