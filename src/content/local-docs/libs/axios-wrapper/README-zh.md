# Axios Wrapper

这个库为 Axios 提供了一个便捷的包装器，添加了自动取消并发请求的功能。

## 安装

```shell
npm install --save-dev @gravity-ui/axios-wrapper
```

## HTTP API

### 构造函数参数

##### config [可选]

`axios` 实例的配置。

##### collector [可选]

请求收集器的配置是一个对象：

```json
{
  "collectErrors": 10,
  "collectRequests": 10
}
```

### 基本方法

包装器提供了 HTTP 方法 `get`、`head`、`put`、`post`、`delete`。

方法 `get` 和 `head` 的签名是 `(url, params, options)`；`put`、`post` 和 `delete` 方法的签名是 `(url, data, params, options)`。

`params` 参数表示查询字符串参数，而 `options` 是请求设置。

目前支持 4 种请求设置：

- `concurrentId (string)`：可选的请求 ID
- `collectRequest (bool)`：可选标志，指示是否记录该请求（默认为 `true`）
- `requestConfig (object)`：可选配置，包含自定义请求参数
- `headers (object)`：可选对象，包含自定义请求头
- `timeout (number)`：可选的请求超时时间
- `onDownloadProgress (function)`：可选回调，用于处理文件下载进度

### 请求头

`setDefaultHeader({name (string), value (string), methods (array)})` 方法允许添加默认请求头。

参数 `name` 和 `value` 是必需的，可选参数 `methods` 指定所有将获得这些默认头的 HTTP 方法（默认为所有方法）。

### CSRF

`setCSRFToken` 方法允许指定 CSRF 令牌，该令牌将被添加到所有 `put`、`post` 和 `delete` 请求中。

### 并发请求

有时，如果请求的结果不再需要，最好取消正在进行的请求。要实现这一点，可以在请求的 `options` 中传递 `concurrentId` ID。当发生下一个具有相同 `concurrentId` 的请求时，先前的具有该 ID 的请求将被取消。

也可以通过调用 `cancelRequest(concurrentId)` 方法手动取消请求。

### 请求收集

可以通过 `collector` 选项将请求收集到本地存储中。它会分别存储所有请求和错误。以下 `apiInstance` 将保留最近 10 个请求（包括成功和失败的）和最近 10 个错误请求。

```javascript
const apiInstance = new API({
  collector: {
    collectErrors: 10,
    collectRequests: 10,
  },
});
```

要获取保存的请求，可以调用 `getCollectedRequests` 方法，它返回对象 `{errors: [...], requests: [...]}`。

### 使用示例

建议的使用方式是子类化基础的 `AxiosWrapper` 类：

```javascript
export class API extends AxiosWrapper {
  getProjects() {
    return this.get('/projects');
  }
  getSensors({project, selectors}) {
    return this.get(`/projects/${project}/sensors`, {selectors, pageSize: 200});
  }
  getNames({project, selectors}) {
    return this.get(`/projects/${project}/sensors/names`, {selectors});
  }
  getLabels({project, names, selectors}) {
    return this.get(`/projects/${project}/sensors/labels`, {names, selectors});
  }
}
```

如果在 `axios` 配置中传递了 `baseURL` 参数，所有请求的路径名都会被附加到它后面。

```typescript
const apiInstance = new API({
  config: {
    baseURL: '/api/v2',
  },
});
```