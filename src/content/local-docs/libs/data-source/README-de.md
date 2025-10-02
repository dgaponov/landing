# Data Source · [![npm version](https://img.shields.io/npm/v/@gravity-ui/data-source?logo=npm&label=version)](https://www.npmjs.com/package/@gravity-ui/data-source) [![ci](https://img.shields.io/github/actions/workflow/status/gravity-ui/data-source/ci.yml?branch=main&label=ci&logo=github)](https://github.com/gravity-ui/data-source/actions/workflows/ci.yml?query=branch:main)

**Data Source** ist ein einfacher Wrapper um das Abrufen von Daten. Es handelt sich um eine Art „Port“ in der [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html). Es ermöglicht Ihnen, Wrapper für alles rund um das Abrufen von Daten je nach Ihren Anwendungsfällen zu erstellen. **Data Source** verwendet [react-query](https://tanstack.com/query/latest) intern.

## Installation

```bash
npm install @gravity-ui/data-source @tanstack/react-query
```

`@tanstack/react-query` ist eine Peer-Dependency.

## Schnellstart

### 1. DataManager einrichten

Erstellen Sie zunächst einen `DataManager` und stellen Sie ihn in Ihrer Anwendung bereit:

```tsx
import React from 'react';
import {ClientDataManager, DataManagerContext} from '@gravity-ui/data-source';

const dataManager = new ClientDataManager({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 Minuten
      retry: 3,
    },
    // ... andere react-query-Optionen
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

### 2. Fehler-Typen und Wrapper definieren

Definieren Sie einen Fehler-Typ und erstellen Sie Ihre Konstruktoren für Data Sources basierend auf den Standard-Konstruktoren:

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

### 3. Benutzerdefiniertes DataLoader-Komponent erstellen

Schreiben Sie eine `DataLoader`-Komponente basierend auf der Standardvariante, um die Anzeige von Lade-Status und Fehlern zu definieren:

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
  LoadingView = YourLoader, // Sie können Ihre eigene Loader-Komponente verwenden
  ErrorView = YourError, // Sie können Ihre eigene Fehler-Komponente verwenden
  ...restProps
}) => {
  return <DataLoaderBase LoadingView={LoadingView} ErrorView={ErrorView} {...restProps} />;
};
```

### 4. Ihre erste Data Source definieren

```ts
import {skipContext} from '@gravity-ui/data-source';

// Ihre API-Funktion
import {fetchUser} from './api';

export const userDataSource = makePlainQueryDataSource({
  // Keys müssen eindeutig sein. Vielleicht sollten Sie einen Helfer für die Erstellung von Data-Source-Namen erstellen
  name: 'user',
  // skipContext ist ein Helfer, um die ersten zwei Parameter in der Funktion zu überspringen (context und fetchContext)
  fetch: skipContext(fetchUser),
  // Optional: Tags für erweiterte Cache-Invalidierung generieren
  tags: (params) => [`user:${params.userId}`, 'users'],
});
```

### 5. In Komponenten verwenden

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

## Kernkonzepte

### Data-Source-Typen

Die Bibliothek bietet zwei Haupttypen von Data Sources:

#### Plain Query Data Source

Für einfache Anfrage/Antwort-Muster:

```ts
const userDataSource = makePlainQueryDataSource({
  name: 'user',
  fetch: skipContext(async (params: {userId: number}) => {
    const response = await fetch(`/api/users/${params.userId}`);
    return response.json();
  }),
});
```

#### Infinite Query Data Source

Für Pagination und Infinite Scrolling:

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

### Status-Verwaltung

Die Bibliothek normalisiert Query-States in drei einfache Status:

- `loading` – Aktuelles Laden der Daten. Dasselbe wie `isLoading` in React Query
- `success` – Daten sind verfügbar (kann mit idle übersprungen werden)
- `error` – Fehler beim Abrufen der Daten

### Idle-Konzept

Die Bibliothek bietet ein spezielles `idle`-Symbol zum Überspringen der Query-Ausführung:

```ts
import {idle} from '@gravity-ui/data-source';

const UserProfile: React.FC<{userId?: number}> = ({userId}) => {
  // Die Query wird nicht ausgeführt, wenn userId nicht definiert ist
  const {data, status} = useQueryData(userDataSource, userId ? {userId} : idle);

  return (
    <DataLoader status={status} error={null}>
      {data && <UserCard user={data} />}
    </DataLoader>
  );
};
```

Wenn die Parameter `idle` entsprechen:

- Die Query wird nicht ausgeführt
- Der Status bleibt `success`
- Die Daten bleiben `undefined`
- Die Komponente kann sicher gerendert werden, ohne Ladeanzeige

**Vorteile von `idle`:**

1. **Typensicherheit** – TypeScript erkennt Typen für bedingte Parameter korrekt
2. **Leistung** – Vermeidet unnötige Server-Anfragen
3. **Einfachheit der Logik** – Kein Bedarf für zusätzliche `enabled`-States
4. **Konsistenz** – Einheitlicher Ansatz für alle bedingten Queries

Dies ist besonders nützlich für bedingte Queries, wenn Sie Daten nur unter bestimmten Bedingungen laden möchten, während die Typensicherheit erhalten bleibt.

## API-Referenz

### Data Sources erstellen

#### `makePlainQueryDataSource(config)`

Erstellt eine einfache Query-Data-Source für Anfrage/Antwort-Muster.

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

**Parameter:**

- `name` - Eindeutiger Bezeichner für die Datenquelle
- `fetch` - Funktion, die das eigentliche Datenabrufen durchführt
- `transformParams` (optional) - Transformiert Eingabeparameter vor der Anfrage
- `transformResponse` (optional) - Transformiert die Antwortdaten
- `tags` (optional) - Generiert Cache-Tags für die Invalidierung
- `options` (optional) - React-Query-Optionen

#### `makeInfiniteQueryDataSource(config)`

Erstellt eine unendliche Abfrage-Datenquelle für Paginierungs- und Infinite-Scrolling-Muster.

```ts
const infiniteDataSource = makeInfiniteQueryDataSource({
  name: 'infinite-data',
  fetch: skipContext(fetchFunction),
  next: (lastPage, allPages) => nextPageParam || undefined,
  prev: (firstPage, allPages) => prevPageParam || undefined,
  // ... other options same as plain
});
```

**Zusätzliche Parameter:**

- `next` - Funktion zur Bestimmung der Parameter für die nächste Seite
- `prev` (optional) - Funktion zur Bestimmung der Parameter für die vorherige Seite

### React-Hooks

#### `useQueryData(dataSource, params, options?)`

Haupthook zum Abrufen von Daten mit einer Datenquelle.

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

**Gibt zurück:**

- `data` - Die abgerufenen Daten
- `status` - Aktueller Status ('loading' | 'success' | 'error')
- `error` - Fehlerobjekt, falls die Anfrage fehlgeschlagen ist
- `refetch` - Funktion zum manuellen erneuten Abrufen der Daten
- Andere React-Query-Eigenschaften

#### `useQueryResponses(responses)`

Kombiniert mehrere Abfrage-Antworten zu einem einzigen Zustand.

```ts
const user = useQueryData(userDataSource, {userId});
const posts = useQueryData(postsDataSource, {userId});

const {status, error, refetch, refetchErrored} = useQueryResponses([user, posts]);
```

**Gibt zurück:**

- `status` - Kombinierter Status aller Abfragen
- `error` - Erster aufgetretener Fehler
- `refetch` - Funktion zum erneuten Abrufen aller Abfragen
- `refetchErrored` - Funktion zum erneuten Abrufen nur fehlgeschlagener Abfragen

#### `useRefetchAll(states)`

Erstellt eine Callback-Funktion zum erneuten Abrufen mehrerer Abfragen.

```ts
const refetchAll = useRefetchAll([user, posts, comments]);
// refetchAll() löst das erneute Abrufen für alle Abfragen aus
```

#### `useRefetchErrored(states)`

Erstellt eine Callback-Funktion zum erneuten Abrufen nur fehlgeschlagener Abfragen.

```ts
const refetchErrored = useRefetchErrored([user, posts, comments]);
// refetchErrored() ruft nur Abfragen mit Fehlern erneut ab
```

#### `useDataManager()`

Gibt den DataManager aus dem Kontext zurück.

```ts
const dataManager = useDataManager();
await dataManager.invalidateTag('users');
```

#### `useQueryContext()`

Gibt den Abfrage-Kontext zurück (zum Erstellen benutzerdefinierter Daten-Hooks basierend auf React Query).

### React-Komponenten

#### `<DataLoader />`

Komponente zum Handhaben von Ladezuständen und Fehlern.

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

**Props:**

- `status` - Aktueller Lade-Status
- `error` - Fehlerobjekt
- `errorAction` - Funktion oder Aktionskonfiguration für Fehler-Wiederholung
- `LoadingView` - Komponente, die während des Ladens angezeigt wird
- `ErrorView` - Komponente, die bei Fehlern angezeigt wird
- `loadingViewProps` - Props, die an LoadingView weitergeleitet werden
- `errorViewProps` - Props, die an ErrorView weitergeleitet werden

#### `<DataInfiniteLoader />`

Spezialisierte Komponente für unendliche Abfragen.

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

**Zusätzliche Props:**

- `hasNextPage` - Ob weitere Seiten verfügbar sind
- `fetchNextPage` - Funktion zum Abrufen der nächsten Seite
- `isFetchingNextPage` - Ob die nächste Seite gerade abgerufen wird
- `MoreView` - Komponente für den „Mehr laden“-Button

#### `withDataManager(Component)`

HOC, das DataManager als Prop injiziert.

```tsx
const MyComponent = withDataManager<Props>(({dataManager, ...props}) => {
  // Die Komponente hat Zugriff auf dataManager
  return <div>...</div>;
});
```

### Datenverwaltung

#### `ClientDataManager`

Hauptklasse für die Datenverwaltung.

```ts
const dataManager = new ClientDataManager({
  defaultOptions: {
    queries: {
      staleTime: 300000, // 5 Minuten
      retry: 3,
      refetchOnWindowFocus: false,
    },
  },
});
```

**Methoden:**

##### `invalidateTag(tag, options?)`

Ungültigmachen aller Abfragen mit einem bestimmten Tag.

```ts
await dataManager.invalidateTag('users');
await dataManager.invalidateTag('posts', {
  repeat: {count: 3, interval: 1000}, // Wiederholung der Invalidierung
});
```

##### `invalidateTags(tags, options?)`

Ungültigmachen von Abfragen, die alle angegebenen Tags haben.

```ts
await dataManager.invalidateTags(['user', 'profile']);
```

##### `invalidateSource(dataSource, options?)`

Ungültigmachen aller Abfragen für eine Datenquelle.

```ts
await dataManager.invalidateSource(userDataSource);
```

##### `invalidateParams(dataSource, params, options?)`

Ungültigmachen einer spezifischen Abfrage mit exakten Parametern.

```ts
await dataManager.invalidateParams(userDataSource, {userId: 123});
```

##### `resetSource(dataSource)`

Zurücksetzen (Löschen) aller zwischengespeicherten Daten für eine Datenquelle.

```ts
await dataManager.resetSource(userDataSource);
```

##### `resetParams(dataSource, params)`

Zurücksetzen der zwischengespeicherten Daten für spezifische Parameter.

```ts
await dataManager.resetParams(userDataSource, {userId: 123});
```

##### `invalidateSourceTags(dataSource, params, options?)`

Ungültigmachen von Abfragen basierend auf Tags, die von einer Datenquelle generiert wurden.

```ts
await dataManager.invalidateSourceTags(userDataSource, {userId: 123});
```

### Hilfsfunktionen

#### `skipContext(fetchFunction)`

Hilfsfunktion, um bestehende Fetch-Funktionen an die Datenquellen-Schnittstelle anzupassen.

```ts
// Bestehende Funktion
async function fetchUser(params: {userId: number}) {
  // ...
}

// Angepasst für Datenquelle
const dataSource = makePlainQueryDataSource({
  name: 'user',
  fetch: skipContext(fetchUser), // Überspringt Kontext- und fetchContext-Parameter
});
```

#### `withCatch(fetchFunction, errorHandler)`

Fügt standardisierte Fehlerbehandlung zu Fetch-Funktionen hinzu.

```ts
const safeFetch = withCatch(fetchUser, (error) => ({error: true, message: error.message}));
```

#### `withCancellation(fetchFunction)`

Fügt Unterstützung für Abbrüche zu Fetch-Funktionen hinzu.

```ts
const cancellableFetch = withCancellation(fetchFunction);
// Automatically handles AbortSignal from React Query
```

#### `getProgressiveRefetch(options)`

Erstellt eine Funktion für progressive Refetch-Intervalle.

```ts
const progressiveRefetch = getProgressiveRefetch({
  minInterval: 1000, // Start with 1 second
  maxInterval: 30000, // Max 30 seconds
  multiplier: 2, // Double each time
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

Konvertiert React-Query-Status in DataLoader-Status.

```ts
const status = normalizeStatus('pending', 'fetching'); // 'loading'
```

#### Status- und Fehler-Hilfsfunktionen

```ts
// Get combined status from multiple states
const status = getStatus([user, posts, comments]);

// Get first error from multiple states
const error = getError([user, posts, comments]);

// Merge multiple statuses
const combinedStatus = mergeStatuses(['loading', 'success', 'error']); // 'error'

// Check if query key has a tag
const hasUserTag = hasTag(queryKey, 'users');
```

#### Hilfsfunktionen für Schlüsselzusammensetzung

```ts
// Compose cache key for a data source
const key = composeKey(userDataSource, {userId: 123});

// Compose full key including tags
const fullKey = composeFullKey(userDataSource, {userId: 123});
```

#### Konstanten

```ts
import {idle} from '@gravity-ui/data-source';

// Special symbol for skipping query execution
const params = shouldFetch ? {userId: 123} : idle;

// Type-safe alternative to enabled: false
// Instead of:
const {data} = useQueryData(userDataSource, {userId: userId || ''}, {enabled: Boolean(userId)});

// Use:
const {data} = useQueryData(userDataSource, userId ? {userId} : idle);
// TypeScript correctly infers types for both branches
```

#### Zusammensetzung von Query-Optionen

```ts
// Compose React Query options for plain queries
const plainOptions = composePlainQueryOptions(context, dataSource, params, options);

// Compose React Query options for infinite queries
const infiniteOptions = composeInfiniteQueryOptions(context, dataSource, params, options);
```

**Hinweis:** Diese Funktionen sind hauptsächlich für den internen Gebrauch gedacht, wenn benutzerdefinierte Data-Source-Implementierungen erstellt werden.

## Erweiterte Muster

### Bedingte Queries mit Idle

Verwenden Sie `idle`, um bedingte Queries zu erstellen:

```ts
import {idle} from '@gravity-ui/data-source';

const ConditionalDataComponent: React.FC<{
  userId?: number;
  shouldLoadPosts: boolean;
}> = ({userId, shouldLoadPosts}) => {
  // Load user only if userId is defined
  const user = useQueryData(
    userDataSource,
    userId ? {userId} : idle
  );

  // Load posts only if user is loaded and flag is enabled
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

### Daten-Transformation

Transformieren Sie Anfrangeparameter und Antwortdaten:

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

### Cache-Ungültigmachung basierend auf Tags

Verwenden Sie Tags für eine anspruchsvolle Cache-Verwaltung:

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

// Invalidate all data for specific user
await dataManager.invalidateTag('user:123');

// Invalidate all user-related data
await dataManager.invalidateTag('users');
```

### Fehlerbehandlung mit Typen

Erstellen Sie typsichere Fehlerbehandlung:

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

### Infinite Queries mit komplexer Pagination

Behandeln Sie komplexe Paginierungsszenarien:

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

### Kombinieren mehrerer Data Sources

Kombinieren Sie Daten aus mehreren Quellen:

```ts
const UserProfile: React.FC<{userId: number}> = ({userId}) => {
  const user = useQueryData(userDataSource, {userId});
  const posts = useQueryData(userPostsDataSource, {userId});
  const followers = useQueryData(userFollowersDataSource, {userId});

  const combined = useQueryResponses([user, posts, followers]);

```

```jsx
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

## TypeScript-Unterstützung

Die Bibliothek wurde mit einem TypeScript-first-Ansatz entwickelt und bietet vollständige Typinferenz:

```ts
// Types are automatically inferred
const userDataSource = makePlainQueryDataSource({
  name: 'user',
  fetch: skipContext(async (params: {userId: number}): Promise<User> => {
    // Return type is inferred as User
  }),
});

// Hook return type is automatically typed
const {data} = useQueryData(userDataSource, {userId: 123});
// data is typed as User | undefined
```

### Benutzerdefinierte Fehler-Typen

Definieren und verwenden Sie benutzerdefinierte Fehler-Typen:

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
  {id: number}, // Params type
  {id: number}, // Request type
  ApiResponse, // Response type
  User, // Data type
  ApiError // Error type
>({
  name: 'typed-user',
  fetch: skipContext(fetchUser),
});
```

## Mitwirken

Lesen Sie bitte [CONTRIBUTING.md](CONTRIBUTING.md) für Details zu unserem Verhaltenskodex und dem Prozess zur Einreichung von Pull Requests.

## Lizenz

MIT-Lizenz. Siehe [LICENSE](LICENSE)-Datei für Details.