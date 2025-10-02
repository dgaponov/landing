# Data Source · [![npm version](https://img.shields.io/npm/v/@gravity-ui/data-source?logo=npm&label=version)](https://www.npmjs.com/package/@gravity-ui/data-source) [![ci](https://img.shields.io/github/actions/workflow/status/gravity-ui/data-source/ci.yml?branch=main&label=ci&logo=github)](https://github.com/gravity-ui/data-source/actions/workflows/ci.yml?query=branch:main)

**Data Source** — это простая обертка для работы с получением данных. Она выступает в роли своеобразного «порта» в [чистой архитектуре](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html). Благодаря ей вы можете создавать обертки для различных аспектов работы с данными в зависимости от ваших сценариев использования. **Data Source** под капотом использует [react-query](https://tanstack.com/query/latest).

## Установка

```bash
npm install @gravity-ui/data-source @tanstack/react-query
```

`@tanstack/react-query` является peer-зависимостью.

## Быстрый старт

### 1. Настройка DataManager

Сначала создайте и предоставьте `DataManager` в вашем приложении:

```tsx
import React from 'react';
import {ClientDataManager, DataManagerContext} from '@gravity-ui/data-source';

const dataManager = new ClientDataManager({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      retry: 3,
    },
    // ... other react-query options
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

### 2. Определение типов ошибок и оберток

Определите тип ошибки и создайте конструкторы для источников данных на основе стандартных конструкторов:

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

### 3. Создание кастомного компонента DataLoader

Напишите компонент `DataLoader` на основе стандартного, чтобы определить, как отображать состояния загрузки и ошибок:

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
  LoadingView = YourLoader, // You can use your own loader component
  ErrorView = YourError, // You can use your own error component
  ...restProps
}) => {
  return <DataLoaderBase LoadingView={LoadingView} ErrorView={ErrorView} {...restProps} />;
};
```

### 4. Определение первого источника данных

```ts
import {skipContext} from '@gravity-ui/data-source';

// Your API function
import {fetchUser} from './api';

export const userDataSource = makePlainQueryDataSource({
  // Keys have to be unique. Maybe you should create a helper for making names of data sources
  name: 'user',
  // skipContext is a helper to skip 2 first parameters in the function (context and fetchContext)
  fetch: skipContext(fetchUser),
  // Optional: generate tags for advanced cache invalidation
  tags: (params) => [`user:${params.userId}`, 'users'],
});
```

### 5. Использование в компонентах

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

## Основные концепции

### Типы источников данных

Библиотека предоставляет два основных типа источников данных:

#### Простой источник данных для запросов

Для простых паттернов запрос/ответ:

```ts
const userDataSource = makePlainQueryDataSource({
  name: 'user',
  fetch: skipContext(async (params: {userId: number}) => {
    const response = await fetch(`/api/users/${params.userId}`);
    return response.json();
  }),
});
```

#### Источник данных для бесконечных запросов

Для пагинации и бесконечной прокрутки:

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

### Управление статусами

Библиотека нормализует состояния запросов в три простых статуса:

- `loading` — фактическая загрузка данных. Аналогично `isLoading` в React Query
- `success` — данные доступны (может быть пропущено с помощью idle)
- `error` — не удалось получить данные

### Концепция Idle

Библиотека предоставляет специальный символ `idle` для пропуска выполнения запроса:

```ts
import {idle} from '@gravity-ui/data-source';

const UserProfile: React.FC<{userId?: number}> = ({userId}) => {
  // Query won't execute if userId is not defined
  const {data, status} = useQueryData(userDataSource, userId ? {userId} : idle);

  return (
    <DataLoader status={status} error={null}>
      {data && <UserCard user={data} />}
    </DataLoader>
  );
};
```

Когда параметры равны `idle`:

- Запрос не выполняется
- Статус остается `success`
- Данные остаются `undefined`
- Компонент может безопасно рендериться без загрузки

**Преимущества `idle`:**

1. **Типобезопасность** — TypeScript правильно выводит типы для условных параметров
2. **Производительность** — избегает ненужных запросов к серверу
3. **Простота логики** — не нужно управлять дополнительным состоянием `enabled`
4. **Согласованность** — единый подход для всех условных запросов

Это особенно полезно для условных запросов, когда вы хотите загружать данные только при определенных условиях, сохраняя при этом типобезопасность.

## Справочник API

### Создание источников данных

#### `makePlainQueryDataSource(config)`

Создает простой источник данных для запросов для простых паттернов запрос/ответ.

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

**Параметры:**

- `name` - Уникальный идентификатор источника данных
- `fetch` - Функция, выполняющая фактический запрос данных
- `transformParams` (необязательный) - Преобразование входных параметров перед запросом
- `transformResponse` (необязательный) - Преобразование данных ответа
- `tags` (необязательный) - Генерация тегов кэша для инвалидации
- `options` (необязательный) - Опции React Query

#### `makeInfiniteQueryDataSource(config)`

Создает источник данных для бесконечных запросов, подходящий для пагинации и сценариев бесконечной прокрутки.

```ts
const infiniteDataSource = makeInfiniteQueryDataSource({
  name: 'infinite-data',
  fetch: skipContext(fetchFunction),
  next: (lastPage, allPages) => nextPageParam || undefined,
  prev: (firstPage, allPages) => prevPageParam || undefined,
  // ... other options same as plain
});
```

**Дополнительные параметры:**

- `next` - Функция для определения параметров следующей страницы
- `prev` (необязательный) - Функция для определения параметров предыдущей страницы

### Хуки React

#### `useQueryData(dataSource, params, options?)`

Основной хук для получения данных с помощью источника данных.

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

**Возвращает:**

- `data` - Полученные данные
- `status` - Текущий статус ('loading' | 'success' | 'error')
- `error` - Объект ошибки, если запрос завершился неудачей
- `refetch` - Функция для ручного повторного запроса данных
- Другие свойства React Query

#### `useQueryResponses(responses)`

Объединяет несколько ответов запросов в единое состояние.

```ts
const user = useQueryData(userDataSource, {userId});
const posts = useQueryData(postsDataSource, {userId});

const {status, error, refetch, refetchErrored} = useQueryResponses([user, posts]);
```

**Возвращает:**

- `status` - Объединенный статус всех запросов
- `error` - Первая встреченная ошибка
- `refetch` - Функция для повторного запроса всех запросов
- `refetchErrored` - Функция для повторного запроса только неудачных запросов

#### `useRefetchAll(states)`

Создает колбэк для повторного запроса нескольких запросов.

```ts
const refetchAll = useRefetchAll([user, posts, comments]);
// refetchAll() запустит повторный запрос для всех запросов
```

#### `useRefetchErrored(states)`

Создает колбэк для повторного запроса только неудачных запросов.

```ts
const refetchErrored = useRefetchErrored([user, posts, comments]);
// refetchErrored() повторно запросит только запросы с ошибками
```

#### `useDataManager()`

Возвращает DataManager из контекста.

```ts
const dataManager = useDataManager();
await dataManager.invalidateTag('users');
```

#### `useQueryContext()`

Возвращает контекст запроса (для создания пользовательских хуков данных на основе react-query).

### Компоненты React

#### `<DataLoader />`

Компонент для обработки состояний загрузки и ошибок.

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

**Пропсы:**

- `status` - Текущий статус загрузки
- `error` - Объект ошибки
- `errorAction` - Функция или конфигурация действия для повторной попытки при ошибке
- `LoadingView` - Компонент, отображаемый во время загрузки
- `ErrorView` - Компонент, отображаемый при ошибке
- `loadingViewProps` - Пропсы, передаваемые в LoadingView
- `errorViewProps` - Пропсы, передаваемые в ErrorView

#### `<DataInfiniteLoader />`

Специализированный компонент для бесконечных запросов.

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

**Дополнительные пропсы:**

- `hasNextPage` - Доступны ли еще страницы
- `fetchNextPage` - Функция для получения следующей страницы
- `isFetchingNextPage` - Загружается ли следующая страница
- `MoreView` - Компонент для кнопки "загрузить больше"

#### `withDataManager(Component)`

HOC, который注入 DataManager как пропс.

```tsx
const MyComponent = withDataManager<Props>(({dataManager, ...props}) => {
  // Компонент имеет доступ к dataManager
  return <div>...</div>;
});
```

### Управление данными

#### `ClientDataManager`

Основной класс для управления данными.

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

**Методы:**

##### `invalidateTag(tag, options?)`

Инвалидирует все запросы с указанным тегом.

```ts
await dataManager.invalidateTag('users');
await dataManager.invalidateTag('posts', {
  repeat: {count: 3, interval: 1000}, // Повторная попытка инвалидации
});
```

##### `invalidateTags(tags, options?)`

Инвалидирует запросы, имеющие все указанные теги.

```ts
await dataManager.invalidateTags(['user', 'profile']);
```

##### `invalidateSource(dataSource, options?)`

Инвалидирует все запросы для источника данных.

```ts
await dataManager.invalidateSource(userDataSource);
```

##### `invalidateParams(dataSource, params, options?)`

Инвалидирует конкретный запрос с точными параметрами.

```ts
await dataManager.invalidateParams(userDataSource, {userId: 123});
```

##### `resetSource(dataSource)`

Сбрасывает (очищает) все кэшированные данные для источника данных.

```ts
await dataManager.resetSource(userDataSource);
```

##### `resetParams(dataSource, params)`

Сбрасывает кэшированные данные для конкретных параметров.

```ts
await dataManager.resetParams(userDataSource, {userId: 123});
```

##### `invalidateSourceTags(dataSource, params, options?)`

Инвалидирует запросы на основе тегов, сгенерированных источником данных.

```ts
await dataManager.invalidateSourceTags(userDataSource, {userId: 123});
```

### Утилиты

#### `skipContext(fetchFunction)`

Утилита для адаптации существующих функций fetch к интерфейсу источника данных.

```ts
// Существующая функция
async function fetchUser(params: {userId: number}) {
  // ...
}

// Адаптированная для источника данных
const dataSource = makePlainQueryDataSource({
  name: 'user',
  fetch: skipContext(fetchUser), // Пропускает контекст и параметры fetchContext
});
```

#### `withCatch(fetchFunction, errorHandler)`

Добавляет стандартизированную обработку ошибок к функциям fetch.

```ts
const safeFetch = withCatch(fetchUser, (error) => ({error: true, message: error.message}));
```

#### `withCancellation(fetchFunction)`

Добавляет поддержку отмены к функциям fetch.

```ts
const cancellableFetch = withCancellation(fetchFunction);
// Автоматически обрабатывает AbortSignal из React Query
```

#### `getProgressiveRefetch(options)`

Создает функцию для прогрессивного повторного запроса с интервалом.

```ts
const progressiveRefetch = getProgressiveRefetch({
  minInterval: 1000, // Начинаем с 1 секунды
  maxInterval: 30000, // Максимум 30 секунд
  multiplier: 2, // Удваиваем каждый раз
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

Преобразует статусы React Query в статусы DataLoader.

```ts
const status = normalizeStatus('pending', 'fetching'); // 'loading'
```

#### Утилиты для статусов и ошибок

```ts
// Получить объединенный статус из нескольких состояний
const status = getStatus([user, posts, comments]);

// Получить первую ошибку из нескольких состояний
const error = getError([user, posts, comments]);

// Объединить несколько статусов
const combinedStatus = mergeStatuses(['loading', 'success', 'error']); // 'error'

// Проверить, есть ли тег в ключе запроса
const hasUserTag = hasTag(queryKey, 'users');
```

#### Утилиты для композиции ключей

```ts
// Составить ключ кэша для источника данных
const key = composeKey(userDataSource, {userId: 123});

// Составить полный ключ, включая теги
const fullKey = composeFullKey(userDataSource, {userId: 123});
```

#### Константы

```ts
import {idle} from '@gravity-ui/data-source';

// Специальный символ для пропуска выполнения запроса
const params = shouldFetch ? {userId: 123} : idle;

// Типобезопасная альтернатива enabled: false
// Вместо:
const {data} = useQueryData(userDataSource, {userId: userId || ''}, {enabled: Boolean(userId)});

// Используйте:
const {data} = useQueryData(userDataSource, userId ? {userId} : idle);
// TypeScript правильно выводит типы для обоих веток
```

#### Композиция опций запроса

```ts
// Составить опции React Query для обычных запросов
const plainOptions = composePlainQueryOptions(context, dataSource, params, options);

// Составить опции React Query для бесконечных запросов
const infiniteOptions = composeInfiniteQueryOptions(context, dataSource, params, options);
```

**Примечание:** Эти функции предназначены в основном для внутреннего использования при создании пользовательских реализаций источников данных.

## Продвинутые шаблоны

### Условные запросы с Idle

Используйте `idle` для создания условных запросов:

```ts
import {idle} from '@gravity-ui/data-source';

const ConditionalDataComponent: React.FC<{
  userId?: number;
  shouldLoadPosts: boolean;
}> = ({userId, shouldLoadPosts}) => {
  // Загружаем пользователя только если userId определен
  const user = useQueryData(
    userDataSource,
    userId ? {userId} : idle
  );

  // Загружаем посты только если пользователь загружен и флаг включен
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

### Преобразование данных

Преобразуйте параметры запроса и данные ответа:

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

### Инвалидация кэша на основе тегов

Используйте теги для сложного управления кэшем:

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

// Инвалидировать все данные для конкретного пользователя
await dataManager.invalidateTag('user:123');

// Инвалидировать все данные, связанные с пользователями
await dataManager.invalidateTag('users');
```

### Обработка ошибок с типами

Создайте типобезопасную обработку ошибок:

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

### Бесконечные запросы со сложной пагинацией

Обработайте сценарии со сложной пагинацией:

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

### Комбинирование нескольких источников данных

Объедините данные из нескольких источников:

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
```

## Поддержка TypeScript

Библиотека создана с приоритетом на TypeScript и обеспечивает полную типизацию:

```ts
// Типы автоматически выводятся
const userDataSource = makePlainQueryDataSource({
  name: 'user',
  fetch: skipContext(async (params: {userId: number}): Promise<User> => {
    // Возвращаемый тип выводится как User
  }),
});

// Тип возвращаемого значения хука автоматически типизирован
const {data} = useQueryData(userDataSource, {userId: 123});
// data типизирован как User | undefined
```

### Пользовательские типы ошибок

Определите и используйте пользовательские типы ошибок:

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
  {id: number}, // Тип параметров
  {id: number}, // Тип запроса
  ApiResponse, // Тип ответа
  User, // Тип данных
  ApiError // Тип ошибки
>({
  name: 'typed-user',
  fetch: skipContext(fetchUser),
});
```

## Вклад в разработку

Пожалуйста, ознакомьтесь с [CONTRIBUTING.md](CONTRIBUTING.md) для получения подробностей о нашем кодексе поведения и процессе отправки pull-запросов.

## Лицензия

Лицензия MIT. Подробности в файле [LICENSE](LICENSE).