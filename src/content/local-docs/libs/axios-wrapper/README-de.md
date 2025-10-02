# Axios Wrapper

<p align="center">
  <a href="README.md">English</a> |
  <a href="README-zh.md">简体中文</a> |
  <span>Deutsch</span> |
  <a href="README-fr.md">Français</a>
</p>

Diese Bibliothek bietet einen bequemen Wrapper um Axios und erweitert dessen Funktionen um das automatische Abbrechen paralleler Anfragen.

## Installation

```shell
npm install --save-dev @gravity-ui/axios-wrapper
```

## HTTP-API

### Konstruktorparameter

##### config [optional]

Die Konfiguration einer `axios`-Instanz.

##### collector [optional]

Die Konfiguration des Anfragesammlers ist ein Objekt:

```json
{
  "collectErrors": 10,
  "collectRequests": 10
}
```

### Basis-Methoden

Der Wrapper stellt die HTTP-Methoden `get`, `head`, `put`, `post` und `delete` zur Verfügung.

Die Methoden `get` und `head` haben die Signatur `(url, params, options)`; `put`, `post` und die `delete`-Methode verwenden die Signatur `(url, data, params, options)`.

Das Argument `params` steht für Query-String-Parameter, während `options` Einstellungen für die Anfrage enthält.

Derzeit werden folgende Anfrageeinstellungen unterstützt:

- `concurrentId (string)`: optionale Anfrage-ID
- `collectRequest (bool)`: optionale Flagge, die angibt, ob die Anfrage protokolliert werden soll (Standard: `true`)
- `requestConfig (object)`: optionale Konfiguration mit benutzerdefinierten Anfragemparametern
- `headers (object)`: optionales Objekt mit benutzerdefinierten Anfrage-Headern
- `timeout (number)`: optionale Anfrage-Timeout-Dauer
- `onDownloadProgress (function)`: optionale Rückruf-Funktion zur Verarbeitung des Download-Fortschritts von Dateien

### Header

Die Methode `setDefaultHeader({name (string), value (string), methods (array)})` ermöglicht das Hinzufügen eines Standard-Anfrage-Headers.

Die Argumente `name` und `value` sind erforderlich; das optionale Argument `methods` legt fest, für welche Methoden diese Standard-Header gelten sollen (standardmäßig für alle Methoden).

### CSRF

Die Methode `setCSRFToken` ermöglicht die Festlegung eines CSRF-Tokens, das automatisch zu allen `put`-, `post`- und `delete`-Anfragen hinzugefügt wird.

### Parallele Anfragen

Manchmal ist es sinnvoll, eine laufende Anfrage abzubrechen, wenn ihre Ergebnisse nicht mehr benötigt werden. Um dies zu erreichen, übergeben Sie der `options` der Anfrage eine `concurrentId`. Sobald eine neue Anfrage mit derselben `concurrentId` eingeht, wird die vorherige Anfrage mit dieser ID automatisch abgebrochen.

Sie können eine Anfrage auch manuell abbrechen, indem Sie die Methode `cancelRequest(concurrentId)` aufrufen.

### Sammeln von Anfragen

Mit der `collector`-Option können Sie das Sammeln von Anfragen im Local Storage einrichten. Dabei werden Anfragen und Fehler getrennt gespeichert. Die folgende `apiInstance` speichert die 10 letzten Anfragen (erfolgreich oder nicht) sowie die 10 letzten fehlerhaften Anfragen.

```javascript
const apiInstance = new API({
  collector: {
    collectErrors: 10,
    collectRequests: 10,
  },
});
```

Um die gespeicherten Anfragen abzurufen, rufen Sie die Methode `getCollectedRequests` auf, die ein Objekt der Form `{errors: [...], requests: [...]}` zurückgibt.

### Verwendung

Die empfohlene Vorgehensweise ist, die Basis-Klasse `AxiosWrapper` zu erweitern:

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

Wenn Sie den `baseURL`-Parameter in die `axios`-Konfiguration übergeben, werden alle angeforderten Pfade automatisch an diese angehängt.

```javascript
const apiInstance = new API({
  config: {
    baseURL: '/api/v2',
  },
});
```