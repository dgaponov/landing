# Page Constructor Builder

Un utilitaire en ligne de commande puissant pour construire des pages statiques à partir de configurations YAML en utilisant le package [@gravity-ui/page-constructor](https://github.com/gravity-ui/page-constructor). Consultez le [storybook de page-constructor](https://preview.gravity-ui.com/page-constructor/) pour plus de détails sur la configuration.

## Démarrage rapide

1. **Installer le package :**

```bash
npm install @gravity-ui/page-constructor-builder
```

2. **Ajouter la commande de build à package.json :**

```json
{
  "scripts": {
    "build": "page-builder build"
  }
}
```

3. **Ajouter les fichiers sources :**

`page-builder.config.yml` :

```yaml
input: ./pages
output: ./dist
assets: ./assets
favicon: logo.svg
theme: light
minify: true
```

`pages/index.yml` :

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

4. **Construire vos pages :**

```bash
npm run build
```

5. **Ouvrir les fichiers HTML générés dans votre navigateur :**

```bash
open dist/index.html
```

## Utilisation

### Commandes

#### `page-builder build`

Construit des pages à partir de configurations YAML.

```bash
page-builder build [options]
```

**Options :**

- `-i, --input <path>` : Répertoire d'entrée contenant les fichiers YAML (par défaut : "./pages")
- `-o, --output <path>` : Répertoire de sortie pour les fichiers construits (par défaut : "./dist")
- `-c, --config <path>` : Chemin vers le fichier de configuration (par défaut : "./page-builder.config.yml")
- `--css <files...>` : Fichiers CSS personnalisés à inclure
- `--components <path>` : Répertoire des composants personnalisés
- `--navigation <path>` : Fichier de données de navigation
- `--assets <path>` : Répertoire des ressources statiques à copier
- `--theme <theme>` : Thème (light|dark) (par défaut : "light")
- `--base-url <url>` : URL de base pour le site
- `--minify` : Activer la minification
- `--source-maps` : Générer des source maps
- `--watch` : Activer le mode surveillance

### Configuration

Créez un fichier `page-builder.config.yml` à la racine de votre projet :

```yaml
input: ./pages
output: ./dist
assets: ./assets
favicon: logo.svg # Fichier favicon depuis les assets ou URL externe
theme: light
baseUrl: https://mysite.com
minify: true
sourceMaps: false # Générer des source maps pour le débogage (augmente la taille du bundle)
css:
  - ./styles/main.css
  - ./styles/components.scss
components: ./components
navigation: ./navigation.yml
webpack:
  # Configuration webpack personnalisée
```

### Configuration des pages

Créez des fichiers YAML dans votre répertoire de pages :

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

  - type: CustomBlock # Votre composant personnalisé
    title: Custom Component
    content: This uses a custom component
```

### Composants personnalisés

Créez des composants React dans votre répertoire components :

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

### Styles personnalisés

Ajoutez vos fichiers CSS/SCSS personnalisés :

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

### Ressources statiques

Le builder de page constructor gère automatiquement les ressources statiques comme les images, les icônes et les autres fichiers. Configurez le répertoire des assets dans votre fichier de configuration :

```yaml
# page-builder.config.yml
input: ./pages
output: ./dist
assets: ./assets # Répertoire des assets à copier
```

**Structure du répertoire Assets :**

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

**Utilisation des assets dans vos pages :**

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

Le builder de page constructor prend en charge l'ajout de favicons à vos pages statiques. Vous pouvez spécifier soit un fichier local depuis votre répertoire assets, soit une URL externe.

#### Configuration

Ajoutez l'option `favicon` à votre fichier de configuration :

```yaml
# page-builder.config.yml
favicon: logo.svg # Fichier local depuis le répertoire assets
# ou
favicon: https://cdn.example.com/favicon.ico # URL externe
```

#### Fichiers Favicon locaux

Pour les fichiers favicon locaux, le builder va :

- Détecter automatiquement le fichier dans votre répertoire assets
- Le copier vers le répertoire de sortie
- Générer les balises HTML `<link>` appropriées avec les types MIME corrects

**Formats de fichiers pris en charge :**

- **SVG** (recommandé) - `image/svg+xml`
- **ICO** (classique) - `image/x-icon`
- **PNG** (moderne) - `image/png`
- **JPG/JPEG** (acceptable) - `image/jpeg`
- **GIF** (animé) - `image/gif`

**Exemples :**

```yaml
# page-builder.config.yml
favicon: logo.svg                    # File in assets/ directory
favicon: icons/favicon.ico           # File in assets/icons/ subdirectory
favicon: ./custom/path/favicon.png   # Custom path relative to project
favicon: /absolute/path/favicon.ico  # Absolute path
```

#### URLs Externes pour Favicon

Vous pouvez également utiliser des URLs externes pour favicon provenant de CDNs ou d'autres domaines :

```yaml
# page-builder.config.yml
favicon: https://cdn.example.com/favicon.ico
favicon: https://mysite.com/assets/logo.svg
```

#### HTML Généré

Le constructeur génère automatiquement les balises HTML appropriées en fonction du type de favicon :

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

Le constructeur de pages prend en charge la configuration globale de la navigation, qui apparaît sur toutes les pages. La navigation est configurée via un fichier YAML séparé.

#### Configuration de la Navigation

Créez un fichier `navigation.yml` à la racine de votre projet (ou spécifiez un chemin personnalisé dans votre configuration) :

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

#### Remplacement de la Navigation par Page

Vous pouvez remplacer la navigation pour des pages spécifiques en ajoutant une section `navigation` directement dans votre fichier YAML de page :

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