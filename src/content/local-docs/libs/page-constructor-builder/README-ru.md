# Page Constructor Builder

Мощная утилита командной строки для сборки статических страниц из YAML-конфигураций с использованием пакета [@gravity-ui/page-constructor](https://github.com/gravity-ui/page-constructor). Подробности конфигурации смотрите в [storybook page-constructor](https://preview.gravity-ui.com/page-constructor/).

## Быстрый старт

1. **Установка пакета:**

```bash
npm install @gravity-ui/page-constructor-builder
```

2. **Добавление команды сборки в package.json:**

```json
{
  "scripts": {
    "build": "page-builder build"
  }
}
```

3. **Добавление исходных файлов:**

`page-builder.config.yml`:

```yaml
input: ./pages
output: ./dist
assets: ./assets
favicon: logo.svg
theme: light
minify: true
```

`pages/index.yml`:

```yaml
meta:
  title: Hello, World
  description: A simple page constructor page

blocks:
  - type: header-block
    title: Hello, World
    description: |
      Build beautiful static pages from **YAML configurations** using the power of 
      [@gravity-ui/page-constructor](https://github.com/gravity-ui/page-constructor).
    background:
      color: '#f8f9fa'
```

4. **Сборка страниц:**

```bash
npm run build
```

5. **Открытие сгенерированных HTML-файлов в браузере:**

```bash
open dist/index.html
```

## Использование

### Команды

#### `page-builder build`

Сборка страниц из YAML-конфигураций.

```bash
page-builder build [options]
```

**Опции:**

- `-i, --input <path>`: Входная директория с YAML-файлами (по умолчанию: "./pages")
- `-o, --output <path>`: Выходная директория для собранных файлов (по умолчанию: "./dist")
- `-c, --config <path>`: Путь к файлу конфигурации (по умолчанию: "./page-builder.config.yml")
- `--css <files...>`: Пользовательские CSS-файлы для включения
- `--components <path>`: Директория с пользовательскими компонентами
- `--navigation <path>`: Файл с данными навигации
- `--assets <path>`: Директория со статическими активами для копирования
- `--theme <theme>`: Тема (light|dark) (по умолчанию: "light")
- `--base-url <url>`: Базовый URL сайта
- `--minify`: Включить минификацию
- `--source-maps`: Генерировать source maps
- `--watch`: Включить режим наблюдения

### Конфигурация

Создайте файл `page-builder.config.yml` в корне проекта:

```yaml
input: ./pages
output: ./dist
assets: ./assets
favicon: logo.svg # Файл фавикона из assets или внешний URL
theme: light
baseUrl: https://mysite.com
minify: true
sourceMaps: false # Генерировать source maps для отладки (увеличивает размер бандла)
css:
  - ./styles/main.css
  - ./styles/components.scss
components: ./components
navigation: ./navigation.yml
webpack:
  # Пользовательская конфигурация webpack
```

### Конфигурация страницы

Создайте YAML-файлы в директории страниц:

```yaml
# pages/index.yml
meta:
  title: Welcome to My Site
  description: This is the homepage of my awesome site

blocks:
  - type: header-block
    title: Welcome!
    description: This is a **header block** with markdown support
    background:
      color: '#f0f0f0'

  - type: content-block
    title: About Us
    text: |
      This is a content block with multiple lines of text.

      You can use **markdown** formatting here.

  - type: CustomBlock # Your custom component
    title: Custom Component
    content: This uses a custom component
```

### Пользовательские компоненты

Создайте React-компоненты в директории components:

```typescript
// components/CustomBlock.tsx
import React from 'react';

interface CustomBlockProps {
  title: string;
  content: string;
  className?: string;
}

export const CustomBlock: React.FC<CustomBlockProps> = ({
  title,
  content,
  className = ''
}) => {
  return (
    <div className={`custom-block ${className}`}>
      <h2>{title}</h2>
      <p>{content}</p>
    </div>
  );
};

export default CustomBlock;
```

### Пользовательские стили

Добавьте свои CSS/SCSS-файлы:

```css
/* styles/main.css */
.custom-block {
  padding: 20px;
  margin: 20px 0;
  border-radius: 8px;
  background: #f5f5f5;
  border-left: 4px solid #007acc;
}

.custom-block h2 {
  margin-top: 0;
  color: #007acc;
}
```

### Статические активы

Page Constructor Builder автоматически обрабатывает статические активы, такие как изображения, иконки и другие файлы. Настройте директорию активов в файле конфигурации:

```yaml
# page-builder.config.yml
input: ./pages
output: ./dist
assets: ./assets # Директория активов для копирования
```

**Структура директории активов:**

```
assets/
├── images/
│   ├── hero-banner.jpg
│   └── about-photo.png
├── icons/
│   ├── logo.svg
│   └── social-icons.svg
└── documents/
    └── brochure.pdf
```

**Использование активов в страницах:**

```yaml
# pages/index.yml
blocks:
  - type: header-block
    title: Welcome
    description: Check out our amazing content
    background:
      image: assets/images/hero-banner.jpg

  - type: media-block
    title: About Us
    media:
      - type: image
        src: assets/images/about-photo.png
        alt: Our team photo
```

### Фавикон

Page Constructor Builder поддерживает добавление фавиконов к статическим страницам. Вы можете указать локальный файл из директории assets или внешний URL.

#### Конфигурация

Добавьте опцию `favicon` в файл конфигурации:

```yaml
# page-builder.config.yml
favicon: logo.svg # Локальный файл из директории assets
# или
favicon: https://cdn.example.com/favicon.ico # Внешний URL
```

#### Локальные файлы фавикона

Для локальных файлов фавикона сборщик:

- Автоматически обнаружит файл в директории assets
- Скопирует его в выходную директорию
- Сгенерирует правильные HTML-теги `<link>` с корректными MIME-типами

**Поддерживаемые форматы файлов:**

- **SVG** (рекомендуется) - `image/svg+xml`
- **ICO** (классический) - `image/x-icon`
- **PNG** (современный) - `image/png`
- **JPG/JPEG** (приемлемо) - `image/jpeg`
- **GIF** (анимация) - `image/gif`

**Примеры:**

```yaml
# page-builder.config.yml
favicon: logo.svg                    # File in assets/ directory
favicon: icons/favicon.ico           # File in assets/icons/ subdirectory
favicon: ./custom/path/favicon.png   # Custom path relative to project
favicon: /absolute/path/favicon.ico  # Absolute path
```

#### Внешние URL-адреса фавиконок

Вы также можете использовать внешние URL-адреса фавиконок из CDN или других доменов:

```yaml
# page-builder.config.yml
favicon: https://cdn.example.com/favicon.ico
favicon: https://mysite.com/assets/logo.svg
```

#### Генерируемый HTML

Конструктор автоматически генерирует соответствующие HTML-теги на основе типа фавиконки:

```html
<!-- Для SVG-фаиконок -->
<link rel="icon" type="image/svg+xml" href="assets/logo.svg" />

<!-- Для ICO-фаиконок (включая поддержку устаревших браузеров) -->
<link rel="icon" type="image/x-icon" href="assets/favicon.ico" />
<link rel="shortcut icon" href="assets/favicon.ico" />

<!-- Для внешних URL -->
<link rel="icon" href="https://example.com/favicon.ico" />
```

### Навигация

Конструктор страниц поддерживает глобальную конфигурацию навигации, которая отображается на всех страницах. Навигация настраивается через отдельный YAML-файл.

#### Конфигурация навигации

Создайте файл `navigation.yml` в корне вашего проекта (или укажите пользовательский путь в вашей конфигурации):

```yaml
# navigation.yml
logo:
  text: Your Site Name
  url: 'index.html'
  icon: 'assets/logo.svg'

header:
  leftItems:
    - text: Home
      url: 'index.html'
      type: 'link'
    - text: About
      url: 'about.html'
      type: 'link'
    - text: Documentation
      url: 'https://external-site.com/docs'
      type: 'link'
  rightItems:
    - text: GitHub
      url: 'https://github.com/your-repo'
      type: 'link'
    - text: Contact
      url: 'contact.html'
      type: 'link'

footer:
  leftItems:
    - text: Privacy Policy
      url: 'privacy.html'
      type: 'link'
  rightItems:
    - text: © 2024 Your Company
      type: 'text'
```

#### Переопределение навигации для конкретной страницы

Вы можете переопределить навигацию для конкретных страниц, добавив раздел `navigation` непосредственно в YAML страницы:

```yaml
# pages/special-page.yml
meta:
  title: Special Page

navigation:
  logo:
    text: Special Site
    url: 'index.html'
  header:
    leftItems:
      - text: Back to Main
        url: 'index.html'
        type: 'link'

blocks:
  - type: header-block
    title: This page has custom navigation
```