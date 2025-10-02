## Kurze Beschreibung

Der Zweck der `DFDialog`-Komponente ist es, die Erstellung von Formularen zu erleichtern. Sie verwendet intern react-final-form.
Sie unterstützt mehrere vordefinierte Feldtypen, aber Benutzer können sie erweitern, indem sie neue mit `registerDialogControl` registrieren.

### Steuerelemente

- [Basis-Steuerelemente](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-00-base-controls)
  - `plain` - statischer Text
  - `text` - editierbarer Text
  - `multi-text` - benutzerdefinierte Zeichenfolgenarray
  - `checkbox` - Kontrollkästchen
  - `tumbler` - Schalter
  - `radio` - Optionsfeld
  - `editable-list` - Liste entfernbarer Zeichenfolgen
  - `multi-editable-list` - Mehrfachliste entfernbarer Zeichenfolgen
  - `text area` - Textbereich
  - `select` - Auswahlfeld
  - `block` - ermöglicht das Hinzufügen eines ReactNode
- [Registrierung benutzerdefinierter Steuerelemente](https://preview.yandexcloud.dev/dialog-fields/?path=/story/tutorials-custom-control-registration)

### Funktionen

- Ansicht vor Ort und als Modalfenster
- [Ein Tab](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-01-one-tab) und [mehrere Tabs für Formulare](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-02-several-tab--horizontal-tabs)
- [Vertikale/Horizontale Tabs](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-02-several-tab)
- [Versteckte Felder und Tabs](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-04-visibility-condition)
- [Verknüpfte Felder nach Werten](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-05-extras-and-linked-fields)
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

Je nach Paketmanager müssen Sie möglicherweise die `peerDependencies` manuell installieren.

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