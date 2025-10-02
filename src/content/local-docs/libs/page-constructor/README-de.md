# @gravity-ui/page-constructor &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/page-constructor)](https://www.npmjs.com/package/@gravity-ui/page-constructor) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/page-constructor/ci.yml?branch=main&label=CI)](https://github.com/gravity-ui/page-constructor/actions/workflows/ci.yml?query=branch:main) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/page-constructor/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/page-constructor/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/page-constructor/)

## Seitenkonstruktor

`Page-constructor` ist eine Bibliothek zum Rendern von Webseiten oder Teilen davon basierend auf `JSON`-Daten (die Unterstützung für das `YAML`-Format wird später hinzugefügt).

Beim Erstellen von Seiten wird ein komponentenbasiertes Vorgehen verwendet: Eine Seite wird aus einem Satz vorgefertigter Blöcke aufgebaut, die in beliebiger Reihenfolge platziert werden können. Jeder Block hat einen bestimmten Typ und eine Reihe von Eingabedatenparametern.

Für das Format der Eingabedaten und die Liste der verfügbaren Blöcke siehe die [Dokumentation](https://preview.gravity-ui.com/page-constructor/?path=/docs/documentation-blocks--docs).

## Installation

```shell
npm install @gravity-ui/page-constructor
```

## Schnellstart

Zuerst benötigen wir ein React-Projekt und einen Server. Zum Beispiel können Sie ein React-Projekt mit Vite und einem Express-Server erstellen oder eine Next.js-Anwendung – diese hat auf einmal Client- und Serverseite.

Installieren Sie die erforderlichen Abhängigkeiten:

```shell
npm install @gravity-ui/page-constructor @diplodoc/transform @gravity-ui/uikit
```

Fügen Sie den `Page Constructor` in die Seite ein. Um korrekt zu funktionieren, muss er in einen `PageConstructorProvider` eingebettet werden:

```tsx
import {PageConstructor, PageConstructorProvider} from '@gravity-ui/page-constructor';
import '@gravity-ui/page-constructor/styles/styles.scss';

const App = () => {
  const content = {
    blocks: [
      {
        type: 'header-block',
        title: 'Hello world',
        background: {color: '#f0f0f0'},
        description:
          '**Congratulations!** Have you built a [page-constructor](https://github.com/gravity-ui/page-constructor) into your website',
      },
    ],
  };

  return (
    <PageConstructorProvider>
      <PageConstructor content={content} />
    </PageConstructorProvider>
  );
};

export default App;
```

Das war das einfachste Beispiel für eine Integration. Damit YFM-Markup funktioniert, müssen Sie den Inhalt auf dem Server verarbeiten und ihn auf dem Client empfangen.

Falls Ihr Server eine separate Anwendung ist, installieren Sie page-constructor dort:

```shell
npm install @gravity-ui/page-constructor
```

Um YFM in allen Basisblöcken zu verarbeiten, rufen Sie `contentTransformer` auf und übergeben den Inhalt und Optionen:

```ts
const express = require('express');
const app = express();
const {contentTransformer} = require('@gravity-ui/page-constructor/server');

const content = {
  blocks: [
    {
      type: 'header-block',
      title: 'Hello world',
      background: {color: '#f0f0f0'},
      description:
        '**Congratulations!** Have you built a [page-constructor](https://github.com/gravity-ui/page-constructor) into your website',
    },
  ],
};

app.get('/content', (req, res) => {
  res.send({content: contentTransformer({content, options: {lang: 'en'}})});
});

app.listen(3000);
```

Auf dem Client fügen Sie einen Aufruf des Endpunkts hinzu, um den Inhalt zu empfangen:

```tsx
import {PageConstructor, PageConstructorProvider} from '@gravity-ui/page-constructor';
import '@gravity-ui/page-constructor/styles/styles.scss';
import {useEffect, useState} from 'react';

const App = () => {
  const [content, setContent] = useState();

  useEffect(() => {
    (async () => {
      const response = await fetch('http://localhost:3000/content').then((r) => r.json());
      setContent(response.content);
    })();
  }, []);

  return (
    <PageConstructorProvider>
      <PageConstructor content={content} />
    </PageConstructorProvider>
  );
};

export default App;
```

### Fertige Vorlage

Um ein neues Projekt zu starten, können Sie die [vorgefertigte Vorlage auf Next.js](https://github.com/gravity-ui/page-constructor-website-template) verwenden, die wir vorbereitet haben.

### Statischer Site-Builder

[Page Constructor Builder](https://github.com/gravity-ui/page-constructor-builder) – Kommandozeilen-Utility zum Erstellen statischer Seiten aus YAML-Konfigurationen mit @gravity-ui/page-constructor

## Dokumentation

### Parameter

```typescript
interface PageConstructorProps {
  content: PageContent; // Blockdaten im JSON-Format.
  shouldRenderBlock?: ShouldRenderBlock; // Eine Funktion, die beim Rendern jedes Blocks aufgerufen wird und Bedingungen für dessen Anzeige festlegt.
  custom?: Custom; // Benutzerdefinierte Blöcke (siehe `Anpassung`).
  renderMenu?: () => React.ReactNode; // Eine Funktion, die das Seitenmenü mit Navigation rendert (wir planen, die Darstellung der Standard-Menüversion hinzuzufügen).
  navigation?: NavigationData; // Navigationsdaten für die Verwendung der Navigationskomponente im JSON-Format
  isBranded?: boolean; // Wenn true, fügt einen Footer hinzu, der auf https://gravity-ui.com/ verlinkt. Verwenden Sie die BrandFooter-Komponente für mehr Anpassungsmöglichkeiten.
}

interface PageConstructorProviderProps {
  isMobile?: boolean; //Ein Flag, das anzeigt, dass der Code im mobilen Modus ausgeführt wird.
  locale?: LocaleContextProps; //Informationen über Sprache und Domain (wird bei der Generierung und Formatierung von Links verwendet).
  location?: Location; //API des Browsers oder des Router-History, die URL der Seite.
  analytics?: AnalyticsContextProps; // Funktion zur Behandlung von Analytics-Events

  ssrConfig?: SSR; //Ein Flag, das anzeigt, dass der Code auf der Serverseite ausgeführt wird.
  theme?: 'light' | 'dark'; //Thema, mit dem die Seite gerendert werden soll.
  mapsContext?: MapsContextType; //Parameter für die Karte: apikey, type, scriptSrc, nonce
}

export interface PageContent extends Animatable {
  blocks: Block[];
  menu?: Menu;
  background?: MediaProps;
}

interface Custom {
  blocks?: CustomItems;
  subBlocks?: CustomItems;
  headers?: CustomItems;
  loadable?: LoadableConfig;
}

type ShouldRenderBlock = (block: Block, blockKey: string) => Boolean;

interface Location {
  history?: History;
  search?: string;
  hash?: string;
  pathname?: string;
  hostname?: string;
}

interface Locale {
  lang?: Lang;
  tld?: string;
}

interface SSR {
  isServer?: boolean;
}

interface NavigationData {
  logo: NavigationLogo;
  header: HeaderData;
}

interface NavigationLogo {
  icon: ImageProps;
  text?: string;
  url?: string;
}

interface HeaderData {
  leftItems: NavigationItem[];
  rightItems?: NavigationItem[];
}

```

```typescript
interface NavigationLogo {
  icon: ImageProps;
  text?: string;
  url?: string;
}
```

### Server-Utils

Das Paket stellt eine Reihe von Server-Utilities zur Verfügung, um Ihren Inhalt zu transformieren.

```ts
const {fullTransform} = require('@gravity-ui/page-constructor/server');

const {html} = fullTransform(content, {
  lang,
  extractTitle: true,
  allowHTML: true,
  path: __dirname,
  plugins,
});
```

Unter der Haube wird ein Paket verwendet, um Yandex Flavored Markdown in HTML umzuwandeln – `diplodoc/transform`, das daher auch in den Peer-Dependencies enthalten ist.

Sie können auch nützliche Utilities an den Stellen verwenden, an denen Sie sie benötigen, z. B. in Ihren benutzerdefinierten Komponenten.

```ts
const {
  typografToText,
  typografToHTML,
  yfmTransformer,
} = require('@gravity-ui/page-constructor/server');

const post = {
  title: typografToText(title, lang),
  content: typografToHTML(content, lang),
  description: yfmTransformer(lang, description, {plugins}),
};
```

Weitere Utilities finden Sie in diesem [Abschnitt](https://github.com/gravity-ui/page-constructor/tree/main/src/text-transform).

### Detaillierte Dokumentation zu Server-Utilities und Transformern

Für eine umfassende Anleitung zur Verwendung von Server-Utilities, einschließlich detaillierter Erklärungen und fortgeschrittener Anwendungsfälle, besuchen Sie das [zusätzliche Kapitel zur Nutzung von Server-Utils](./docs/data-preparation.md).

### Benutzerdefinierte Blöcke

Der Page Constructor ermöglicht die Verwendung von benutzerdefinierten Blöcken in Ihrer App. Blöcke sind reguläre React-Komponenten.

Um benutzerdefinierte Blöcke an den Constructor zu übergeben:

1. Erstellen Sie einen Block in Ihrer App.

2. Erstellen Sie in Ihrem Code ein Objekt mit dem Block-Typ (String) als Schlüssel und der importierten Block-Komponente als Wert.

3. Übergeben Sie das erstellte Objekt an den Parameter `custom.blocks`, `custom.headers` oder `custom.subBlocks` der `PageConstructor`-Komponente (`custom.headers` gibt die Block-Header an, die separat über dem allgemeinen Inhalt gerendert werden sollen).

4. Nun können Sie den erstellten Block in den Eingabedaten (dem `content`-Parameter) verwenden, indem Sie seinen Typ und seine Daten angeben.

Um Mixins und Constructor-Style-Variablen bei der Erstellung benutzerdefinierter Blöcke zu verwenden, fügen Sie einen Import in Ihre Datei hinzu:

```css
@import '~@gravity-ui/page-constructor/styles/styles.scss';
```

Um die Standard-Schriftart zu verwenden, fügen Sie einen Import in Ihre Datei hinzu:

```css
@import '~@gravity-ui/page-constructor/styles/fonts.scss';
```

### Ladbare Blöcke

Manchmal ist es notwendig, dass ein Block sich selbst basierend auf zu ladenden Daten rendert. In diesem Fall werden ladbare Blöcke verwendet.

Um benutzerdefinierte `loadable`-Blöcke hinzuzufügen, übergeben Sie an den `PageConstructor` die Eigenschaft `custom.loadable` mit den Datenquellen-Namen (String) für die Komponente als Schlüssel und einem Objekt als Wert.

```typescript
export interface LoadableConfigItem {
  fetch: FetchLoadableData; // Methode zum Laden der Daten
  component: React.ComponentType; // Komponente, an die die geladenen Daten übergeben werden
}

type FetchLoadableData<TData = any> = (blockKey: string) => Promise<TData>;
```

### Grid

Der Page Constructor verwendet das `bootstrap`-Grid und seine Implementierung basierend auf React-Komponenten, die Sie in Ihrem eigenen Projekt verwenden können (einschließlich separat vom Constructor).

Beispiel zur Verwendung:

```jsx
import {Grid, Row, Col} from '@gravity-ui/page-constructor';

const Page = ({children}: PropsWithChildren<PageProps>) => (
  <Grid>
    <Row>
      <Col sizes={{lg: 4, sm: 6, all: 12}}>{children}</Col>
    </Row>
  </Grid>
);
```

### Navigation

Die Seiten-Navigation kann auch separat vom Constructor verwendet werden:

```jsx
import {Navigation} from '@gravity-ui/page-constructor';

const Page= ({data, logo}: React.PropsWithChildren<PageProps>) => <Navigation data={data} logo={logo} />;
```

### Blöcke

Jeder Block ist eine atomare Top-Level-Komponente. Sie werden im Verzeichnis `src/units/constructor/blocks` gespeichert.

### Sub-Blöcke

Sub-Blöcke sind Komponenten, die in der `children`-Eigenschaft eines Blocks verwendet werden können. In einer Konfiguration wird eine Liste von Kind-Komponenten aus Sub-Blöcken angegeben. Sobald sie gerendert werden, werden diese Sub-Blöcke als `children` an den Block übergeben.

### So fügen Sie einen neuen Block zum `page-constructor` hinzu

1. Erstellen Sie im Verzeichnis `src/blocks` oder `src/sub-blocks` einen Ordner mit dem Code für den Block oder Sub-Block.

2. Fügen Sie den Namen des Blocks oder Sub-Blocks zum Enum `BlockType` oder `SubBlockType` hinzu und beschreiben Sie seine Eigenschaften in der Datei `src/models/constructor-items/blocks.ts` oder `src/models/constructor-items/sub-blocks.ts` ähnlich wie bei den bestehenden.

3. Fügen Sie einen Export für den Block in der Datei `src/blocks/index.ts` und für den Sub-Block in der Datei `src/sub-blocks/index.ts` hinzu.

4. Fügen Sie die neue Komponente oder den Block zur Zuordnung in `src/constructor-items.ts` hinzu.

5. Fügen Sie einen Validator für den neuen Block hinzu:

   - Erstellen Sie eine Datei `schema.ts` im Verzeichnis des Blocks oder Sub-Blocks. In dieser Datei beschreiben Sie einen Parameter-Validator für die Komponente im Format von [`json-schema`](http://json-schema.org/).
   - Exportieren Sie sie in der Datei `schema/validators/blocks.ts` oder `schema/validators/sub-blocks.ts`.
   - Fügen Sie sie zu `enum` oder `selectCases` in der Datei `schema/index.ts` hinzu.

6. Fügen Sie im Block-Verzeichnis eine Datei `README.md` mit einer Beschreibung der Eingabeparameter hinzu.

7. Fügen Sie im Block-Verzeichnis ein Storybook-Demo im Ordner `__stories__` hinzu. Aller Demo-Inhalt für die Story sollte in `data.json` im Story-Verzeichnis platziert werden. Die generische `Story` muss den Typ der Block-Props akzeptieren, andernfalls werden falsche Block-Props in Storybook angezeigt.

8. Fügen Sie eine Block-Daten-Vorlage zum Ordner `src/editor/data/templates/` hinzu; der Dateiname sollte mit dem Block-Typ übereinstimmen.

9. (optional) Fügen Sie ein Block-Vorschau-Icon zum Ordner `src/editor/data/previews/` hinzu; der Dateiname sollte mit dem Block-Typ übereinstimmen.

### Themes

Der `PageConstructor` ermöglicht die Verwendung von Themes: Sie können unterschiedliche Werte für einzelne Block-Eigenschaften je nach im App ausgewähltem Theme festlegen.

Um ein Theme zu einer Block-Eigenschaft hinzuzufügen:

1. Definieren Sie in der Datei `models/blocks.ts` den Typ der jeweiligen Block-Eigenschaft mit dem Generic `ThemeSupporting<T>`, wobei `T` der Typ der Eigenschaft ist.

2. In der Datei mit der `react`-Komponente des Blocks erhalten Sie den Wert der Eigenschaft mit Theme über `getThemedValue` und den `useTheme`-Hook (siehe Beispiele im Block `MediaBlock.tsx`).

3. Fügen Sie Theme-Unterstützung zum Eigenschafts-Validator hinzu: In der Datei `schema.ts` des Blocks umschließen Sie diese Eigenschaft mit `withTheme`.

### i18n

Der `page-constructor` ist eine `uikit-basierte` Bibliothek, und wir verwenden eine Instanz von `i18n` aus uikit. Um die Internationalisierung einzurichten, müssen Sie nur `configure` aus uikit verwenden:

```typescript
import {configure} from '@gravity-ui/uikit';

configure({
  lang: 'ru',
});
```

### Karten

Um Karten zu verwenden, geben Sie den Kartentyp, `scriptSrc` und `apiKey` im Feld `mapContext` im `PageConstructorProvider` an.

Sie können Umgebungsvariablen für den Dev-Modus in der Datei `.env.development` im Projekt-Root definieren.  
`STORYBOOK_GMAP_API_KEY` – apiKey für Google Maps.

### Analytics

#### Initialisierung

Um Analytics zu verwenden, übergeben Sie einen Handler an den Constructor. Der Handler muss auf der Projektseite erstellt werden. Der Handler erhält die `default`- und `custom`-Event-Objekte. Der übergebene Handler wird bei Klicks auf Buttons, Links, Navigation und Steuerelemente ausgelöst. Da ein Handler für alle Event-Behandlungen verwendet wird, achten Sie darauf, wie Sie verschiedene Events in Ihrem Handler behandeln. Es gibt vordefinierte Felder, die Ihnen helfen, komplexe Logik aufzubauen.

Übergeben Sie `autoEvents: true` an den Constructor, um automatisch konfigurierte Events auszulösen.

```ts
function sendEvents(events: MyEventType []) {
  ...
}

<PageConstructorProvider
    ...

    analytics={{sendEvents, autoEvents: true}}

    ...
/>
```

Ein Event-Objekt hat nur ein erforderliches Feld – `name`. Es enthält zudem vordefinierte Felder, die dabei helfen, komplexe Logik zu verwalten. Zum Beispiel kann `counter.include` nützlich sein, um ein Event in einem bestimmten Counter zu senden, falls mehrere Analytics-Systeme in einem Projekt im Einsatz sind.

```ts
type AnalyticsEvent<T = {}> = T & {
  name: string;
  type?: string;
  counters?: AnalyticsCounters;
  context?: string;
};
```

Es ist möglich, den für das Projekt benötigten Event-Typ anzupassen.

```ts
type MyEventType = AnalyticsEvent<{
  [key: string]?: string; // nur der Typ 'string' wird unterstützt
}>;
```

#### Counter-Auswahl

Es ist möglich, ein Event so zu konfigurieren, dass es an ein bestimmtes Analytics-System gesendet wird.

```ts
type AnalyticsCounters = {
  include?: string[]; // Array von Analytics-Counter-IDs, die angewendet werden sollen
  exclude?: string[]; // Array von Analytics-Counter-IDs, die nicht angewendet werden sollen
};
```

#### context-Parameter

Übergeben Sie den Wert `context`, um den Ort im Projekt anzugeben, an dem das Event ausgelöst wird.

Verwenden Sie den folgenden Selektor oder erstellen Sie eine Logik, die den Anforderungen des Projekts entspricht.

```ts
// analyticsHandler.ts
if (isCounterAllowed(counterName, counters)) {
  analyticsCounter.reachGoal(counterName, name, parameters);
}
```

#### Vordefinierte Event-Typen

Mehrere vordefinierte Event-Typen werden verwendet, um automatisch konfigurierte Events zu markieren. Diese Typen eignen sich beispielsweise zum Filtern von Standard-Events.

```ts
enum PredefinedEventTypes {
  Default = 'default-event', // Standard-Events, die bei jedem Button-Klick ausgelöst werden
  Play = 'play', // React-Player-Event
  Stop = 'stop', // React-Player-Event
}
```

## Entwicklung

```bash
npm ci
npm run dev
```

#### Hinweis zu Vite

```ts
import react from '@vitejs/plugin-react-swc';
import dynamicImport from 'vite-plugin-dynamic-import';

export default defineConfig({
  plugins: [
    react(),
    dynamicImport({
      filter: (id) => id.includes('/node_modules/@gravity-ui/page-constructor'),
    }),
  ],
});
```

Für Vite müssen Sie das Plugin `vite-plugin-dynamic-import` installieren und die Konfiguration so anpassen, dass dynamische Imports funktionieren.

## Release-Prozess

In der Regel verwenden wir zwei Arten von Commits:

1. fix: Ein Commit vom Typ fix behebt einen Fehler im Code (das entspricht PATCH in Semantic Versioning).
2. feat: Ein Commit vom Typ feat führt eine neue Funktion ein (das entspricht MINOR in Semantic Versioning).
3. BREAKING CHANGE: Ein Commit mit einem Footer BREAKING CHANGE: oder einem ! nach dem Typ/Scope führt eine breaking API-Änderung ein (das entspricht MAJOR in Semantic Versioning). Ein BREAKING CHANGE kann in Commits beliebiger Typen vorkommen.
4. Um die Release-Paketversion manuell festzulegen, fügen Sie `Release-As: <version>` zu Ihrer Commit-Nachricht hinzu, z. B.:

```bash
git commit -m 'chore: bump release

Release-As: 1.2.3'
```

Alle Informationen dazu finden Sie [hier](https://www.conventionalcommits.org/en/v1.0.0/).

Sobald Ihre Pull-Request von den Code-Owners genehmigt wurde und alle Checks bestanden sind, gehen Sie wie folgt vor:

1. Überprüfen Sie, ob es eine Release-Pull-Request vom Robot mit Änderungen eines anderen Mitwirkenden gibt (sie sieht aus wie `chore(main): release 0.0.0`). Falls ja, prüfen Sie, warum sie nicht gemerged wurde. Wenn der Mitwirkende einer gemeinsamen Version zustimmt, fahren Sie mit dem nächsten Schritt fort. Andernfalls bitten Sie ihn, seine Version zuerst zu releasen, und fahren Sie dann fort.
2. Squaschen und mergen Sie Ihren PR (Es ist wichtig, eine neue Version mit GitHub Actions zu releasen.)
3. Warten Sie, bis der Robot einen PR mit einer neuen Paketversion und Informationen zu Ihren Änderungen in der CHANGELOG.md erstellt. Den Prozess können Sie im [Actions-Tab](https://github.com/gravity-ui/page-constructor/actions) verfolgen.
4. Überprüfen Sie Ihre Änderungen in der CHANGELOG.md und genehmigen Sie den PR des Robots.
5. Squaschen und mergen Sie den PR. Den Release-Prozess können Sie im [Actions-Tab](https://github.com/gravity-ui/page-constructor/actions) verfolgen.

### Release von Alpha-Versionen

Falls Sie eine Alpha-Version des Pakets aus Ihrem Branch releasen möchten, können Sie das manuell tun:

1. Gehen Sie zum Tab Actions.
2. Wählen Sie auf der linken Seite den Workflow „Release alpha version“ aus.
3. Auf der rechten Seite sehen Sie den Button „Run workflow“. Hier können Sie den Branch auswählen.
4. Sie sehen auch ein Feld für die manuelle Version. Bei der ersten Alpha-Release aus Ihrem Branch lassen Sie es leer. Nach dem ersten Release müssen Sie die neue Version manuell angeben, da wir package.json nicht ändern, falls der Branch bald ausläuft. Verwenden Sie das Präfix `alpha` in Ihrer manuellen Version, andernfalls erhalten Sie einen Fehler.
5. Klicken Sie auf „Run workflow“ und warten Sie, bis die Action abgeschlossen ist. Sie können so viele Versionen releasen, wie Sie möchten, aber missbrauchen Sie es nicht und releasen Sie nur, wenn es wirklich nötig ist. In anderen Fällen verwenden Sie [npm pack](https://docs.npmjs.com/cli/v7/commands/npm-pack).

### Release von Beta-Major-Versionen

Falls Sie eine neue Major-Version releasen möchten, benötigen Sie wahrscheinlich Beta-Versionen vor der stabilen Version. Gehen Sie wie folgt vor:

1. Erstellen oder aktualisieren Sie den Branch `beta`.
2. Fügen Sie dort Ihre Änderungen hinzu.
3. Wenn Sie für eine neue Beta-Version bereit sind, releasen Sie sie manuell mit einem leeren Commit (oder fügen Sie diese Commit-Nachricht mit Footer zum letzten Commit hinzu):

```bash
git commit -m 'fix: last commit

Release-As: 3.0.0-beta.0' --allow-empty
```

4. Der Robot erstellt einen neuen PR zum Branch `beta` mit aktualisierter CHANGELOG.md und angepasster Paketversion.
5. Sie können das so oft wiederholen, wie Sie möchten. Wenn Sie bereit sind, die finale Major-Version ohne Beta-Tag zu releasen, erstellen Sie einen PR vom Branch `beta` zum Branch `main`. Beachten Sie, dass es normal ist, wenn Ihre Paketversion den Beta-Tag trägt. Der Robot erkennt das und passt es korrekt an. `3.0.0-beta.0` wird zu `3.0.0`.

### Release-Prozess für vorherige Major-Versionen

Falls Sie eine neue Version in einer vorherigen Major-Version releasen möchten, nachdem Sie sie in den main-Branch committet haben, gehen Sie wie folgt vor:

1. Aktualisieren Sie den notwendigen Branch. Die Namen der Branches für vorherige Major-Releases lauten:
   1. `version-1.x.x/fixes` – für Major 1.x.x
   2. `version-2.x.x` – für Major 2.x.x
2. Checken Sie einen neuen Branch aus dem vorherigen Major-Release-Branch aus.
3. Cherry-pick-en Sie Ihren Commit aus dem Branch `main`.
4. Erstellen Sie einen PR, lassen Sie ihn genehmigen und mergen Sie ihn in den vorherigen Major-Release-Branch.
5. Squaschen und mergen Sie Ihren PR (Es ist wichtig, eine neue Version mit GitHub Actions zu releasen.)
6. Warten Sie, bis der Robot einen PR mit einer neuen Paketversion und Informationen zu Ihren Änderungen in der CHANGELOG.md erstellt. Den Prozess können Sie im [Actions-Tab](https://github.com/gravity-ui/page-constructor/actions) verfolgen.
7. Überprüfen Sie Ihre Änderungen in der CHANGELOG.md und genehmigen Sie den PR des Robots.
8. Squaschen und mergen Sie den PR. Den Release-Prozess können Sie im [Actions-Tab](https://github.com/gravity-ui/page-constructor/actions) verfolgen.

## Page-Constructor-Editor

Der Editor bietet eine Benutzeroberfläche zur Verwaltung von Seiteninhalten mit Echtzeit-Vorschau.

So verwenden Sie ihn:

```tsx
import {Editor} from '@gravity-ui/page-constructor/editor';

interface MyAppEditorProps {
  initialContent: PageContent;
  transformContent: ContentTransformer;
  onChange: (content: PageContent) => void;
}

export const MyAppEditor = ({initialContent, onChange, transformContent}: MyAppEditorProps) => (
  <Editor content={initialContent} onChange={onChange} transformContent={transformContent} />
);
```

## Memory Bank

```

Dieses Projekt umfasst eine umfassende **Memory Bank** – eine Sammlung von Markdown-Dokumentationsdateien, die detaillierte Informationen zur Projektarchitektur, den Komponenten und Nutzungsmustern bieten. Die Memory Bank ist besonders nützlich beim Arbeiten mit KI-Agenten, da sie strukturierte Informationen zu folgenden Themen enthält:

- **Projektübersicht**: Kernanforderungen, Ziele und Kontext
- **Komponentendokumentation**: Detaillierte Nutzungsanleitungen für alle Komponenten
- **Systemarchitektur**: Technische Muster und Designentscheidungen
- **Entwicklungsfortschritt**: Aktueller Status und Implementierungsdetails

### Die Memory Bank verwenden

Die Memory Bank befindet sich im Verzeichnis `memory-bank/` und besteht aus regulären Markdown-Dateien, die wie jede andere Dokumentation gelesen werden können:

- `projectbrief.md` – Grundsatz-Dokument mit den Kernanforderungen
- `productContext.md` – Projektzweck und Ziele für die Benutzererfahrung
- `systemPatterns.md` – Architektur und technische Entscheidungen
- `techContext.md` – Technologien, Einrichtung und Einschränkungen
- `activeContext.md` – Aktueller Arbeitsfokus und kürzliche Änderungen
- `progress.md` – Implementierungsstatus und bekannte Probleme
- `usage/` – Komponentenspezifische Nutzungsdokumentation
- `storybookComponents.md` – Details zur Storybook-Integration

### Für KI-Agenten

Beim Arbeiten mit KI-Agenten an diesem Projekt dient die Memory Bank als umfassende Wissensbasis, die Agenten hilft zu verstehen:

- Projektstruktur und Muster
- Komponenten-APIs und Nutzungsbeispiele
- Entwicklungsworkflows und Best Practices
- Aktuellen Implementierungsstatus und nächsten Schritten

KI-Agenten können diese Dateien lesen, um sich schnell mit dem Projektkontext vertraut zu machen und fundiertere Entscheidungen zu Code-Änderungen und Implementierungen zu treffen.

## Tests

Umfassende Dokumentation ist unter dem angegebenen [Link](./test-utils/docs/README.md) verfügbar.