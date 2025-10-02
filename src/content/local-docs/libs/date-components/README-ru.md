# @gravity-ui/date-components &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/date-components)](https://www.npmjs.com/package/@gravity-ui/date-components) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/date-components/.github/workflows/ci.yml?label=CI&logo=github)](https://github.com/gravity-ui/date-components/actions/workflows/ci.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/date-components/)

## Установка

```shell
npm install react react-dom @gravity-ui/uikit @gravity-ui/date-components @gravity-ui/date-utils
```

## Использование

```jsx
import {createRoot} from 'react-dom/client';
import {DatePicker} from '@gravity-ui/date-components';
import {ThemeProvider} from '@gravity-ui/uikit';

import '@gravity-ui/uikit/styles/styles.css';

function App() {
  return (
    <ThemeProvider>
      <h1>Выбор даты</h1>
      <form>
        <label forHtml="date-picker">Дата:</label>
        <DatePicker id="date-picker" name="date" />
      </form>
    </ThemeProvider>
  );
}

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

### Локализация

```jsx
import {settings} from '@gravity-ui/date-utils';

// Загрузите локали дат, которые будут использоваться в приложении.
settings.loadLocale('ru');

function App() {
  return (
    // Установите язык для использования с компонентами.
    <ThemeProvider lang="ru">
      <h1>Выбор даты</h1>
      <form>
        <label forHtml="date-picker">Дата:</label>
        <DatePicker id="date-picker" name="date" />
      </form>
    </ThemeProvider>
  );
}
```

Если приложение поддерживает переключение языка, предзагрузите все поддерживаемые локали при первом запуске приложения или загрузите локали перед переключением языка:

```jsx
// Предзагрузка локалей
settings.loadLocale('ru');
settings.loadLocale('nl');

const root = createRoot(document.getElementById('root'));
root.render(<App />);

// Или загрузка локалей по требованию.

function App() {
  const [lang, setLang] = React.useState('en');

  const handleLangChange = (newLang) => {
    settings.loadLocale(newLang).then(() => {
      setLang(newLang);
    });
  };

  return <ThemeProvider lang={lang}>...</ThemeProvider>;
}
```

Компоненты имеют переводы на английский и русский языки. Чтобы добавить переводы на другие языки, используйте `addLanguageKeysets` из `@gravity-ui/uikit`:

```ts
import {addLanguageKeysets} from '@gravity-ui/uikit/i18n';
import type {Keysets, PartialKeysets} from '@gravity-ui/date-components';

// Используйте тип Keyset, чтобы указать переводы для всех доступных компонентов
addLanguageKeysets<Keysets>(lang, {...});

// Или используйте тип PartialKeysets, чтобы указать только необходимые
addLanguageKeysets<PartialKeysets>(lang, {...});

// Чтобы указать переводы для некоторых компонентов
addLanguageKeysets<Pick<Keysets, 'g-date-calendar' | 'g-date-date-field' | 'g-date-date-picker'>>(lang, {...});
```

## Разработка

Чтобы запустить сервер разработки со Storybook, выполните следующую команду:

```shell
npm start
```