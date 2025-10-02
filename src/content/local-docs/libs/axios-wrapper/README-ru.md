# Axios Wrapper
Эта библиотека предоставляет удобную обёртку вокруг Axios, добавляя к её возможностям автоматическую отмену параллельных запросов.

## Установка

```shell
npm install --save-dev @gravity-ui/axios-wrapper
```

## HTTP API

### Параметры конструктора

##### config [optional]
Конфигурация экземпляра `axios`.

##### collector [optional]
Конфигурация коллектора запросов — это объект:
```json
{
    "collectErrors": 10,
    "collectRequests": 10
}
```

### Основные методы
Обёртка предоставляет HTTP-методы `get`, `head`, `put`, `post`, `delete`.

Методы `get` и `head` имеют сигнатуру `(url, params, options)`; методы `put`, `post`, а также `delete`
имеют сигнатуру `(url, data, params, options)`.

Аргумент `params` отвечает за параметры строки запроса, а `options` — за настройки запроса.

В настоящее время поддерживается 4 настройки запроса:
- `concurrentId (string)`: опциональный идентификатор запроса
- `collectRequest (bool)`: опциональный флаг, указывающий, нужно ли логировать запрос (по умолчанию `true`)
- `requestConfig (object)`: опциональная конфигурация с пользовательскими параметрами запроса
- `headers (object)`: опциональный объект с пользовательскими заголовками запроса
- `timeout (number)`: опциональный таймаут запроса
- `onDownloadProgress (function)`: опциональный колбэк для обработки прогресса загрузки файла

### Заголовки
Метод `setDefaultHeader({name (string), value (string), methods (array)})` позволяет добавить заголовок по умолчанию для запросов.

Аргументы `name` и `value` обязательны, опциональный аргумент `methods` указывает все методы, которые получат эти заголовки по умолчанию (по умолчанию все методы получат эти заголовки).

### CSRF
Метод `setCSRFToken` позволяет указать CSRF-токен, который будет добавлен ко всем запросам `put`, `post` и `delete`.

### Параллельные запросы
Иногда лучше отменить запрос в процессе выполнения, если его результаты больше не нужны. Чтобы это произошло, в опции запроса нужно передать идентификатор `concurrentId`. Когда выполняется следующий запрос с тем же `concurrentId`, предыдущий запрос с этим идентификатором будет отменён.

Также запрос можно отменить вручную, вызвав метод `cancelRequest(concurrentId)`.

### Сбор запросов
Возможно настроить сбор запросов в локальное хранилище с помощью опции `collector`. Он хранит все запросы и ошибки отдельно. Следующий экземпляр `apiInstance` будет хранить 10 последних запросов (как успешных, так и неуспешных) и 10 последних ошибочных запросов.
```javascript
const apiInstance = new API({
    collector: {
        collectErrors: 10,
        collectRequests: 10
    }
});
```

Чтобы получить сохранённые запросы, нужно вызвать метод `getCollectedRequests`, который возвращает объект
`{errors: [...], requests: [...]}`.

### Использование
Рекомендуется использовать подкласс базового класса `AxiosWrapper`:
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

Если в конфигурацию `axios` передан параметр `baseURL`, все пути запросов будут присоединены к нему.
```javascript
const apiInstance = new API({
    config: {
        baseURL: '/api/v2'
    }
});
```