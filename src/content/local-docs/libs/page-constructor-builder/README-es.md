# Page Constructor Builder

Una potente utilidad de línea de comandos para construir páginas estáticas a partir de configuraciones en YAML utilizando el paquete [@gravity-ui/page-constructor](https://github.com/gravity-ui/page-constructor). Consulta el [storybook de page-constructor](https://preview.gravity-ui.com/page-constructor/) para obtener detalles sobre la configuración.

## Inicio Rápido

1. **Instala el paquete:**

```bash
npm install @gravity-ui/page-constructor-builder
```

2. **Agrega el comando de compilación a package.json:**

```json
{
  "scripts": {
    "build": "page-builder build"
  }
}
```

3. **Agrega los archivos fuente:**

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
  title: Hola, Mundo
  description: Una página simple construida con page constructor

blocks:
  - type: header-block
    title: Hola, Mundo
    description: |
      Construye hermosas páginas estáticas a partir de **configuraciones en YAML** utilizando el poder de 
      [@gravity-ui/page-constructor](https://github.com/gravity-ui/page-constructor).
    background:
      color: '#f8f9fa'
```

4. **Compila tus páginas:**

```bash
npm run build
```

5. **Abre los archivos HTML generados en tu navegador:**

```bash
open dist/index.html
```

## Uso

### Comandos

#### `page-builder build`

Compila páginas a partir de configuraciones en YAML.

```bash
page-builder build [options]
```

**Opciones:**

- `-i, --input <path>`: Directorio de entrada que contiene archivos YAML (predeterminado: "./pages")
- `-o, --output <path>`: Directorio de salida para los archivos compilados (predeterminado: "./dist")
- `-c, --config <path>`: Ruta del archivo de configuración (predeterminado: "./page-builder.config.yml")
- `--css <files...>`: Archivos CSS personalizados para incluir
- `--components <path>`: Directorio de componentes personalizados
- `--navigation <path>`: Archivo de datos de navegación
- `--assets <path>`: Directorio de activos estáticos para copiar
- `--theme <theme>`: Tema (light|dark) (predeterminado: "light")
- `--base-url <url>`: URL base para el sitio
- `--minify`: Habilita la minificación
- `--source-maps`: Genera mapas de origen
- `--watch`: Habilita el modo de vigilancia

### Configuración

Crea un archivo `page-builder.config.yml` en la raíz de tu proyecto:

```yaml
input: ./pages
output: ./dist
assets: ./assets
favicon: logo.svg # Archivo favicon desde assets o URL externa
theme: light
baseUrl: https://mysite.com
minify: true
sourceMaps: false # Genera mapas de origen para depuración (aumenta el tamaño del bundle)
css:
  - ./styles/main.css
  - ./styles/components.scss
components: ./components
navigation: ./navigation.yml
webpack:
  # Configuración personalizada de webpack
```

### Configuración de Página

Crea archivos YAML en tu directorio de páginas:

```yaml
# pages/index.yml
meta:
  title: Bienvenido a Mi Sitio
  description: Esta es la página principal de mi sitio genial

blocks:
  - type: header-block
    title: ¡Bienvenido!
    description: Este es un **bloque de encabezado** con soporte para markdown
    background:
      color: '#f0f0f0'

  - type: content-block
    title: Sobre Nosotros
    text: |
      Este es un bloque de contenido con múltiples líneas de texto.

      Puedes usar formato **markdown** aquí.

  - type: CustomBlock # Tu componente personalizado
    title: Componente Personalizado
    content: Esto usa un componente personalizado
```

### Componentes Personalizados

Crea componentes React en tu directorio de componentes:

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

### Estilos Personalizados

Agrega tus archivos CSS/SCSS personalizados:

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

### Activos Estáticos

El constructor de páginas maneja automáticamente los activos estáticos como imágenes, iconos y otros archivos. Configura el directorio de activos en tu archivo de configuración:

```yaml
# page-builder.config.yml
input: ./pages
output: ./dist
assets: ./assets # Directorio de activos para copiar
```

**Estructura del Directorio de Activos:**

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

**Uso de Activos en Tus Páginas:**

```yaml
# pages/index.yml
blocks:
  - type: header-block
    title: Bienvenido
    description: Revisa nuestro contenido increíble
    background:
      image: assets/images/hero-banner.jpg

  - type: media-block
    title: Sobre Nosotros
    media:
      - type: image
        src: assets/images/about-photo.png
        alt: Foto de nuestro equipo
```

### Favicon

El constructor de páginas soporta agregar favicons a tus páginas estáticas. Puedes especificar un archivo local desde tu directorio de activos o una URL externa.

#### Configuración

Agrega la opción `favicon` a tu archivo de configuración:

```yaml
# page-builder.config.yml
favicon: logo.svg # Archivo local desde el directorio de activos
# o
favicon: https://cdn.example.com/favicon.ico # URL externa
```

#### Archivos Favicon Locales

Para archivos favicon locales, el constructor:

- Detecta automáticamente el archivo en tu directorio de activos
- Lo copia al directorio de salida
- Genera etiquetas HTML `<link>` adecuadas con tipos MIME correctos

**Formatos de archivo compatibles:**

- **SVG** (recomendado) - `image/svg+xml`
- **ICO** (clásico) - `image/x-icon`
- **PNG** (moderno) - `image/png`
- **JPG/JPEG** (aceptable) - `image/jpeg`
- **GIF** (animado) - `image/gif`

**Ejemplos:**

**Ejemplos:**

```yaml
# page-builder.config.yml
favicon: logo.svg                    # File in assets/ directory
favicon: icons/favicon.ico           # File in assets/icons/ subdirectory
favicon: ./custom/path/favicon.png   # Custom path relative to project
favicon: /absolute/path/favicon.ico  # Absolute path
```

#### URLs Externas para Favicon

También puedes usar URLs externas para favicons de CDNs u otros dominios:

```yaml
# page-builder.config.yml
favicon: https://cdn.example.com/favicon.ico
favicon: https://mysite.com/assets/logo.svg
```

#### HTML Generado

El constructor genera automáticamente las etiquetas HTML apropiadas basadas en el tipo de favicon:

```html
<!-- For SVG favicons -->
<link rel="icon" type="image/svg+xml" href="assets/logo.svg" />

<!-- For ICO favicons (includes legacy browser support) -->
<link rel="icon" type="image/x-icon" href="assets/favicon.ico" />
<link rel="shortcut icon" href="assets/favicon.ico" />

<!-- For external URLs -->
<link rel="icon" href="https://example.com/favicon.ico" />
```

### Navegación

El constructor de páginas soporta configuración global de navegación que aparece en todas las páginas. La navegación se configura a través de un archivo YAML separado.

#### Configuración de Navegación

Crea un archivo `navigation.yml` en la raíz de tu proyecto (o especifica una ruta personalizada en tu configuración):

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

#### Anulación de Navegación por Página

Puedes anular la navegación para páginas específicas agregando una sección `navigation` directamente en tu YAML de página:

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