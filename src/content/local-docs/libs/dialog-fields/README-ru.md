## Краткое описание

Компонент `DFDialog` упрощает создание форм и использует react-final-form под капотом.  
Он поддерживает несколько предопределённых типов полей, но вы можете расширить его, регистрируя новые с помощью `registerDialogControl`.

### Элементы управления

- [Базовые элементы управления](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-00-base-controls)
  - `plain` — статический текст
  - `text` — редактируемый текст
  - `multi-text` — массив строк, определяемый пользователем
  - `checkbox` — флажок
  - `tumbler` — переключатель
  - `radio` — радиокнопка
  - `editable-list` — список удаляемых строк
  - `multi-editable-list` — многоуровневый список удаляемых строк
  - `text area` — текстовая область
  - `select` — выпадающий список
  - `block` — позволяет добавить ReactNode
- [Регистрация пользовательских элементов управления](https://preview.yandexcloud.dev/dialog-fields/?path=/story/tutorials-custom-control-registration)

### Возможности

- Встроенное и модальное отображение
- [Одна вкладка](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-01-one-tab) и [формы с несколькими вкладками](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-02-several-tab--horizontal-tabs)
- [Вертикальные/горизонтальные вкладки](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-02-several-tab)
- [Скрытые поля и вкладки](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-04-visibility-condition)
- [Поля, связанные по значениям](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-05-extras-and-linked-fields)
- [Валидация на уровне поля](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-06-field-validators)
- [Валидация на уровне формы](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-07-form-validation)
- [Виртуализированные вкладки](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-08-virtualized-tabs)
- [Клонируемые вкладки](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-08-cloneable-tabs-)
- [Группированные поля](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-03-sections)

## Установка

```bash
$ npm install @gravity-ui/dialog-fields
# Установите необходимые версии react/react-dom, если они ещё не установлены
$ npm install @gravity-ui/dialog-fields react@18 react-dom@18
```

В зависимости от вашего пакетного менеджера может потребоваться установить `peerDependencies` вручную.

## Использование

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

Дополнительные примеры смотрите в [Storybook](https://preview.yandexcloud.dev/dialog-fields).