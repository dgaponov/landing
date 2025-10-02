![Markdown Editor](https://github.com/user-attachments/assets/0b4e5f65-54cf-475f-9c68-557a4e9edb46)

# @gravity-ui/markdown-editor &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/markdown-editor)](https://www.npmjs.com/package/@gravity-ui/markdown-editor) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/markdown-editor/ci.yml?branch=main&label=CI)](https://github.com/gravity-ui/markdown-editor/actions/workflows/ci.yml?query=branch:main) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/markdown-editor/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/markdown-editor/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/md-editor/)

## Éditeur Markdown WYSIWYG et markup

MarkdownEditor est un outil puissant pour travailler avec Markdown, qui combine les modes WYSIWYG et Markup. Cela signifie que vous pouvez créer et modifier du contenu dans un mode visuel pratique, tout en ayant un contrôle total sur le markup.

### 🔧 Fonctionnalités principales

- Support de la syntaxe Markdown de base et de [YFM](https://ydocs.tech).
- Extensibilité grâce à l'utilisation des moteurs ProseMirror et CodeMirror.
- Possibilité de travailler en modes WYSIWYG et Markup pour une flexibilité maximale.

## Installation

```shell
npm install @gravity-ui/markdown-editor
```

### Dépendances requises

Notez que pour commencer à utiliser le package, votre projet doit également avoir les éléments suivants installés : `@diplodoc/transform`, `react`, `react-dom`, `@gravity-ui/uikit`, `@gravity-ui/components` et d'autres. Consultez la section `peerDependencies` du fichier `package.json` pour des informations précises.

## Démarrage rapide

L'éditeur Markdown est fourni sous forme de hook React pour créer une instance d'éditeur et un composant pour afficher la vue.\
Pour configurer le style et le thème, consultez la [documentation UIKit](https://github.com/gravity-ui/uikit?tab=readme-ov-file#styles).

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
En savoir plus :
- [Comment connecter l'éditeur dans Create React App](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-getting-started-create-react-app--docs)
- [Comment ajouter un aperçu pour le mode markup](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-getting-started-preview--docs)
- [Comment ajouter l'extension HTML](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-extensions-html-block--docs)
- [Comment ajouter l'extension Latex](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-extensions-latex-extension--docs)
- [Comment ajouter l'extension Mermaid](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-extensions-mermaid-extension--docs)
- [Comment écrire une extension](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-develop-extension-creation--docs)
- [Comment ajouter l'extension GPT](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-extensions-gpt--docs)
- [Comment ajouter une extension de liaison de texte dans Markdown](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-develop-extension-with-popup--docs)

### Développement
Pour démarrer le Storybook de développement

```shell
npm start
```


### i18n

Pour configurer l'internationalisation, il suffit d'utiliser `configure` :

```typescript
import {configure} from '@gravity-ui/markdown-editor';

configure({
  lang: 'ru',
});
```

N'oubliez pas d'appeler `configure()` depuis [UIKit](https://github.com/gravity-ui/uikit?tab=readme-ov-file#i18n) et d'autres bibliothèques d'interface utilisateur.

### Contribution

- [Directives pour les contributeurs](https://preview.gravity-ui.com/md-editor/?path=/docs/docs-contributing--docs)