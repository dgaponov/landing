# Axios Wrapper
이 라이브러리는 Axios를 편리하게 래핑하여 동시 요청의 자동 취소를 기능으로 추가합니다.

## 설치

```shell
npm install --save-dev @gravity-ui/axios-wrapper
```

## HTTP API

### 생성자 매개변수

##### config [선택]
`axios` 인스턴스의 구성입니다.

##### collector [선택]
요청 수집기 구성은 다음과 같은 객체입니다:
```json
{
    "collectErrors": 10,
    "collectRequests": 10
}
```

### 기본 메서드
래퍼는 `get`, `head`, `put`, `post`, `delete` 등의 HTTP 메서드를 제공합니다.

`get` 및 `head` 메서드는 `(url, params, options)` 시그니처를 가지며, `put`, `post`, `delete` 메서드는 `(url, data, params, options)` 시그니처를 가집니다.

`params` 인수는 쿼리 문자열 매개변수를 나타내며, `options`는 요청 설정입니다.

현재 지원되는 4가지 요청 설정은 다음과 같습니다:
- `concurrentId (string)`: 선택적 요청 ID
- `collectRequest (bool)`: 선택적 플래그로, 요청을 로깅할지 여부를 나타냅니다 (기본값 `true`)
- `requestConfig (object)`: 사용자 지정 요청 매개변수가 포함된 선택적 구성
- `headers (object)`: 사용자 지정 요청 헤더가 포함된 선택적 객체
- `timeout (number)`: 선택적 요청 타임아웃
- `onDownloadProgress (function)`: 파일 다운로드 진행 상황을 처리하는 선택적 콜백

### 헤더
`setDefaultHeader({name (string), value (string), methods (array)})` 메서드를 사용하면 기본 요청 헤더를 추가할 수 있습니다.

`name`과 `value` 인수는 필수이며, 선택적 인수 `methods`는 해당 기본 헤더를 받을 모든 메서드를 지정합니다 (기본적으로 모든 메서드가 해당 헤더를 받습니다).

### CSRF
`setCSRFToken` 메서드를 사용하면 CSRF 토큰을 지정할 수 있으며, 이는 모든 `put`, `post`, `delete` 요청에 추가됩니다.

### 동시 요청
때때로 이전 요청의 결과가 더 이상 필요하지 않으면 해당 요청을 취소하는 것이 좋습니다. 이를 위해 요청의 `options`에 `concurrentId` ID를 전달합니다. 동일한 `concurrentId`를 가진 다음 요청이 발생하면 해당 ID의 이전 요청이 취소됩니다.

`cancelRequest(concurrentId)` 메서드를 호출하여 요청을 수동으로 취소할 수도 있습니다.

### 요청 수집
`collector` 옵션을 사용하여 로컬 스토리지에 요청을 수집하도록 설정할 수 있습니다. 성공한 요청과 오류를 별도로 저장합니다. 다음 `apiInstance`는 성공 및 실패한 최근 10개의 요청과 최근 10개의 오류 요청을 유지합니다.
```javascript
const apiInstance = new API({
    collector: {
        collectErrors: 10,
        collectRequests: 10
    }
});
```

저장된 요청을 가져오려면 `getCollectedRequests` 메서드를 호출하면 `{errors: [...], requests: [...]}` 객체를 반환합니다.

### 사용법
권장되는 사용법은 기본 `AxiosWrapper` 클래스를 상속하는 것입니다:
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

`axios` 구성에 `baseURL` 매개변수가 전달되면 모든 요청 경로가 이에 추가됩니다.
```javascript
const apiInstance = new API({
    config: {
        baseURL: '/api/v2'
    }
});
```