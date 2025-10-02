# Axios Wrapper
Diese Bibliothek stellt einen praktischen Wrapper für Axios bereit, der das automatische Abbrechen gleichzeitiger Anfragen als zusätzliche Funktion bietet.

## Install

```shell
npm install --save-dev @gravity-ui/axios-wrapper
```

## HTTP API

### Constructor parameters

##### config [optional]
Die Konfiguration einer `axios`-Instanz.

##### collector [optional]
Die Konfiguration des Anfragen-Sammlers ist ein Objekt:
```json
{
    "collectErrors": 10,
    "collectRequests": 10
}
```

### Basic methods
Der Wrapper stellt die HTTP-Methoden `get`, `head`, `put`, `post`, `delete` zur Verfügung.

Die Methoden `get` und `head` haben die Signatur `(url, params, options)`; `put`, `post` und die `delete`-Methode haben die Signatur `(url, data, params, options)`.

Das `params`-Argument steht für Query-String-Parameter, während `options` Einstellungen für die Anfrage sind.

Derzeit werden folgende Anfrage-Einstellungen unterstützt:
- `concurrentId (string)`: optionale Anfrage-ID
- `collectRequest (bool)`: optionale Flagge, die angibt, ob die Anfrage protokolliert werden soll (Standard: `true`)
- `requestConfig (object)`: optionale Konfiguration mit benutzerdefinierten Anfrageparametern
- `headers (object)`: optionales Objekt mit benutzerdefinierten Anfrage-Headern.
- `timeout (number)`: optionale Anfrage-Timeout
- `onDownloadProgress (function)`: optionaler Callback zur Verarbeitung des Download-Fortschritts von Dateien

### Headers
Die Methode `setDefaultHeader({name (string), value (string), methods (array)})` ermöglicht das Hinzufügen eines Standard-Anfrage-Headers.

Die Argumente `name` und `value` sind erforderlich, das optionale Argument `methods` gibt alle Methoden an, die diese Standard-Header erhalten (standardmäßig erhalten alle Methoden diese Header).

### CSRF
Die Methode `setCSRFToken` ermöglicht die Angabe eines CSRF-Tokens, das zu allen `put`-, `post`- und `delete`-Anfragen hinzugefügt wird.

### Concurrent requests
Manchmal ist es besser, eine laufende Anfrage abzubrechen, wenn ihre Ergebnisse nicht mehr benötigt werden. Um dies zu erreichen, sollte man der `options` der Anfrage die `concurrentId` übergeben. Wenn die nächste Anfrage mit derselben `concurrentId` auftritt, wird die vorherige Anfrage mit dieser ID abgebrochen.

Man kann eine Anfrage auch manuell abbrechen, indem man die Methode `cancelRequest(concurrentId)` aufruft.

### Collecting requests
Es ist möglich, die Sammlung von Anfragen in den lokalen Speicher mit der `collector`-Option einzurichten. Sie speichert alle Anfragen und Fehler getrennt. Die folgende `apiInstance` speichert die 10 letzten Anfragen (sowohl erfolgreiche als auch nicht) und 10 letzte fehlerhafte Anfragen.
```javascript
const apiInstance = new API({
    collector: {
        collectErrors: 10,
        collectRequests: 10
    }
});
```

Um die gespeicherten Anfragen zu erhalten, muss man die Methode `getCollectedRequests` aufrufen, die das Objekt `{errors: [...], requests: [...]}` zurückgibt.

### Usage
Die empfohlene Verwendung ist, die Basis-Klasse `AxiosWrapper` zu erweitern:
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

Wenn der `baseURL`-Parameter in die `axios`-Konfiguration übergeben wird, werden alle angeforderten Pfade an diese angehängt.
```javascript
const apiInstance = new API({
    config: {
        baseURL: '/api/v2'
    }
});
```