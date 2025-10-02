# @gravity-ui/page-constructor &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/page-constructor)](https://www.npmjs.com/package/@gravity-ui/page-constructor) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/page-constructor/ci.yml?branch=main&label=CI)](https://github.com/gravity-ui/page-constructor/actions/workflows/ci.yml?query=branch:main) [![Release](https://img.shields.io/github/actions/workflow/status/gravity-ui/page-constructor/release.yml?branch=main&label=Release)](https://github.com/gravity-ui/page-constructor/actions/workflows/release.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/page-constructor/)

## Constructeur de pages

`Page-constructor` est une bibliothèque pour le rendu de pages web ou de leurs parties en se basant sur des données `JSON` (le support du format `YAML` sera ajouté ultérieurement).

Lors de la création de pages, une approche basée sur les composants est utilisée : une page est construite à l'aide d'un ensemble de blocs prêts à l'emploi qui peuvent être placés dans n'importe quel ordre. Chaque bloc possède un type spécifique et un ensemble de paramètres d'entrée.

Pour le format des données d'entrée et la liste des blocs disponibles, consultez la [documentation](https://preview.gravity-ui.com/page-constructor/?path=/docs/documentation-blocks--docs).

## Installation

```shell
npm install @gravity-ui/page-constructor
```

## Démarrage rapide

Tout d'abord, nous avons besoin d'un projet React et d'un serveur quelconque. Par exemple, vous pouvez créer un projet React avec Vite et un serveur Express, ou bien une application Next.js – qui disposera à la fois d'un côté client et serveur.

Installez les dépendances requises :

```shell
npm install @gravity-ui/page-constructor @diplodoc/transform @gravity-ui/uikit
```

Insérez le `Page Constructor` dans la page. Pour un fonctionnement correct, il doit être enveloppé dans un `PageConstructorProvider` :

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

C'était l'exemple le plus simple d'intégration. Pour que le balisage YFM fonctionne, il faut traiter le contenu côté serveur et le recevoir côté client.

Si votre serveur est une application séparée, installez alors page-constructor :

```shell
npm install @gravity-ui/page-constructor
```

Pour traiter le YFM dans tous les blocs de base, appelez `contentTransformer` et passez-y le contenu et les options :

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

Côté client, ajoutez un appel à l'endpoint pour recevoir le contenu :

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

### Modèle prêt à l'emploi

