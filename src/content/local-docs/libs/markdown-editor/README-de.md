![Markdown Editor](https://github.com/user-attachments/assets/0b4e5f65-54cf-475f-9c68-557a4e9edb46)

# @gravity-ui/markdown-editor &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/markdown-editor)](https://www.npmjs.com/package/@gravity-ui/markdown-editor) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/markdown-editor/ci.yml?branch=main&label=CI)](https://github.com/gravity-ui/markdown-editor/actions/workflows/ci.yml?query=branch:main) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/markdown-editor/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/markdown-editor/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/md-editor/)

## Markdown-WYSIWYG- und Markup-Editor

MarkdownEditor ist ein leistungsstarkes Werkzeug zur Arbeit mit Markdown, das WYSIWYG- und Markup-Modi kombiniert. Das bedeutet, dass Sie Inhalte in einem bequemen visuellen Modus erstellen und bearbeiten können und gleichzeitig volle Kontrolle über das Markup haben.

### 🔧 Hauptfunktionen

- Unterstützung für die grundlegende Markdown- und [YFM](https://ydocs.tech)-Syntax.
- Erweiterbarkeit durch die Verwendung der ProseMirror- und CodeMirror-Engines.
- Die Möglichkeit, in WYSIWYG- und Markup-Modi zu arbeiten, für maximale Flexibilität.

## Installation

```shell
npm install @gravity-ui/markdown-editor
```

### Erforderliche Abhängigkeiten

Bitte beachten Sie, dass für die Nutzung des Pakets in Ihrem Projekt auch folgende Pakete installiert sein müssen: `@diplodoc/transform`, `react`, `react-dom`, `@gravity-ui/uikit`, `@gravity-ui/components` und einige andere. Schauen Sie im Abschnitt `peerDependencies` der `package.json` nach für genaue Informationen.

## Erste Schritte

Der Markdown-Editor wird als React-Hook bereitgestellt, um eine Instanz des Editors zu erstellen, und als Komponente zur Darstellung der Ansicht.  
Um das Styling und das Theme einzurichten, siehe [UIKit-Dokumentation](https://github.com/gravity-ui/uikit?tab=readme-ov-file#styles).

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
Mehr lesen:
- [Wie man den Editor in Create React App verbindet](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-getting-started-create-react-app--docs)
- [Wie man eine Vorschau für den Markup-Modus hinzufügt](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-getting-started-preview--docs)
- [Wie man die HTML-Erweiterung hinzufügt](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-extensions-html-block--docs)
- [Wie man die Latex-Erweiterung hinzufügt](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-extensions-latex-extension--docs)
- [Wie man die Mermaid-Erweiterung hinzufügt](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-extensions-mermaid-extension--docs)
- [Wie man eine Erweiterung schreibt](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-develop-extension-creation--docs)
- [Wie man die GPT-Erweiterung hinzufügt](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-extensions-gpt--docs)
- [Wie man eine Textbindungserweiterung in Markdown hinzufügt](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-develop-extension-with-popup--docs)

### Entwicklung
Um den Dev-Storybook zu starten:

```shell
npm start
```

### i18n

Um die Internationalisierung einzurichten, müssen Sie nur `configure` verwenden:

```typescript
import {configure} from '@gravity-ui/markdown-editor';

configure({
  lang: 'ru',
});
```

Vergessen Sie nicht, `configure()` aus [UIKit](https://github.com/gravity-ui/uikit?tab=readme-ov-file#i18n) und anderen UI-Bibliotheken aufzurufen.

### Mitwirkung

- [Richtlinien für Mitwirkende](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-contributing--docs)