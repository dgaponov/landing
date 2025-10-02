## Kurze Beschreibung

Der Zweck der `DFDialog`-Komponente ist es, die Erstellung von Formularen zu erleichtern. Intern wird react-final-form verwendet.
Sie unterstützt mehrere vordefinierte Feldtypen, aber der Benutzer kann sie erweitern, indem er neue mit `registerDialogControl` registriert.

### Steuerelemente

- [Basis-Steuerelemente](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-00-base-controls)
  - `plain` - statischer Text
  - `text` - editierbarer Text
  - `multi-text` - benutzerdefinierte Liste von Strings
  - `checkbox` - Checkbox
  - `tumbler` - Umschalter
  - `radio` - Radiobutton
  - `editable-list` - Liste editierbarer und entfernbarer Strings
  - `multi-editable-list` - mehrstufige Liste editierbarer und entfernbarer Strings
  - `text area` - Textbereich
  - `select` - Auswahlfeld
  - `block` - ermöglicht das Hinzufügen eines ReactNode
- [Registrierung benutzerdefinierter Steuerelemente](https://preview.yandexcloud.dev/dialog-fields/?path=/story/tutorials-custom-control-registration)

### Funktionen

- Inline- und Modal-Ansicht
- [Ein-Tab-Formular](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-01-one-tab) und [mehrstufige Tab-Formulare](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-02-several-tab--horizontal-tabs)
- [Vertikale/Horizontale Tabs](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-02-several-tab)
- [Versteckte Felder und Tabs](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-04-visibility-condition)
- [Verknüpfte Felder basierend auf Werten](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-05-extras-and-linked-fields)
- [Feldspezifische Validierung](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-06-field-validators)
- [Formularspezifische Validierung](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-07-form-validation)
- [Virtualisierte Tabs](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-08-virtualized-tabs)
- [Klonbare Tabs](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-08-cloneable-tabs-)
- [Gruppierten Felder](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-03-sections)

## Installation

```bash
$ npm install @gravity-ui/dialog-fields
# Verwenden Sie die erforderliche Version von react/react-dom, falls Sie sie noch nicht installiert haben
$ npm install @gravity-ui/dialog-fields react@18 react-dom@18
```

Je nach verwendeten Package-Manager müssen Sie die `peerDependencies` möglicherweise manuell installieren.

## Verwendung

```ts
import {DFDialog, FormApi} from '@gravity-ui/dialog-fields';

interface FormValues {
  firstName: string;
  lastName: string;
}

function MyForm() {
  return (
    <DFDialog<FormValues>
      visible={true}
      headerProps={{
        title: 'My form',
      }}
      onAdd={(form) => {
        console.log(form.getState().values);
        return Promise.resolve();
      }}
      fields={[
        {
          name: 'firstName',
          type: 'text',
          caption: 'First name',
          tooltip: 'Description for first name field',
        },
        {
          name: 'lastName',
          type: 'text',
          caption: 'LastName',
          tooltip: 'Description for last name field',
        },
      ]}
    />
  );
}
```

Weitere Beispiele finden Sie in [storybook](https://preview.yandexcloud.dev/dialog-fields).