Pour démarrer un nouveau projet, vous pouvez utiliser le [modèle prêt à l'emploi sur Next.js](https://github.com/gravity-ui/page-constructor-website-template) que nous avons préparé.

### Générateur de sites statiques

[Page Constructor Builder](https://github.com/gravity-ui/page-constructor-builder) - utilitaire en ligne de commande pour générer des pages statiques à partir de configurations YAML en utilisant @gravity-ui/page-constructor

## Documentation

### Paramètres

```typescript
interface PageConstructorProps {
  content: PageContent; // Données des blocs au format JSON.
  shouldRenderBlock?: ShouldRenderBlock; // Une fonction invoquée lors du rendu de chaque bloc et qui permet de définir des conditions pour son affichage.
  custom?: Custom; // Blocs personnalisés (voir `Customization`).
  renderMenu?: () => React.ReactNode; // Une fonction qui rend le menu de la page avec la navigation (nous prévoyons d'ajouter le rendu pour la version par défaut du menu).
  navigation?: NavigationData; // Données de navigation pour utiliser le composant de navigation au format JSON
  isBranded?: boolean; // Si true, ajoute un pied de page qui renvoie vers https://gravity-ui.com/. Essayez le composant BrandFooter pour plus de personnalisation.
}

interface PageConstructorProviderProps {
  isMobile?: boolean; // Un indicateur signalant que le code s'exécute en mode mobile.
  locale?: LocaleContextProps; // Informations sur la langue et le domaine (utilisées lors de la génération et du formatage des liens).
  location?: Location; // API du navigateur ou de l'historique du routeur, l'URL de la page.
  analytics?: AnalyticsContextProps; // fonction pour gérer les événements d'analyse

  ssrConfig?: SSR; // Un indicateur signalant que le code s'exécute côté serveur.
  theme?: 'light' | 'dark'; // Thème pour rendre la page.
  mapsContext?: MapsContextType; // Paramètres pour la carte : apikey, type, scriptSrc, nonce
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

### Utilitaires serveur

Le package fournit un ensemble d'utilitaires serveur pour transformer votre contenu.

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

En arrière-plan, un package est utilisé pour transformer le Markdown aromatisé Yandex en HTML - `diplodoc/transform`, il fait donc également partie des dépendances peer.

Vous pouvez également utiliser des utilitaires pratiques là où vous en avez besoin, par exemple dans vos composants personnalisés.

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

Vous trouverez plus d'utilitaires dans cette [section](https://github.com/gravity-ui/page-constructor/tree/main/src/text-transform).

### Documentation détaillée sur les utilitaires serveur et les transformateurs

Pour un guide complet sur l'utilisation des utilitaires serveur, incluant des explications détaillées et des cas d'utilisation avancés, consultez le [chapitre supplémentaire sur l'utilisation des utilitaires serveur](./docs/data-preparation.md).

### Blocs personnalisés

Le constructeur de pages vous permet d'utiliser des blocs définis par l'utilisateur dans votre application. Les blocs sont des composants React ordinaires.

Pour passer des blocs personnalisés au constructeur :

1. Créez un bloc dans votre application.

2. Dans votre code, créez un objet avec le type de bloc (chaîne de caractères) comme clé et un composant de bloc importé comme valeur.

3. Passez l'objet créé au paramètre `custom.blocks`, `custom.headers` ou `custom.subBlocks` du composant `PageConstructor` (`custom.headers` spécifie les en-têtes de blocs à rendre séparément au-dessus du contenu général).

4. Vous pouvez maintenant utiliser le bloc créé dans les données d'entrée (le paramètre `content`) en spécifiant son type et ses données.

Pour utiliser des mixins et des variables de style du constructeur lors de la création de blocs personnalisés, ajoutez un import dans votre fichier :

```css
@import '~@gravity-ui/page-constructor/styles/styles.scss';
```

Pour utiliser la police par défaut, ajoutez un import dans votre fichier :

```css
@import '~@gravity-ui/page-constructor/styles/fonts.scss';
```

### Blocs chargeables

Il est parfois nécessaire qu'un bloc se rende lui-même en fonction de données à charger. Dans ce cas, on utilise des blocs chargeables.

Pour ajouter des blocs `loadable` personnalisés, passez à `PageConstructor` la propriété `custom.loadable` avec les noms des sources de données (chaîne de caractères) pour le composant comme clé et un objet comme valeur.

```typescript
export interface LoadableConfigItem {
  fetch: FetchLoadableData; // méthode de chargement des données
  component: React.ComponentType; // pour passer les données chargées
}

type FetchLoadableData<TData = any> = (blockKey: string) => Promise<TData>;
```

### Grille

Le constructeur de pages utilise la grille `bootstrap` et son implémentation basée sur des composants React que vous pouvez utiliser dans votre propre projet (y compris séparément du constructeur).

Exemple d'utilisation :

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

La navigation des pages peut également être utilisée séparément du constructeur :

```jsx
import {Navigation} from '@gravity-ui/page-constructor';

const Page= ({data, logo}: React.PropsWithChildren<PageProps>) => <Navigation data={data} logo={logo} />;
```

### Blocs

Chaque bloc est un composant de niveau supérieur atomique. Ils sont stockés dans le répertoire `src/units/constructor/blocks`.

### Sous-blocs

Les sous-blocs sont des composants qui peuvent être utilisés dans la propriété `children` d'un bloc. Dans une configuration, une liste de composants enfants provenant de sous-blocs est spécifiée. Une fois rendus, ces sous-blocs sont passés au bloc en tant que `children`.

### Comment ajouter un nouveau bloc au `page-constructor`

1. Dans le répertoire `src/blocks` ou `src/sub-blocks`, créez un dossier avec le code du bloc ou du sous-bloc.

2. Ajoutez le nom du bloc ou du sous-bloc à l'enum `BlockType` ou `SubBlockType` et décrivez ses propriétés dans le fichier `src/models/constructor-items/blocks.ts` ou `src/models/constructor-items/sub-blocks.ts` de manière similaire aux existants.

3. Ajoutez l'export pour le bloc dans le fichier `src/blocks/index.ts` et pour le sous-bloc dans le fichier `src/sub-blocks/index.ts`.

4. Ajoutez un nouveau composant ou bloc au mapping dans `src/constructor-items.ts`.

5. Ajoutez un validateur pour le nouveau bloc :

   - Ajoutez un fichier `schema.ts` dans le répertoire du bloc ou du sous-bloc. Dans ce fichier, décrivez un validateur de paramètres pour le composant au format [`json-schema`](http://json-schema.org/).
   - Exportez-le dans le fichier `schema/validators/blocks.ts` ou `schema/validators/sub-blocks.ts`.
   - Ajoutez-le à `enum` ou `selectCases` dans le fichier `schema/index.ts`.

6. Dans le répertoire du bloc, ajoutez le fichier `README.md` avec une description des paramètres d'entrée.
7. Dans le répertoire du bloc, ajoutez une démo Storybook dans le dossier `__stories__`. Tout le contenu de la démo pour l'histoire doit être placé dans `data.json` dans le répertoire de l'histoire. L'histoire générique `Story` doit accepter le type des props du bloc, sinon des props de bloc incorrects seront affichés dans Storybook.
8. Ajoutez un modèle de données de bloc au dossier `src/editor/data/templates/`, le nom du fichier doit correspondre au type de bloc.
9. (optionnel) Ajoutez une icône d'aperçu de bloc au dossier `src/editor/data/previews/`, le nom du fichier doit correspondre au type de bloc.

### Thèmes

Le `PageConstructor` vous permet d'utiliser des thèmes : vous pouvez définir des valeurs différentes pour les propriétés individuelles d'un bloc en fonction du thème sélectionné dans l'application.

Pour ajouter un thème à une propriété de bloc :

1. Dans le fichier `models/blocks.ts`, définissez le type de la propriété de bloc respective en utilisant le générique `ThemeSupporting<T>`, où `T` est le type de la propriété.

2. Dans le fichier avec le composant `react` du bloc, obtenez la valeur de la propriété avec le thème via `getThemedValue` et le hook `useTheme` (voir les exemples dans le bloc `MediaBlock.tsx`).

3. Ajoutez le support des thèmes au validateur de propriété : dans le fichier `schema.ts` du bloc, enveloppez cette propriété dans `withTheme`.

### i18n

Le `page-constructor` est une bibliothèque basée sur `uikit`, et nous utilisons une instance de `i18n` provenant de uikit. Pour configurer l'internationalisation, il suffit d'utiliser `configure` de uikit :

```typescript
import {configure} from '@gravity-ui/uikit';

configure({
  lang: 'ru',
});
```

### Cartes

Pour utiliser des cartes, placez le type de carte, `scriptSrc` et `apiKey` dans le champ `mapContext` du `PageConstructorProvider`.

Vous pouvez définir des variables d'environnement pour le mode dev dans le fichier `.env.development` à la racine du projet.
`STORYBOOK_GMAP_API_KEY` - apiKey pour Google Maps.

### Analytics

#### Initialisation

Pour commencer à utiliser n'importe quel analytics, passez un gestionnaire au constructeur. Le gestionnaire doit être créé du côté du projet. Le gestionnaire recevra les objets d'événements `default` et `custom`. Le gestionnaire passé sera déclenché lors des clics sur les boutons, les liens, la navigation et les contrôles. Comme un seul gestionnaire est utilisé pour le traitement de tous les événements, prêtez attention à la façon de traiter les différents événements lors de la création du gestionnaire. Il existe des champs prédéfinis qui servent à vous aider à construire une logique complexe.

Passez `autoEvents: true` au constructeur pour déclencher automatiquement les événements configurés.

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

Un objet d'événement ne comporte qu'un seul champ obligatoire : `name`. Il possède également des champs prédéfinis qui aident à gérer une logique complexe. Par exemple, `counter.include` permet d'envoyer un événement vers un compteur spécifique si plusieurs systèmes d'analyse sont utilisés dans un projet.

```ts
type AnalyticsEvent<T = {}> = T & {
  name: string;
  type?: string;
  counters?: AnalyticsCounters;
  context?: string;
};
```

Il est possible de configurer un type d'événement adapté aux besoins du projet.

```ts
type MyEventType = AnalyticsEvent<{
  [key: string]?: string; // seul le type 'string' est pris en charge
}>;
```

#### Sélecteur de compteur

Il est possible de configurer un événement pour spécifier vers quel système d'analyse il doit être envoyé.

```ts
type AnalyticsCounters = {
  include?: string[]; // tableau d'ID de compteurs d'analyse qui seront appliqués
  exclude?: string[]; // tableau d'ID de compteurs d'analyse qui ne seront pas appliqués
};
```

#### Paramètre context

Passez une valeur `context` pour définir l'endroit dans le projet où l'événement est déclenché.

Utilisez le sélecteur ci-dessous ou créez une logique adaptée aux besoins du projet.

```ts
// analyticsHandler.ts
if (isCounterAllowed(counterName, counters)) {
  analyticsCounter.reachGoal(counterName, name, parameters);
}
```

#### Types d'événements réservés

Plusieurs types d'événements prédéfinis sont utilisés pour marquer les événements configurés automatiquement. Utilisez ces types pour filtrer les événements par défaut, par exemple.

```ts
enum PredefinedEventTypes {
  Default = 'default-event', // événements par défaut qui se déclenchent sur chaque clic de bouton
  Play = 'play', // événement du lecteur React
  Stop = 'stop', // événement du lecteur React
}
```

## Développement

```bash
npm ci
npm run dev
```

#### Note sur Vite

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

Pour Vite, vous devez installer le plugin `vite-plugin-dynamic-import` et configurer le fichier de configuration afin que les imports dynamiques fonctionnent.

## Flux de publication

Dans la plupart des cas, nous utilisons deux types de commits :

1. fix : un commit de type fix corrige un bug dans le code (cela correspond à PATCH dans Semantic Versioning).
2. feat : un commit de type feat introduit une nouvelle fonctionnalité dans le code (cela correspond à MINOR dans Semantic Versioning).
3. BREAKING CHANGE : un commit qui comporte un pied de page BREAKING CHANGE:, ou qui ajoute un ! après le type/scope, introduit un changement d'API rompant (correspondant à MAJOR dans Semantic Versioning). Un BREAKING CHANGE peut faire partie de commits de n'importe quel type.
4. Pour définir manuellement la version du package de publication, ajoutez `Release-As: <version>` à votre message de commit, par exemple.

```bash
git commit -m 'chore: bump release

Release-As: 1.2.3'
```

Vous pouvez trouver toutes les informations [ici](https://www.conventionalcommits.org/en/v1.0.0/).

Lorsque vous obtenez l'approbation de votre pull-request par les propriétaires du code et que vous passez tous les contrôles, veuillez procéder comme suit :

1. Vérifiez s'il existe une pull-request de publication créée par le robot avec les changements d'un autre contributeur (elle ressemble à `chore(main): release 0.0.0`). Si elle existe, vérifiez pourquoi elle n'a pas été fusionnée. Si le contributeur accepte de publier une version partagée, passez à l'étape suivante. Sinon, demandez-lui de publier sa version, puis passez à l'étape suivante.
2. Fusionnez votre PR en squash (Il est important de publier une nouvelle version avec Github-Actions)
3. Attendez que le robot crée une PR avec une nouvelle version du package et des informations sur vos changements dans CHANGELOG.md. Vous pouvez suivre le processus sur [l'onglet Actions](https://github.com/gravity-ui/page-constructor/actions).
4. Vérifiez vos changements dans CHANGELOG.md et approuvez la PR du robot.
5. Fusionnez la PR en squash. Vous pouvez suivre le processus de publication sur [l'onglet Actions](https://github.com/gravity-ui/page-constructor/actions).

### Publication de versions alpha

Si vous souhaitez publier une version alpha du package depuis votre branche, vous pouvez le faire manuellement :

1. Allez dans l'onglet Actions
2. Sélectionnez le workflow "Release alpha version" sur le côté gauche de la page
3. Vous verrez sur le côté droit le bouton "Run workflow". Ici, vous pouvez choisir la branche.
4. Vous verrez également un champ pour la version manuelle. Si vous publiez une alpha depuis votre branche pour la première fois, ne définissez rien ici. Après la première publication, vous devrez définir la nouvelle version manuellement car nous ne modifions pas package.json au cas où la branche expirerait très bientôt. Utilisez le préfixe `alpha` dans votre version manuelle, sinon vous obtiendrez une erreur.
5. Cliquez sur "Run workflow" et attendez que l'action se termine. Vous pouvez publier autant de versions que vous le souhaitez, mais n'en abusez pas et publiez uniquement si vous en avez vraiment besoin. Dans les autres cas, utilisez [npm pack](https://docs.npmjs.com/cli/v7/commands/npm-pack).

### Publication de versions beta-major

Si vous souhaitez publier une nouvelle version majeure, vous aurez probablement besoin de versions beta avant une version stable, veuillez procéder comme suit :

1. Créez ou mettez à jour la branche `beta`.
2. Ajoutez-y vos changements.
3. Lorsque vous êtes prêt pour une nouvelle version beta, publiez-la manuellement avec un commit vide (ou vous pouvez ajouter ce message de commit avec un pied de page au dernier commit) :

```bash
git commit -m 'fix: last commit

Release-As: 3.0.0-beta.0' --allow-empty
```

4. Le robot créera une nouvelle PR vers la branche `beta` avec un CHANGELOG.md mis à jour et une version du package incrémentée
5. Vous pouvez répéter cela autant de fois que vous le souhaitez. Lorsque vous êtes prêt à publier la dernière version majeure sans le tag beta, vous devez créer une PR depuis la branche `beta` vers la branche `main`. Notez qu'il est normal que la version de votre package comporte un tag beta. Le robot le sait et le modifie correctement. `3.0.0-beta.0` deviendra `3.0.0`

### Flux de publication pour les versions majeures précédentes

Si vous souhaitez publier une nouvelle version dans une version majeure précédente après l'avoir committée dans main, veuillez procéder comme suit :

1. Mettez à jour la branche nécessaire, les noms des branches de publication des versions majeures précédentes sont :
   1. `version-1.x.x/fixes` - pour la version majeure 1.x.x
   2. `version-2.x.x` - pour la version majeure 2.x.x
2. Créez une nouvelle branche à partir de la branche de publication de la version majeure précédente
3. Cherry-pick votre commit depuis la branche `main`
4. Créez une PR, obtenez une approbation et fusionnez dans la branche de publication de la version majeure précédente
5. Fusionnez votre PR en squash (Il est important de publier une nouvelle version avec Github-Actions)
6. Attendez que le robot crée une PR avec une nouvelle version du package et des informations sur vos changements dans CHANGELOG.md. Vous pouvez suivre le processus sur [l'onglet Actions](https://github.com/gravity-ui/page-constructor/actions).
7. Vérifiez vos changements dans CHANGELOG.md et approuvez la PR du robot.
8. Fusionnez la PR en squash. Vous pouvez suivre le processus de publication sur [l'onglet Actions](https://github.com/gravity-ui/page-constructor/actions).

## Éditeur de constructeur de page

L'éditeur fournit une interface utilisateur pour la gestion du contenu des pages avec une prévisualisation en temps réel.

Comment l'utiliser :

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

Ce projet inclut une **Memory Bank** complète - une collection de fichiers de documentation Markdown qui fournissent des informations détaillées sur l'architecture du projet, les composants et les patterns d'utilisation. La Memory Bank est particulièrement utile lors du travail avec des agents IA, car elle contient des informations structurées sur :

- **Aperçu du Projet** : Exigences principales, objectifs et contexte
- **Documentation des Composants** : Guides d'utilisation détaillés pour tous les composants
- **Architecture du Système** : Patterns techniques et décisions de design
- **Progrès du Développement** : Statut actuel et détails d'implémentation

### Utilisation de la Memory Bank

La Memory Bank se trouve dans le répertoire `memory-bank/` et consiste en des fichiers Markdown ordinaires qui peuvent être lus comme n'importe quelle autre documentation :

- `projectbrief.md` - Document de fondation avec les exigences principales
- `productContext.md` - Objectif du projet et objectifs d'expérience utilisateur
- `systemPatterns.md` - Architecture et décisions techniques
- `techContext.md` - Technologies, configuration et contraintes
- `activeContext.md` - Focus de travail actuel et changements récents
- `progress.md` - Statut d'implémentation et problèmes connus
- `usage/` - Documentation d'utilisation spécifique aux composants
- `storybookComponents.md` - Détails d'intégration de Storybook

### Pour les Agents IA

Lors du travail avec des agents IA sur ce projet, la Memory Bank sert de base de connaissances complète qui aide les agents à comprendre :

- Structure et patterns du projet
- APIs des composants et exemples d'utilisation
- Flux de travail de développement et meilleures pratiques
- Statut d'implémentation actuel et étapes suivantes

Les agents IA peuvent lire ces fichiers pour se mettre rapidement au courant du contexte du projet et prendre des décisions plus informées concernant les changements de code et les implémentations.

## Tests

Une documentation complète est disponible au lien fourni [ici](./test-utils/docs/README.md).