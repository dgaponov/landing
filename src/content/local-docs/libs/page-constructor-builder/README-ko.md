# 페이지 생성기 빌더

YAML 구성으로부터 정적 페이지를 빌드하기 위한 강력한 명령줄 유틸리티로, [@gravity-ui/page-constructor](https://github.com/gravity-ui/page-constructor) 패키지를 사용합니다. 구성 세부 사항은 [page-constructor storybook](https://preview.gravity-ui.com/page-constructor/)을 참조하세요.

## 빠른 시작

1. **패키지 설치:**

```bash
npm install @gravity-ui/page-constructor-builder
```

2. **package.json에 빌드 명령 추가:**

```json
{
  "scripts": {
    "build": "page-builder build"
  }
}
```

3. **소스 파일 추가:**

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

4. **페이지 빌드:**

```bash
npm run build
```

5. **생성된 HTML 파일을 브라우저에서 열기:**

```bash
open dist/index.html
```

## 사용법

### 명령어

#### `page-builder build`

YAML 구성으로부터 페이지를 빌드합니다.

```bash
page-builder build [options]
```

**옵션:**

- `-i, --input <path>`: YAML 파일이 포함된 입력 디렉터리 (기본값: "./pages")
- `-o, --output <path>`: 빌드된 파일의 출력 디렉터리 (기본값: "./dist")
- `-c, --config <path>`: 구성 파일 경로 (기본값: "./page-builder.config.yml")
- `--css <files...>`: 포함할 사용자 정의 CSS 파일
- `--components <path>`: 사용자 정의 컴포넌트 디렉터리
- `--navigation <path>`: 네비게이션 데이터 파일
- `--assets <path>`: 복사할 정적 에셋 디렉터리
- `--theme <theme>`: 테마 (light|dark) (기본값: "light")
- `--base-url <url>`: 사이트의 기본 URL
- `--minify`: 압축 활성화
- `--source-maps`: 소스 맵 생성
- `--watch`: 감시 모드 활성화

### 구성

프로젝트 루트에 `page-builder.config.yml` 파일을 생성하세요:

```yaml
input: ./pages
output: ./dist
assets: ./assets
favicon: logo.svg # assets에서 가져오거나 외부 URL의 파비콘 파일
theme: light
baseUrl: https://mysite.com
minify: true
sourceMaps: false # 디버깅을 위한 소스 맵 생성 (번들 크기 증가)
css:
  - ./styles/main.css
  - ./styles/components.scss
components: ./components
navigation: ./navigation.yml
webpack:
  # 사용자 정의 webpack 구성
```

### 페이지 구성

페이지 디렉터리에 YAML 파일을 생성하세요:

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

### 사용자 정의 컴포넌트

컴포넌트 디렉터리에 React 컴포넌트를 생성하세요:

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

### 사용자 정의 스타일

사용자 정의 CSS/SCSS 파일을 추가하세요:

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

### 정적 에셋

페이지 생성기 빌더는 이미지, 아이콘 및 기타 파일과 같은 정적 에셋을 자동으로 처리합니다. 구성 파일에서 에셋 디렉터리를 설정하세요:

```yaml
# page-builder.config.yml
input: ./pages
output: ./dist
assets: ./assets # 복사할 에셋 디렉터리
```

**에셋 디렉터리 구조:**

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

**페이지에서 에셋 사용:**

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

### 파비콘

페이지 생성기 빌더는 정적 페이지에 파비콘을 추가하는 것을 지원합니다. 에셋 디렉터리의 로컬 파일이나 외부 URL을 지정할 수 있습니다.

#### 구성

구성 파일에 `favicon` 옵션을 추가하세요:

```yaml
# page-builder.config.yml
favicon: logo.svg # 에셋 디렉터리의 로컬 파일
# 또는
favicon: https://cdn.example.com/favicon.ico # 외부 URL
```

#### 로컬 파비콘 파일

로컬 파비콘 파일의 경우, 빌더는 다음을 수행합니다:

- 에셋 디렉터리에서 파일을 자동으로 감지
- 출력 디렉터리로 복사
- 올바른 MIME 유형으로 적절한 HTML `<link>` 태그 생성

**지원 파일 형식:**

- **SVG** (권장) - `image/svg+xml`
- **ICO** (클래식) - `image/x-icon`
- **PNG** (현대적) - `image/png`
- **JPG/JPEG** (허용) - `image/jpeg`
- **GIF** (애니메이션) - `image/gif`

**예시:**

```yaml
# page-builder.config.yml
favicon: logo.svg                    # File in assets/ directory
favicon: icons/favicon.ico           # File in assets/icons/ subdirectory
favicon: ./custom/path/favicon.png   # Custom path relative to project
favicon: /absolute/path/favicon.ico  # Absolute path
```

#### 외부 Favicon URL

CDN이나 다른 도메인에서 제공하는 외부 favicon URL도 사용할 수 있습니다:

```yaml
# page-builder.config.yml
favicon: https://cdn.example.com/favicon.ico
favicon: https://mysite.com/assets/logo.svg
```

#### 생성된 HTML

빌더는 favicon 유형에 따라 적절한 HTML 태그를 자동으로 생성합니다:

```html
<!-- For SVG favicons -->
<link rel="icon" type="image/svg+xml" href="assets/logo.svg" />

<!-- For ICO favicons (includes legacy browser support) -->
<link rel="icon" type="image/x-icon" href="assets/favicon.ico" />
<link rel="shortcut icon" href="assets/favicon.ico" />

<!-- For external URLs -->
<link rel="icon" href="https://example.com/favicon.ico" />
```

### 네비게이션

페이지 빌더는 모든 페이지에 나타나는 전역 네비게이션 구성을 지원합니다. 네비게이션은 별도의 YAML 파일을 통해 구성됩니다.

#### 네비게이션 구성

프로젝트 루트에 `navigation.yml` 파일을 생성하거나(또는 구성에서 사용자 지정 경로를 지정) 다음과 같이 구성하세요:

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

#### 페이지별 네비게이션 재정의

특정 페이지에 대해 네비게이션을 재정의하려면 페이지 YAML에 직접 `navigation` 섹션을 추가하세요:

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