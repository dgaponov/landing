# Data Source &middot; [![npm version](https://img.shields.io/npm/v/@gravity-ui/data-source?logo=npm&label=version)](https://www.npmjs.com/package/@gravity-ui/data-source) [![ci](https://img.shields.io/github/actions/workflow/status/gravity-ui/data-source/ci.yml?branch=main&label=ci&logo=github)](https://github.com/gravity-ui/data-source/actions/workflows/ci.yml?query=branch:main)

**Data Source** est un simple wrapper autour de la récupération de données. Il s'agit d'une sorte de « port » dans l'[architecture propre](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html). Il vous permet de créer des wrappers pour les aspects liés à la récupération de données en fonction de vos cas d'utilisation. **Data Source** utilise [react-query](https://tanstack.com/query/latest) en arrière-plan.

## Installation

```bash
npm install @gravity-ui/data-source @tanstack/react-query
```

`@tanstack/react-query` est une dépendance peer.

## Démarrage rapide

### 1. Configuration de DataManager

D'abord, créez et fournissez un `DataManager` dans votre application :

```tsx
import React from 'react';
import {ClientDataManager, DataManagerContext} from '@gravity-ui/data-source';

const dataManager = new ClientDataManager({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      retry: 3,
    },
    // ... autres options react-query
  },
});

function App() {
  return (
    <DataManagerContext.Provider value={dataManager}>
      <YourApplication />
    </DataManagerContext.Provider>
  );
}
```

### 2. Définition des types d'erreurs et des wrappers

Définissez un type d'erreur et créez vos constructeurs pour les sources de données basés sur les constructeurs par défaut :

```ts
import {makePlainQueryDataSource as makePlainQueryDataSourceBase} from '@gravity-ui/data-source';

export interface ApiError {
  code: number;
  title: string;
  description?: string;
}

export const makePlainQueryDataSource = <TParams, TRequest, TResponse, TData, TError = ApiError>(
  config: Omit<PlainQueryDataSource<TParams, TRequest, TResponse, TData, TError>, 'type'>,
): PlainQueryDataSource<TParams, TRequest, TResponse, TData, TError> => {
  return makePlainQueryDataSourceBase(config);
};
```

### 3. Création d'un composant DataLoader personnalisé

Écrivez un composant `DataLoader` basé sur le composant par défaut pour définir l'affichage de l'état de chargement et des erreurs :

```tsx
import {
  DataLoader as DataLoaderBase,
  DataLoaderProps as DataLoaderPropsBase,
  ErrorViewProps,
} from '@gravity-ui/data-source';

export interface DataLoaderProps
  extends Omit<DataLoaderPropsBase<ApiError>, 'LoadingView' | 'ErrorView'> {
  LoadingView?: ComponentType;
  ErrorView?: ComponentType<ErrorViewProps<ApiError>>;
}

export const DataLoader: React.FC<DataLoaderProps> = ({
  LoadingView = YourLoader, // Vous pouvez utiliser votre propre composant de chargement
  ErrorView = YourError, // Vous pouvez utiliser votre propre composant d'erreur
  ...restProps
}) => {
  return <DataLoaderBase LoadingView={LoadingView} ErrorView={ErrorView} {...restProps} />;
};
```

### 4. Définition de votre première source de données

```ts
import {skipContext} from '@gravity-ui/data-source';

// Votre fonction API
import {fetchUser} from './api';

export const userDataSource = makePlainQueryDataSource({
  // Les clés doivent être uniques. Peut-être devriez-vous créer un helper pour générer les noms des sources de données
  name: 'user',
  // skipContext est un helper pour ignorer les 2 premiers paramètres de la fonction (context et fetchContext)
  fetch: skipContext(fetchUser),
  // Optionnel : générer des tags pour une invalidation avancée du cache
  tags: (params) => [`user:${params.userId}`, 'users'],
});
```

### 5. Utilisation dans les composants

```tsx
import {useQueryData} from '@gravity-ui/data-source';

export const UserProfile: React.FC<{userId: number}> = ({userId}) => {
  const {data, status, error, refetch} = useQueryData(userDataSource, {userId});

  return (
    <DataLoader status={status} error={error} errorAction={refetch}>
      {data && <UserCard user={data} />}
    </DataLoader>
  );
};
```

## Concepts de base

### Types de sources de données

La bibliothèque fournit deux principaux types de sources de données :

#### Source de données de requête simple

Pour les patterns de requête/réponse simples :

```ts
const userDataSource = makePlainQueryDataSource({
  name: 'user',
  fetch: skipContext(async (params: {userId: number}) => {
    const response = await fetch(`/api/users/${params.userId}`);
    return response.json();
  }),
});
```

#### Source de données de requête infinie

Pour la pagination et le défilement infini :

```ts
const postsDataSource = makeInfiniteQueryDataSource({
  name: 'posts',
  fetch: skipContext(async (params: {page: number; limit: number}) => {
    const response = await fetch(`/api/posts?page=${params.page}&limit=${params.limit}`);
    return response.json();
  }),
  next: (lastPage, allPages) => {
    if (lastPage.hasNext) {
      return {page: allPages.length + 1, limit: 20};
    }
    return undefined;
  },
});
```

### Gestion des états

La bibliothèque normalise les états des requêtes en trois états simples :

- `loading` - Chargement réel des données. Identique à `isLoading` dans React Query
- `success` - Données disponibles (peut être ignoré en utilisant idle)
- `error` - Échec de la récupération des données

### Concept d'idle

La bibliothèque fournit un symbole spécial `idle` pour ignorer l'exécution d'une requête :

```ts
import {idle} from '@gravity-ui/data-source';

const UserProfile: React.FC<{userId?: number}> = ({userId}) => {
  // La requête ne s'exécutera pas si userId n'est pas défini
  const {data, status} = useQueryData(userDataSource, userId ? {userId} : idle);

  return (
    <DataLoader status={status} error={null}>
      {data && <UserCard user={data} />}
    </DataLoader>
  );
};
```

Lorsque les paramètres sont égaux à `idle` :

- La requête ne s'exécute pas
- L'état reste `success`
- Les données restent `undefined`
- Le composant peut s'afficher en toute sécurité sans chargement

**Avantages de `idle` :**

1. **Sécurité des types** - TypeScript infère correctement les types pour les paramètres conditionnels
2. **Performance** - Évite les requêtes inutiles vers le serveur
3. **Simplicité de la logique** - Pas besoin de gérer un état `enabled` supplémentaire
4. **Cohérence** - Approche unifiée pour toutes les requêtes conditionnelles

Ceci est particulièrement utile pour les requêtes conditionnelles lorsque vous souhaitez charger les données uniquement dans certaines conditions tout en maintenant la sécurité des types.

## Référence API

### Création de sources de données

#### `makePlainQueryDataSource(config)`

Crée une source de données de requête simple pour les patterns de requête/réponse simples.

```ts
const dataSource = makePlainQueryDataSource({
  name: 'unique-name',
  fetch: skipContext(fetchFunction),
  transformParams: (params) => transformedRequest,
  transformResponse: (response) => transformedData,
  tags: (params) => ['tag1', 'tag2'],
  options: {
    staleTime: 60000,
    retry: 3,
    // ... other react-query options
  },
});
```

**Paramètres :**

- `name` - Identifiant unique pour la source de données
- `fetch` - Fonction qui effectue la récupération réelle des données
- `transformParams` (optionnel) - Transformer les paramètres d'entrée avant la requête
- `transformResponse` (optionnel) - Transformer les données de réponse
- `tags` (optionnel) - Générer des étiquettes de cache pour l'invalidation
- `options` (optionnel) - Options React Query

#### `makeInfiniteQueryDataSource(config)`

Crée une source de données de requête infinie pour les motifs de pagination et de défilement infini.

```ts
const infiniteDataSource = makeInfiniteQueryDataSource({
  name: 'infinite-data',
  fetch: skipContext(fetchFunction),
  next: (lastPage, allPages) => nextPageParam || undefined,
  prev: (firstPage, allPages) => prevPageParam || undefined,
  // ... other options same as plain
});
```

**Paramètres supplémentaires :**

- `next` - Fonction pour déterminer les paramètres de la page suivante
- `prev` (optionnel) - Fonction pour déterminer les paramètres de la page précédente

### Crochets React

#### `useQueryData(dataSource, params, options?)`

Crochet principal pour récupérer des données avec une source de données.

```ts
const {data, status, error, refetch, ...rest} = useQueryData(
  userDataSource,
  {userId: 123},
  {
    enabled: true,
    refetchInterval: 30000,
  },
);
```

**Retours :**

- `data` - Les données récupérées
- `status` - État actuel ('loading' | 'success' | 'error')
- `error` - Objet d'erreur si la requête a échoué
- `refetch` - Fonction pour relancer manuellement la récupération des données
- Autres propriétés React Query

#### `useQueryResponses(responses)`

Combine plusieurs réponses de requêtes en un seul état.

```ts
const user = useQueryData(userDataSource, {userId});
const posts = useQueryData(postsDataSource, {userId});

const {status, error, refetch, refetchErrored} = useQueryResponses([user, posts]);
```

**Retours :**

- `status` - État combiné de toutes les requêtes
- `error` - Première erreur rencontrée
- `refetch` - Fonction pour relancer toutes les requêtes
- `refetchErrored` - Fonction pour relancer uniquement les requêtes échouées

#### `useRefetchAll(states)`

Crée un rappel pour relancer plusieurs requêtes.

```ts
const refetchAll = useRefetchAll([user, posts, comments]);
// refetchAll() déclenchera la relance pour toutes les requêtes
```

#### `useRefetchErrored(states)`

Crée un rappel pour relancer uniquement les requêtes échouées.

```ts
const refetchErrored = useRefetchErrored([user, posts, comments]);
// refetchErrored() ne relancera que les requêtes avec des erreurs
```

#### `useDataManager()`

Retourne le DataManager depuis le contexte.

```ts
const dataManager = useDataManager();
await dataManager.invalidateTag('users');
```

#### `useQueryContext()`

Retourne le contexte de requête (pour créer des crochets de données personnalisés basés sur react-query).

### Composants React

#### `<DataLoader />`

Composant pour gérer les états de chargement et les erreurs.

```tsx
<DataLoader
  status={status}
  error={error}
  errorAction={refetch}
  LoadingView={SpinnerComponent}
  ErrorView={ErrorComponent}
  loadingViewProps={{size: 'large'}}
  errorViewProps={{showDetails: true}}
>
  {data && <YourContent data={data} />}
</DataLoader>
```

**Props :**

- `status` - État de chargement actuel
- `error` - Objet d'erreur
- `errorAction` - Fonction ou configuration d'action pour réessayer en cas d'erreur
- `LoadingView` - Composant à afficher pendant le chargement
- `ErrorView` - Composant à afficher en cas d'erreur
- `loadingViewProps` - Props passés à LoadingView
- `errorViewProps` - Props passés à ErrorView

#### `<DataInfiniteLoader />`

Composant spécialisé pour les requêtes infinies.

```tsx
<DataInfiniteLoader
  status={status}
  error={error}
  hasNextPage={hasNextPage}
  fetchNextPage={fetchNextPage}
  isFetchingNextPage={isFetchingNextPage}
  LoadingView={SpinnerComponent}
  ErrorView={ErrorComponent}
  MoreView={LoadMoreButton}
>
  {data.map((item) => (
    <Item key={item.id} data={item} />
  ))}
</DataInfiniteLoader>
```

**Props supplémentaires :**

- `hasNextPage` - Indique si d'autres pages sont disponibles
- `fetchNextPage` - Fonction pour récupérer la page suivante
- `isFetchingNextPage` - Indique si la page suivante est en cours de récupération
- `MoreView` - Composant pour le bouton « charger plus »

#### `withDataManager(Component)`

HOC qui injecte DataManager en tant que prop.

```tsx
const MyComponent = withDataManager<Props>(({dataManager, ...props}) => {
  // Le composant a accès à dataManager
  return <div>...</div>;
});
```

### Gestion des données

#### `ClientDataManager`

Classe principale pour la gestion des données.

```ts
const dataManager = new ClientDataManager({
  defaultOptions: {
    queries: {
      staleTime: 300000, // 5 minutes
      retry: 3,
      refetchOnWindowFocus: false,
    },
  },
});
```

**Méthodes :**

##### `invalidateTag(tag, options?)`

Invalide toutes les requêtes avec une étiquette spécifique.

```ts
await dataManager.invalidateTag('users');
await dataManager.invalidateTag('posts', {
  repeat: {count: 3, interval: 1000}, // Réessayer l'invalidation
});
```

##### `invalidateTags(tags, options?)`

Invalide les requêtes qui ont toutes les étiquettes spécifiées.

```ts
await dataManager.invalidateTags(['user', 'profile']);
```

##### `invalidateSource(dataSource, options?)`

Invalide toutes les requêtes pour une source de données.

```ts
await dataManager.invalidateSource(userDataSource);
```

##### `invalidateParams(dataSource, params, options?)`

Invalide une requête spécifique avec des paramètres exacts.

```ts
await dataManager.invalidateParams(userDataSource, {userId: 123});
```

##### `resetSource(dataSource)`

Réinitialise (efface) toutes les données mises en cache pour une source de données.

```ts
await dataManager.resetSource(userDataSource);
```

##### `resetParams(dataSource, params)`

Réinitialise les données mises en cache pour des paramètres spécifiques.

```ts
await dataManager.resetParams(userDataSource, {userId: 123});
```

##### `invalidateSourceTags(dataSource, params, options?)`

Invalide les requêtes basées sur les étiquettes générées par une source de données.

```ts
await dataManager.invalidateSourceTags(userDataSource, {userId: 123});
```

### Utilitaires

#### `skipContext(fetchFunction)`

Utilitaire pour adapter les fonctions de récupération existantes à l'interface de source de données.

```ts
// Fonction existante
async function fetchUser(params: {userId: number}) {
  // ...
}

// Adaptée pour la source de données
const dataSource = makePlainQueryDataSource({
  name: 'user',
  fetch: skipContext(fetchUser), // Ignore le contexte et les params fetchContext
});
```

#### `withCatch(fetchFunction, errorHandler)`

Ajoute une gestion d'erreurs standardisée aux fonctions fetch.

```ts
const safeFetch = withCatch(fetchUser, (error) => ({error: true, message: error.message}));
```

#### `withCancellation(fetchFunction)`

Ajoute un support d'annulation aux fonctions fetch.

```ts
const cancellableFetch = withCancellation(fetchFunction);
// Gère automatiquement AbortSignal depuis React Query
```

#### `getProgressiveRefetch(options)`

Crée une fonction d'intervalle de relecture progressive.

```ts
const progressiveRefetch = getProgressiveRefetch({
  minInterval: 1000, // Commence par 1 seconde
  maxInterval: 30000, // Maximum 30 secondes
  multiplier: 2, // Double à chaque fois
});

const dataSource = makePlainQueryDataSource({
  name: 'data',
  fetch: skipContext(fetchData),
  options: {
    refetchInterval: progressiveRefetch,
  },
});
```

#### `normalizeStatus(status, fetchStatus)`

Convertit les statuts React Query en statuts DataLoader.

```ts
const status = normalizeStatus('pending', 'fetching'); // 'loading'
```

#### Utilitaires pour les statuts et les erreurs

```ts
// Obtient un statut combiné à partir de plusieurs états
const status = getStatus([user, posts, comments]);

// Obtient la première erreur à partir de plusieurs états
const error = getError([user, posts, comments]);

// Fusionne plusieurs statuts
const combinedStatus = mergeStatuses(['loading', 'success', 'error']); // 'error'

// Vérifie si une clé de requête possède un tag
const hasUserTag = hasTag(queryKey, 'users');
```

#### Utilitaires pour la composition de clés

```ts
// Compose une clé de cache pour une source de données
const key = composeKey(userDataSource, {userId: 123});

// Compose une clé complète incluant les tags
const fullKey = composeFullKey(userDataSource, {userId: 123});
```

#### Constantes

```ts
import {idle} from '@gravity-ui/data-source';

// Symbole spécial pour ignorer l'exécution de la requête
const params = shouldFetch ? {userId: 123} : idle;

// Alternative type-safe à enabled: false
// Au lieu de :
const {data} = useQueryData(userDataSource, {userId: userId || ''}, {enabled: Boolean(userId)});

// Utilisez :
const {data} = useQueryData(userDataSource, userId ? {userId} : idle);
// TypeScript infère correctement les types pour les deux branches
```

#### Composition d'options de requête

```ts
// Compose les options React Query pour les requêtes simples
const plainOptions = composePlainQueryOptions(context, dataSource, params, options);

// Compose les options React Query pour les requêtes infinies
const infiniteOptions = composeInfiniteQueryOptions(context, dataSource, params, options);
```

**Note :** Ces fonctions sont principalement destinées à un usage interne lors de la création d'implémentations personnalisées de sources de données.

## Modèles avancés

### Requêtes conditionnelles avec Idle

Utilisez `idle` pour créer des requêtes conditionnelles :

```ts
import {idle} from '@gravity-ui/data-source';

const ConditionalDataComponent: React.FC<{
  userId?: number;
  shouldLoadPosts: boolean;
}> = ({userId, shouldLoadPosts}) => {
  // Charge l'utilisateur uniquement si userId est défini
  const user = useQueryData(
    userDataSource,
    userId ? {userId} : idle
  );

  // Charge les posts uniquement si l'utilisateur est chargé et que le drapeau est activé
  const posts = useQueryData(
    userPostsDataSource,
    user.data && shouldLoadPosts ? {userId: user.data.id} : idle
  );

  const combined = useQueryResponses([user, posts]);

  return (
    <DataLoader status={combined.status} error={combined.error}>
      <div>
        {user.data && <UserInfo user={user.data} />}
        {posts.data && <UserPosts posts={posts.data} />}
      </div>
    </DataLoader>
  );
};
```

### Transformation des données

Transformez les paramètres de requête et les données de réponse :

```ts
const apiDataSource = makePlainQueryDataSource({
  name: 'api-data',
  transformParams: (params: {id: number}) => ({
    userId: params.id,
    apiVersion: 'v2',
    format: 'json',
  }),
  transformResponse: (response: ApiResponse) => ({
    user: response.data.user,
    metadata: response.meta,
  }),
  fetch: skipContext(apiFetch),
});
```

### Invalidation de cache basée sur les tags

Utilisez des tags pour une gestion sophistiquée du cache :

```ts
const userDataSource = makePlainQueryDataSource({
  name: 'user',
  tags: (params) => [`user:${params.userId}`, 'users', 'profiles'],
  fetch: skipContext(fetchUser),
});

const userPostsDataSource = makePlainQueryDataSource({
  name: 'user-posts',
  tags: (params) => [`user:${params.userId}`, 'posts'],
  fetch: skipContext(fetchUserPosts),
});

// Invalide toutes les données pour un utilisateur spécifique
await dataManager.invalidateTag('user:123');

// Invalide toutes les données liées aux utilisateurs
await dataManager.invalidateTag('users');
```

### Gestion d'erreurs avec types

Créez une gestion d'erreurs type-safe :

```ts
interface ApiError {
  code: number;
  message: string;
  details?: Record<string, unknown>;
}

const ErrorView: React.FC<ErrorViewProps<ApiError>> = ({error, action}) => (
  <div className="error">
    <h3>Error {error?.code}</h3>
    <p>{error?.message}</p>
    {action && (
      <button onClick={action.handler}>
        {action.children || 'Retry'}
      </button>
    )}
  </div>
);
```

### Requêtes infinies avec pagination complexe

Gérez des scénarios de pagination complexes :

```ts
interface PaginationParams {
  cursor?: string;
  limit?: number;
  filters?: Record<string, unknown>;
}

interface PaginatedResponse<T> {
  data: T[];
  nextCursor?: string;
  hasMore: boolean;
}

const infiniteDataSource = makeInfiniteQueryDataSource({
  name: 'paginated-data',
  fetch: skipContext(async (params: PaginationParams) => {
    const response = await fetch(`/api/data?${new URLSearchParams(params)}`);
    return response.json() as PaginatedResponse<DataItem>;
  }),
  next: (lastPage) => {
    if (lastPage.hasMore && lastPage.nextCursor) {
      return {cursor: lastPage.nextCursor, limit: 20};
    }
    return undefined;
  },
});
```

### Combinaison de plusieurs sources de données

Combinez des données provenant de plusieurs sources :

```ts
const UserProfile: React.FC<{userId: number}> = ({userId}) => {
  const user = useQueryData(userDataSource, {userId});
  const posts = useQueryData(userPostsDataSource, {userId});
  const followers = useQueryData(userFollowersDataSource, {userId});

  const combined = useQueryResponses([user, posts, followers]);

```

```tsx
return (
  <DataLoader
    status={combined.status}
    error={combined.error}
    errorAction={combined.refetchErrored} // Only retry failed requests
    LoadingView={ProfileSkeleton}
    ErrorView={ProfileError}
  >
    {user && posts && followers && (
      <div>
        <UserInfo user={user.data} />
        <UserPosts posts={posts.data} />
        <UserFollowers followers={followers.data} />
      </div>
    )}
  </DataLoader>
);
};
```

## Support TypeScript

La bibliothèque est construite avec une approche TypeScript en premier et fournit une inférence de type complète :

```ts
// Les types sont automatiquement inférés
const userDataSource = makePlainQueryDataSource({
  name: 'user',
  fetch: skipContext(async (params: {userId: number}): Promise<User> => {
    // Le type de retour est inféré comme User
  }),
});

// Le type de retour du hook est automatiquement typé
const {data} = useQueryData(userDataSource, {userId: 123});
// data est typé comme User | undefined
```

### Types d'erreurs personnalisés

Définissez et utilisez des types d'erreurs personnalisés :

```ts
interface ValidationError {
  field: string;
  message: string;
}

interface ApiError {
  type: 'network' | 'validation' | 'server';
  message: string;
  validation?: ValidationError[];
}

const typedDataSource = makePlainQueryDataSource<
  {id: number}, // Type des paramètres
  {id: number}, // Type de la requête
  ApiResponse, // Type de la réponse
  User, // Type des données
  ApiError // Type d'erreur
>({
  name: 'typed-user',
  fetch: skipContext(fetchUser),
});
```

## Contribution

Veuillez lire [CONTRIBUTING.md](CONTRIBUTING.md) pour plus de détails sur notre code de conduite et le processus de soumission des demandes de tirage.

## Licence

Licence MIT. Voir le fichier [LICENSE](LICENSE) pour plus de détails.