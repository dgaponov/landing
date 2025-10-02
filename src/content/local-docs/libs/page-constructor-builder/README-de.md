# Page Constructor Builder

Ein leistungsstarkes Kommandozeilen-Tool zum Erstellen statischer Seiten aus YAML-Konfigurationen mit dem Paket [@gravity-ui/page-constructor](https://github.com/gravity-ui/page-constructor). Siehe [page-constructor storybook](https://preview.gravity-ui.com/page-constructor/) für Details zur Konfiguration.

## Schneller Einstieg

1. **Paket installieren:**

```bash
npm install @gravity-ui/page-constructor-builder
```

2. **Build-Befehl zu package.json hinzufügen:**

```json
{
  "scripts": {
    "build": "page-builder build"
  }
}
```

3. **Quelldateien hinzufügen:**

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

4. **Ihre Seiten bauen:**

```bash
npm run build
```

5. **Die generierten HTML-Dateien in Ihrem Browser öffnen:**

```bash
open dist/index.html
```

## Verwendung

### Befehle

#### `page-builder build`

Erstellt Seiten aus YAML-Konfigurationen.

```bash
page-builder build [options]
```

**Optionen:**

- `-i, --input <path>`: Eingabeverzeichnis mit YAML-Dateien (Standard: "./pages")
- `-o, --output <path>`: Ausgabeverzeichnis für die gebauten Dateien (Standard: "./dist")
- `-c, --config <path>`: Pfad zur Konfigurationsdatei (Standard: "./page-builder.config.yml")
- `--css <files...>`: Benutzerdefinierte CSS-Dateien zum Einbinden
- `--components <path>`: Verzeichnis für benutzerdefinierte Komponenten
- `--navigation <path>`: Datei mit Navigationsdaten
- `--assets <path>`: Verzeichnis für statische Assets zum Kopieren
- `--theme <theme>`: Thema (light|dark) (Standard: "light")
- `--base-url <url>`: Basis-URL für die Website
- `--minify`: Minifizierung aktivieren
- `--source-maps`: Source Maps generieren
- `--watch`: Watch-Modus aktivieren

### Konfiguration

Erstellen Sie eine `page-builder.config.yml`-Datei im Stammverzeichnis Ihres Projekts:

```yaml
input: ./pages
output: ./dist
assets: ./assets
favicon: logo.svg # Favicon-Datei aus Assets oder externe URL
theme: light
baseUrl: https://mysite.com
minify: true
sourceMaps: false # Source Maps für Debugging generieren (erhöht die Bundle-Größe)
css:
  - ./styles/main.css
  - ./styles/components.scss
components: ./components
navigation: ./navigation.yml
webpack:
  # Benutzerdefinierte Webpack-Konfiguration
```

### Seitenkonfiguration

Erstellen Sie YAML-Dateien in Ihrem Seitenverzeichnis:

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

### Benutzerdefinierte Komponenten

Erstellen Sie React-Komponenten in Ihrem Komponentenverzeichnis:

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

### Benutzerdefinierte Stile

Fügen Sie Ihre benutzerdefinierten CSS/SCSS-Dateien hinzu:

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

### Statische Assets

Der Page-Constructor-Builder behandelt statische Assets wie Bilder, Icons und andere Dateien automatisch. Konfigurieren Sie das Assets-Verzeichnis in Ihrer Konfigurationsdatei:

```yaml
# page-builder.config.yml
input: ./pages
output: ./dist
assets: ./assets # Assets-Verzeichnis zum Kopieren
```

**Struktur des Assets-Verzeichnisses:**

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

**Assets in Ihren Seiten verwenden:**

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

### Favicon

Der Page-Constructor-Builder unterstützt das Hinzufügen von Favicons zu Ihren statischen Seiten. Sie können entweder eine lokale Datei aus Ihrem Assets-Verzeichnis oder eine externe URL angeben.

#### Konfiguration

Fügen Sie die `favicon`-Option zu Ihrer Konfigurationsdatei hinzu:

```yaml
# page-builder.config.yml
favicon: logo.svg # Lokale Datei aus dem Assets-Verzeichnis
# oder
favicon: https://cdn.example.com/favicon.ico # Externe URL
```

#### Lokale Favicon-Dateien

Für lokale Favicon-Dateien übernimmt der Builder folgendes:

- Automatische Erkennung der Datei in Ihrem Assets-Verzeichnis
- Kopieren in das Ausgabeverzeichnis
- Generieren korrekter HTML-`<link>`-Tags mit den richtigen MIME-Typen

**Unterstützte Dateiformate:**

- **SVG** (empfohlen) - `image/svg+xml`
- **ICO** (klassisch) - `image/x-icon`
- **PNG** (modern) - `image/png`
- **JPG/JPEG** (akzeptabel) - `image/jpeg`
- **GIF** (animiert) - `image/gif`

**Beispiele:**

```yaml
# page-builder.config.yml
favicon: logo.svg                    # File in assets/ directory
favicon: icons/favicon.ico           # File in assets/icons/ subdirectory
favicon: ./custom/path/favicon.png   # Custom path relative to project
favicon: /absolute/path/favicon.ico  # Absolute path
```

#### Externe Favicon-URLs

Sie können auch externe Favicon-URLs von CDNs oder anderen Domains verwenden:

```yaml
# page-builder.config.yml
favicon: https://cdn.example.com/favicon.ico
favicon: https://mysite.com/assets/logo.svg
```

#### Generierter HTML-Code

Der Builder generiert automatisch geeignete HTML-Tags basierend auf dem Favicon-Typ:

```html
<!-- For SVG favicons -->
<link rel="icon" type="image/svg+xml" href="assets/logo.svg" />

<!-- For ICO favicons (includes legacy browser support) -->
<link rel="icon" type="image/x-icon" href="assets/favicon.ico" />
<link rel="shortcut icon" href="assets/favicon.ico" />

<!-- For external URLs -->
<link rel="icon" href="https://example.com/favicon.ico" />
```

### Navigation

Der Seitenkonstruktor-Builder unterstützt eine globale Navigationskonfiguration, die auf allen Seiten erscheint. Die Navigation wird über eine separate YAML-Datei konfiguriert.

#### Navigationskonfiguration

Erstellen Sie eine `navigation.yml`-Datei im Wurzelverzeichnis Ihres Projekts (oder geben Sie einen benutzerdefinierten Pfad in Ihrer Konfiguration an):

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

#### Überschreibung der Navigation pro Seite

Sie können die Navigation für spezifische Seiten überschreiben, indem Sie einen `navigation`-Abschnitt direkt in Ihrer Seiten-YAML hinzufügen:

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