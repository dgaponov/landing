# Data Source · [![npm version](https://img.shields.io/npm/v/@gravity-ui/data-source?logo=npm&label=version)](https://www.npmjs.com/package/@gravity-ui/data-source) [![ci](https://img.shields.io/github/actions/workflow/status/gravity-ui/data-source/ci.yml?branch=main&label=ci&logo=github)](https://github.com/gravity-ui/data-source/actions/workflows/ci.yml?query=branch:main)

**Data Source** 是一个围绕数据获取的简单封装。它类似于[clean architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)中的一种“端口”。它允许你根据具体用例，为数据获取相关的逻辑创建封装。**Data Source** 在底层使用了[react-query](https://tanstack.com/query/latest)。

## 安装

```bash
npm install @gravity-ui/data-source @tanstack/react-query
```

`@tanstack/react-query` 是一个 peer dependency。

## 快速开始

### 1. 设置 DataManager

首先，在你的应用中创建并提供一个 `DataManager`：

```tsx
import React from 'react';
import {ClientDataManager, DataManagerContext} from '@gravity-ui/data-source';

const dataManager = new ClientDataManager({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 分钟
      retry: 3,
    },
    // ... 其他 react-query 选项
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

### 2. 定义错误类型和封装

定义错误类型，并基于默认构造函数创建你自己的数据源构造函数：

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

### 3. 创建自定义 DataLoader 组件

基于默认的 `DataLoader` 编写一个组件，来定义加载状态和错误的显示方式：

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
  LoadingView = YourLoader, // 你可以使用自己的加载组件
  ErrorView = YourError, // 你可以使用自己的错误组件
  ...restProps
}) => {
  return <DataLoaderBase LoadingView={LoadingView} ErrorView={ErrorView} {...restProps} />;
};
```

### 4. 定义你的第一个数据源

```ts
import {skipContext} from '@gravity-ui/data-source';

// 你的 API 函数
import {fetchUser} from './api';

export const userDataSource = makePlainQueryDataSource({
  // 键必须唯一。也许你应该创建一个辅助函数来生成数据源的名称
  name: 'user',
  // skipContext 是一个辅助函数，用于跳过函数的前两个参数（context 和 fetchContext）
  fetch: skipContext(fetchUser),
  // 可选：为高级缓存失效生成标签
  tags: (params) => [`user:${params.userId}`, 'users'],
});
```

### 5. 在组件中使用

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

## 核心概念

### 数据源类型

该库提供了两种主要的数据源类型：

#### 简单查询数据源

适用于简单的请求/响应模式：

```ts
const userDataSource = makePlainQueryDataSource({
  name: 'user',
  fetch: skipContext(async (params: {userId: number}) => {
    const response = await fetch(`/api/users/${params.userId}`);
    return response.json();
  }),
});
```

#### 无限查询数据源

适用于分页和无限滚动：

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

### 状态管理

该库将查询状态标准化为三种简单的状态：

- `loading` - 实际数据加载中。与 React Query 中的 `isLoading` 相同
- `success` - 数据可用（可以使用 idle 跳过）
- `error` - 数据获取失败

### Idle 概念

该库提供了一个特殊的 `idle` 符号，用于跳过查询执行：

```ts
import {idle} from '@gravity-ui/data-source';

const UserProfile: React.FC<{userId?: number}> = ({userId}) => {
  // 如果 userId 未定义，查询不会执行
  const {data, status} = useQueryData(userDataSource, userId ? {userId} : idle);

  return (
    <DataLoader status={status} error={null}>
      {data && <UserCard user={data} />}
    </DataLoader>
  );
};
```

当参数等于 `idle` 时：

- 查询不会执行
- 状态保持为 `success`
- 数据保持为 `undefined`
- 组件可以安全渲染，而无需加载

**使用 `idle` 的优势：**

1. **类型安全** - TypeScript 会正确推断条件参数的类型
2. **性能** - 避免不必要的服务器请求
3. **逻辑简洁** - 无需管理额外的 `enabled` 状态
4. **一致性** - 为所有条件查询提供统一的方法

这在条件查询中特别有用，当你只想在特定条件下加载数据时，同时保持类型安全。

## API 参考

### 创建数据源

#### `makePlainQueryDataSource(config)`

创建一个简单查询数据源，用于简单的请求/响应模式。

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

**参数：**

- `name` - 数据源的唯一标识符
- `fetch` - 执行实际数据获取的函数
- `transformParams` (可选) - 在请求前转换输入参数
- `transformResponse` (可选) - 转换响应数据
- `tags` (可选) - 生成用于失效的缓存标签
- `options` (可选) - React Query 选项

#### `makeInfiniteQueryDataSource(config)`

用于分页和无限滚动模式的无限查询数据源创建函数。

```ts
const infiniteDataSource = makeInfiniteQueryDataSource({
  name: 'infinite-data',
  fetch: skipContext(fetchFunction),
  next: (lastPage, allPages) => nextPageParam || undefined,
  prev: (firstPage, allPages) => prevPageParam || undefined,
  // ... other options same as plain
});
```

**额外参数：**

- `next` - 确定下一页参数的函数
- `prev` (可选) - 确定上一页参数的函数

### React Hooks

#### `useQueryData(dataSource, params, options?)`

使用数据源获取数据的主要 Hook。

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

**返回值：**

- `data` - 获取到的数据
- `status` - 当前状态 ('loading' | 'success' | 'error')
- `error` - 如果请求失败的错误对象
- `refetch` - 手动重新获取数据的函数
- 其他 React Query 属性

#### `useQueryResponses(responses)`

将多个查询响应组合成单一状态。

```ts
const user = useQueryData(userDataSource, {userId});
const posts = useQueryData(postsDataSource, {userId});

const {status, error, refetch, refetchErrored} = useQueryResponses([user, posts]);
```

**返回值：**

- `status` - 所有查询的组合状态
- `error` - 遇到的第一个错误
- `refetch` - 重新获取所有查询的函数
- `refetchErrored` - 仅重新获取失败查询的函数

#### `useRefetchAll(states)`

创建用于重新获取多个查询的回调。

```ts
const refetchAll = useRefetchAll([user, posts, comments]);
// refetchAll() 将触发所有查询的重新获取
```

#### `useRefetchErrored(states)`

创建仅重新获取失败查询的回调。

```ts
const refetchErrored = useRefetchErrored([user, posts, comments]);
// refetchErrored() 仅重新获取有错误的查询
```

#### `useDataManager()`

从上下文中返回 DataManager。

```ts
const dataManager = useDataManager();
await dataManager.invalidateTag('users');
```

#### `useQueryContext()`

返回查询上下文（用于基于 react-query 构建自定义数据 Hook）。

### React 组件

#### `<DataLoader />`

用于处理加载状态和错误的组件。

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

**Props：**

- `status` - 当前加载状态
- `error` - 错误对象
- `errorAction` - 用于错误重试的函数或动作配置
- `LoadingView` - 加载期间显示的组件
- `ErrorView` - 错误时显示的组件
- `loadingViewProps` - 传递给 LoadingView 的 Props
- `errorViewProps` - 传递给 ErrorView 的 Props

#### `<DataInfiniteLoader />`

专用于无限查询的组件。

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

**额外 Props：**

- `hasNextPage` - 是否有更多页面可用
- `fetchNextPage` - 获取下一页的函数
- `isFetchingNextPage` - 是否正在获取下一页
- `MoreView` - 用于“加载更多”按钮的组件

#### `withDataManager(Component)`

将 DataManager 作为 Prop 注入的 HOC。

```tsx
const MyComponent = withDataManager<Props>(({dataManager, ...props}) => {
  // 组件可以访问 dataManager
  return <div>...</div>;
});
```

### 数据管理

#### `ClientDataManager`

数据管理的主要类。

```ts
const dataManager = new ClientDataManager({
  defaultOptions: {
    queries: {
      staleTime: 300000, // 5 分钟
      retry: 3,
      refetchOnWindowFocus: false,
    },
  },
});
```

**方法：**

##### `invalidateTag(tag, options?)`

使指定标签的所有查询失效。

```ts
await dataManager.invalidateTag('users');
await dataManager.invalidateTag('posts', {
  repeat: {count: 3, interval: 1000}, // 重试失效操作
});
```

##### `invalidateTags(tags, options?)`

使具有所有指定标签的查询失效。

```ts
await dataManager.invalidateTags(['user', 'profile']);
```

##### `invalidateSource(dataSource, options?)`

使数据源的所有查询失效。

```ts
await dataManager.invalidateSource(userDataSource);
```

##### `invalidateParams(dataSource, params, options?)`

使用确切参数使特定查询失效。

```ts
await dataManager.invalidateParams(userDataSource, {userId: 123});
```

##### `resetSource(dataSource)`

重置（清除）数据源的所有缓存数据。

```ts
await dataManager.resetSource(userDataSource);
```

##### `resetParams(dataSource, params)`

重置特定参数的缓存数据。

```ts
await dataManager.resetParams(userDataSource, {userId: 123});
```

##### `invalidateSourceTags(dataSource, params, options?)`

基于数据源生成的标签使查询失效。

```ts
await dataManager.invalidateSourceTags(userDataSource, {userId: 123});
```

### 实用工具

#### `skipContext(fetchFunction)`

将现有获取函数适配到数据源接口的实用工具。

```ts
// 现有函数
async function fetchUser(params: {userId: number}) {
  // ...
}

// 适配到数据源
const dataSource = makePlainQueryDataSource({
  name: 'user',
  fetch: skipContext(fetchUser), // 跳过 context 和 fetchContext 参数
});
```

#### `withCatch(fetchFunction, errorHandler)`

```

为 fetch 函数添加标准化的错误处理。

```ts
const safeFetch = withCatch(fetchUser, (error) => ({error: true, message: error.message}));
```

#### `withCancellation(fetchFunction)`

为 fetch 函数添加取消支持。

```ts
const cancellableFetch = withCancellation(fetchFunction);
// 自动处理来自 React Query 的 AbortSignal
```

#### `getProgressiveRefetch(options)`

创建一个渐进式重试间隔函数。

```ts
const progressiveRefetch = getProgressiveRefetch({
  minInterval: 1000, // 从 1 秒开始
  maxInterval: 30000, // 最大 30 秒
  multiplier: 2, // 每次加倍
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

将 React Query 状态转换为 DataLoader 状态。

```ts
const status = normalizeStatus('pending', 'fetching'); // 'loading'
```

#### 状态和错误工具函数

```ts
// 从多个状态中获取组合状态
const status = getStatus([user, posts, comments]);

// 从多个状态中获取第一个错误
const error = getError([user, posts, comments]);

// 合并多个状态
const combinedStatus = mergeStatuses(['loading', 'success', 'error']); // 'error'

// 检查查询键是否具有标签
const hasUserTag = hasTag(queryKey, 'users');
```

#### 键组合工具函数

```ts
// 为数据源组合缓存键
const key = composeKey(userDataSource, {userId: 123});

// 组合包含标签的完整键
const fullKey = composeFullKey(userDataSource, {userId: 123});
```

#### 常量

```ts
import {idle} from '@gravity-ui/data-source';

// 用于跳过查询执行的特殊符号
const params = shouldFetch ? {userId: 123} : idle;

// 比 enabled: false 更类型安全的替代方案
// 而不是：
const {data} = useQueryData(userDataSource, {userId: userId || ''}, {enabled: Boolean(userId)});

// 使用：
const {data} = useQueryData(userDataSource, userId ? {userId} : idle);
// TypeScript 会正确推断两个分支的类型
```

#### 查询选项组合

```ts
// 为普通查询组合 React Query 选项
const plainOptions = composePlainQueryOptions(context, dataSource, params, options);

// 为无限查询组合 React Query 选项
const infiniteOptions = composeInfiniteQueryOptions(context, dataSource, params, options);
```

**注意：** 这些函数主要用于创建自定义数据源实现时的内部使用。

## 高级模式

### 使用 Idle 实现条件查询

使用 `idle` 创建条件查询：

```ts
import {idle} from '@gravity-ui/data-source';

const ConditionalDataComponent: React.FC<{
  userId?: number;
  shouldLoadPosts: boolean;
}> = ({userId, shouldLoadPosts}) => {
  // 仅当 userId 定义时加载用户
  const user = useQueryData(
    userDataSource,
    userId ? {userId} : idle
  );

  // 仅当用户已加载且标志启用时加载帖子
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

### 数据转换

转换请求参数和响应数据：

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

### 基于标签的缓存失效

使用标签进行复杂的缓存管理：

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

// 使特定用户的所有数据失效
await dataManager.invalidateTag('user:123');

// 使所有用户相关数据失效
await dataManager.invalidateTag('users');
```

### 带类型的错误处理

创建类型安全的错误处理：

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

### 复杂分页的无限查询

处理复杂分页场景：

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

### 组合多个数据源

组合多个来源的数据：

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

## TypeScript 支持

该库采用 TypeScript 优先的方法构建，并提供完整的类型推断：

```ts
// 类型会自动推断
const userDataSource = makePlainQueryDataSource({
  name: 'user',
  fetch: skipContext(async (params: {userId: number}): Promise<User> => {
    // 返回类型被推断为 User
  }),
});

// Hook 的返回类型会自动类型化
const {data} = useQueryData(userDataSource, {userId: 123});
// data 的类型为 User | undefined
```

### 自定义错误类型

定义并使用自定义错误类型：

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

## 贡献指南

请阅读 [CONTRIBUTING.md](CONTRIBUTING.md) 以了解我们的行为准则以及提交拉取请求的流程详情。

## 许可证

MIT 许可证。详情请参阅 [LICENSE](LICENSE) 文件。