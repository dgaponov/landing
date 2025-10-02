# Data Source · [![npm version](https://img.shields.io/npm/v/@gravity-ui/data-source?logo=npm&label=version)](https://www.npmjs.com/package/@gravity-ui/data-source) [![ci](https://img.shields.io/github/actions/workflow/status/gravity-ui/data-source/ci.yml?branch=main&label=ci&logo=github)](https://github.com/gravity-ui/data-source/actions/workflows/ci.yml?query=branch:main)

**Data Source**는 데이터 가져오기를 위한 간단한 래퍼입니다. 이는 [clean architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)의 일종의 "포트" 역할을 합니다. 사용 사례에 따라 데이터 가져오기 주변의 래퍼를 만들 수 있도록 해줍니다. **Data Source**는 내부적으로 [react-query](https://tanstack.com/query/latest)를 사용합니다.

## 설치

```bash
npm install @gravity-ui/data-source @tanstack/react-query
```

`@tanstack/react-query`는 피어 의존성입니다.

## 빠른 시작

### 1. DataManager 설정

먼저 애플리케이션에서 `DataManager`를 생성하고 제공하세요:

```tsx
import React from 'react';
import {ClientDataManager, DataManagerContext} from '@gravity-ui/data-source';

const dataManager = new ClientDataManager({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5분
      retry: 3,
    },
    // ... 다른 react-query 옵션
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

### 2. 오류 유형 및 래퍼 정의

오류 유형을 정의하고 기본 생성자를 기반으로 데이터 소스 생성자를 만드세요:

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

### 3. 사용자 지정 DataLoader 컴포넌트 생성

기본값을 기반으로 `DataLoader` 컴포넌트를 작성하여 로딩 상태와 오류 표시를 정의하세요:

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
  LoadingView = YourLoader, // 자체 로더 컴포넌트를 사용할 수 있습니다
  ErrorView = YourError, // 자체 오류 컴포넌트를 사용할 수 있습니다
  ...restProps
}) => {
  return <DataLoaderBase LoadingView={LoadingView} ErrorView={ErrorView} {...restProps} />;
};
```

### 4. 첫 번째 데이터 소스 정의

```ts
import {skipContext} from '@gravity-ui/data-source';

// API 함수
import {fetchUser} from './api';

export const userDataSource = makePlainQueryDataSource({
  // 키는 고유해야 합니다. 데이터 소스 이름을 만드는 도우미를 생성하는 것이 좋을 수 있습니다
  name: 'user',
  // skipContext는 함수의 처음 두 매개변수(컨텍스트와 fetchContext)를 건너뛰는 도우미입니다
  fetch: skipContext(fetchUser),
  // 선택 사항: 고급 캐시 무효화를 위한 태그 생성
  tags: (params) => [`user:${params.userId}`, 'users'],
});
```

### 5. 컴포넌트에서 사용

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

## 핵심 개념

### 데이터 소스 유형

이 라이브러리는 두 가지 주요 데이터 소스 유형을 제공합니다:

#### Plain Query Data Source

간단한 요청/응답 패턴에 사용:

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

페이지네이션 및 무한 스크롤에 사용:

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

### 상태 관리

라이브러리는 쿼리 상태를 세 가지 간단한 상태로 정규화합니다:

- `loading` - 실제 데이터 로딩. React Query의 `isLoading`과 동일
- `success` - 데이터 사용 가능 (idle을 사용해 건너뛸 수 있음)
- `error` - 데이터 가져오기 실패

### Idle 개념

라이브러리는 쿼리 실행을 건너뛰기 위한 특수 `idle` 심볼을 제공합니다:

```ts
import {idle} from '@gravity-ui/data-source';

const UserProfile: React.FC<{userId?: number}> = ({userId}) => {
  // userId가 정의되지 않으면 쿼리가 실행되지 않습니다
  const {data, status} = useQueryData(userDataSource, userId ? {userId} : idle);

  return (
    <DataLoader status={status} error={null}>
      {data && <UserCard user={data} />}
    </DataLoader>
  );
};
```

매개변수가 `idle`과 같을 때:

- 쿼리가 실행되지 않음
- 상태가 `success`로 유지됨
- 데이터가 `undefined`로 유지됨
- 컴포넌트가 로딩 없이 안전하게 렌더링될 수 있음

**`idle`의 이점:**

1. **타입 안전성** - TypeScript가 조건부 매개변수에 대한 타입을 올바르게 추론함
2. **성능** - 불필요한 서버 요청 방지
3. **로직 단순화** - 추가 `enabled` 상태 관리 불필요
4. **일관성** - 모든 조건부 쿼리에 대한 통합 접근 방식

이것은 특정 조건에서만 데이터를 로드하고 싶을 때 타입 안전성을 유지하면서 특히 유용합니다.

## API 참조

### 데이터 소스 생성

#### `makePlainQueryDataSource(config)`

간단한 요청/응답 패턴을 위한 일반 쿼리 데이터 소스를 생성합니다.

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

**매개변수:**

- `name` - 데이터 소스의 고유 식별자
- `fetch` - 실제 데이터 가져오기를 수행하는 함수
- `transformParams` (선택) - 요청 전에 입력 매개변수를 변환
- `transformResponse` (선택) - 응답 데이터를 변환
- `tags` (선택) - 무효화のための 캐시 태그 생성
- `options` (선택) - React Query 옵션

#### `makeInfiniteQueryDataSource(config)`

페이징 및 무한 스크롤 패턴을 위한 무한 쿼리 데이터 소스를 생성합니다.

```ts
const infiniteDataSource = makeInfiniteQueryDataSource({
  name: 'infinite-data',
  fetch: skipContext(fetchFunction),
  next: (lastPage, allPages) => nextPageParam || undefined,
  prev: (firstPage, allPages) => prevPageParam || undefined,
  // ... other options same as plain
});
```

**추가 매개변수:**

- `next` - 다음 페이지 매개변수를 결정하는 함수
- `prev` (선택) - 이전 페이지 매개변수를 결정하는 함수

### React Hooks

#### `useQueryData(dataSource, params, options?)`

데이터 소스를 사용한 데이터 가져오기를 위한 주요 훅입니다.

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

**반환값:**

- `data` - 가져온 데이터
- `status` - 현재 상태 ('loading' | 'success' | 'error')
- `error` - 요청이 실패한 경우 오류 객체
- `refetch` - 데이터를 수동으로 다시 가져오는 함수
- 기타 React Query 속성

#### `useQueryResponses(responses)`

여러 쿼리 응답을 하나의 상태로 결합합니다.

```ts
const user = useQueryData(userDataSource, {userId});
const posts = useQueryData(postsDataSource, {userId});

const {status, error, refetch, refetchErrored} = useQueryResponses([user, posts]);
```

**반환값:**

- `status` - 모든 쿼리의 결합된 상태
- `error` - 발생한 첫 번째 오류
- `refetch` - 모든 쿼리를 다시 가져오는 함수
- `refetchErrored` - 실패한 쿼리만 다시 가져오는 함수

#### `useRefetchAll(states)`

여러 쿼리를 다시 가져오는 콜백을 생성합니다.

```ts
const refetchAll = useRefetchAll([user, posts, comments]);
// refetchAll() will trigger refetch for all queries
```

#### `useRefetchErrored(states)`

실패한 쿼리만 다시 가져오는 콜백을 생성합니다.

```ts
const refetchErrored = useRefetchErrored([user, posts, comments]);
// refetchErrored() will only refetch queries with errors
```

#### `useDataManager()`

컨텍스트에서 DataManager를 반환합니다.

```ts
const dataManager = useDataManager();
await dataManager.invalidateTag('users');
```

#### `useQueryContext()`

쿼리 컨텍스트를 반환합니다 (react-query를 기반으로 한 커스텀 데이터 훅을 빌드하기 위해).

### React Components

#### `<DataLoader />`

로딩 상태와 오류를 처리하는 컴포넌트입니다.

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

- `status` - 현재 로딩 상태
- `error` - 오류 객체
- `errorAction` - 오류 재시도のための 함수 또는 액션 설정
- `LoadingView` - 로딩 중에 표시할 컴포넌트
- `ErrorView` - 오류 발생 시 표시할 컴포넌트
- `loadingViewProps` - LoadingView에 전달할 속성
- `errorViewProps` - ErrorView에 전달할 속성

#### `<DataInfiniteLoader />`

무한 쿼리를 위한 특화된 컴포넌트입니다.

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

**추가 Props:**

- `hasNextPage` - 추가 페이지가 사용 가능한지 여부
- `fetchNextPage` - 다음 페이지를 가져오는 함수
- `isFetchingNextPage` - 다음 페이지가 가져오는 중인지 여부
- `MoreView` - "더 가져오기" 버튼のための 컴포넌트

#### `withDataManager(Component)`

DataManager를 prop으로 주입하는 HOC입니다.

```tsx
const MyComponent = withDataManager<Props>(({dataManager, ...props}) => {
  // Component has access to dataManager
  return <div>...</div>;
});
```

### Data Management

#### `ClientDataManager`

데이터 관리를 위한 주요 클래스입니다.

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

**메서드:**

##### `invalidateTag(tag, options?)`

특정 태그가 있는 모든 쿼리를 무효화합니다.

```ts
await dataManager.invalidateTag('users');
await dataManager.invalidateTag('posts', {
  repeat: {count: 3, interval: 1000}, // Retry invalidation
});
```

##### `invalidateTags(tags, options?)`

지정된 모든 태그가 있는 쿼리를 무효화합니다.

```ts
await dataManager.invalidateTags(['user', 'profile']);
```

##### `invalidateSource(dataSource, options?)`

데이터 소스의 모든 쿼리를 무효화합니다.

```ts
await dataManager.invalidateSource(userDataSource);
```

##### `invalidateParams(dataSource, params, options?)`

정확한 매개변수로 특정 쿼리를 무효화합니다.

```ts
await dataManager.invalidateParams(userDataSource, {userId: 123});
```

##### `resetSource(dataSource)`

데이터 소스의 모든 캐시된 데이터를 재설정(지우기)합니다.

```ts
await dataManager.resetSource(userDataSource);
```

##### `resetParams(dataSource, params)`

특정 매개변수에 대한 캐시된 데이터를 재설정합니다.

```ts
await dataManager.resetParams(userDataSource, {userId: 123});
```

##### `invalidateSourceTags(dataSource, params, options?)`

데이터 소스가 생성한 태그를 기반으로 쿼리를 무효화합니다.

```ts
await dataManager.invalidateSourceTags(userDataSource, {userId: 123});
```

### Utilities

#### `skipContext(fetchFunction)`

기존 가져오기 함수를 데이터 소스 인터페이스에 맞게 적응시키는 유틸리티입니다.

```ts
// Existing function
async function fetchUser(params: {userId: number}) {
  // ...
}

// Adapted for data source
const dataSource = makePlainQueryDataSource({
  name: 'user',
  fetch: skipContext(fetchUser), // Skips context and fetchContext params
});
```

#### `withCatch(fetchFunction, errorHandler)`

Adds fetch 함수에 표준화된 오류 처리를 추가합니다.

```ts
const safeFetch = withCatch(fetchUser, (error) => ({error: true, message: error.message}));
```

#### `withCancellation(fetchFunction)`

fetch 함수에 취소 지원을 추가합니다.

```ts
const cancellableFetch = withCancellation(fetchFunction);
// Automatically handles AbortSignal from React Query
```

#### `getProgressiveRefetch(options)`

점진적인 재요청 간격 함수를 생성합니다.

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

React Query 상태를 DataLoader 상태로 변환합니다.

```ts
const status = normalizeStatus('pending', 'fetching'); // 'loading'
```

#### 상태 및 오류 유틸리티

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

#### 키 구성 유틸리티

```ts
// Compose cache key for a data source
const key = composeKey(userDataSource, {userId: 123});

// Compose full key including tags
const fullKey = composeFullKey(userDataSource, {userId: 123});
```

#### 상수

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

#### 쿼리 옵션 구성

```ts
// Compose React Query options for plain queries
const plainOptions = composePlainQueryOptions(context, dataSource, params, options);

// Compose React Query options for infinite queries
const infiniteOptions = composeInfiniteQueryOptions(context, dataSource, params, options);
```

**참고:** 이러한 함수들은 주로 사용자 지정 데이터 소스 구현 시 내부적으로 사용하기 위한 것입니다.

## 고급 패턴

### Idle을 사용한 조건부 쿼리

`idle`을 사용하여 조건부 쿼리를 생성합니다:

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

### 데이터 변환

요청 매개변수와 응답 데이터를 변환합니다:

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

### 태그 기반 캐시 무효화

태그를 사용하여 정교한 캐시 관리를 수행합니다:

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

### 타입을 사용한 오류 처리

타입 안전한 오류 처리를 생성합니다:

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

### 복잡한 페이지네이션을 사용한 무한 쿼리

복잡한 페이지네이션 시나리오를 처리합니다:

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

### 여러 데이터 소스 결합

여러 소스의 데이터를 결합합니다:

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

## TypeScript 지원

이 라이브러리는 TypeScript 우선 접근 방식을 기반으로 구축되었으며, 완전한 타입 추론을 제공합니다:

```ts
// 타입이 자동으로 추론됩니다
const userDataSource = makePlainQueryDataSource({
  name: 'user',
  fetch: skipContext(async (params: {userId: number}): Promise<User> => {
    // 반환 타입이 User로 추론됩니다
  }),
});

// 훅 반환 타입이 자동으로 타입화됩니다
const {data} = useQueryData(userDataSource, {userId: 123});
// data는 User | undefined로 타입화됩니다
```

### 사용자 지정 오류 타입

사용자 지정 오류 타입을 정의하고 사용하세요:

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

## 기여하기

코드 오브 컨덕트와 풀 리퀘스트 제출 과정에 대한 자세한 내용은 [CONTRIBUTING.md](CONTRIBUTING.md)를 읽어주세요.

## 라이선스

MIT 라이선스. 자세한 내용은 [LICENSE](LICENSE) 파일을 참조하세요.