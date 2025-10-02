# Axios Wrapper

<p align="center">
  <a href="README.md">English</a> |
  <a href="README-zh.md">简体中文</a> |
  <a href="README-de.md">Deutsch</a> |
  <span>Español</span> |
  <a href="README-fr.md">Français</a>
</p>

Esta biblioteca proporciona un envoltorio conveniente alrededor de Axios, agregando la cancelación automática de solicitudes concurrentes a sus características.

## Instalación

```shell
npm install --save-dev @gravity-ui/axios-wrapper
```

## API HTTP

### Parámetros del constructor

##### config [opcional]

La configuración de una instancia de `axios`.

##### collector [opcional]

La configuración del colector de solicitudes es un objeto:

```json
{
  "collectErrors": 10,
  "collectRequests": 10
}
```

### Métodos básicos

El envoltorio proporciona los métodos HTTP `get`, `head`, `put`, `post`, `delete`.

Los métodos `get` y `head` tienen la firma `(url, params, options)`; mientras que `put`, `post` y el método `delete` tienen la firma `(url, data, params, options)`.

El argumento `params` representa los parámetros de la cadena de consulta, mientras que `options` son las configuraciones de la solicitud.

Actualmente se admiten las siguientes configuraciones de solicitud:

- `concurrentId (string)`: ID de solicitud opcional
- `collectRequest (bool)`: Bandera opcional que indica si la solicitud debe registrarse (por defecto `true`)
- `requestConfig (object)`: Configuración opcional con parámetros de solicitud personalizados
- `headers (object)`: Objeto opcional con encabezados de solicitud personalizados
- `timeout (number)`: Tiempo de espera opcional para la solicitud
- `onDownloadProgress (function)`: Callback opcional para procesar el progreso de descarga de archivos

### Encabezados

El método `setDefaultHeader({name (string), value (string), methods (array)})` permite agregar un encabezado de solicitud predeterminado.

Los argumentos `name` y `value` son obligatorios; el argumento opcional `methods` especifica todos los métodos que recibirán esos encabezados predeterminados (por defecto, todos los métodos los recibirán).

### CSRF

El método `setCSRFToken` permite especificar el token CSRF, que se agregará a todas las solicitudes `put`, `post` y `delete`.

### Solicitudes concurrentes

A veces es mejor cancelar una solicitud en curso si sus resultados ya no son necesarios. Para lograr esto, se debe pasar el ID `concurrentId` en las `options` de la solicitud. Cuando se realiza la siguiente solicitud con el mismo `concurrentId`, la solicitud anterior con ese ID se cancelará.

También es posible cancelar una solicitud manualmente invocando el método `cancelRequest(concurrentId)`.

### Recolección de solicitudes

Es posible configurar la recolección de solicitudes en el almacenamiento local utilizando la opción `collector`. Almacena todas las solicitudes y errores por separado. La siguiente instancia de `apiInstance` mantendrá las 10 solicitudes más recientes (tanto exitosas como fallidas) y las 10 solicitudes erróneas más recientes.

```javascript
const apiInstance = new API({
  collector: {
    collectErrors: 10,
    collectRequests: 10,
  },
});
```

Para obtener las solicitudes guardadas, se debe invocar el método `getCollectedRequests`, que devuelve el objeto `{errors: [...], requests: [...]}`.

### Uso

El uso recomendado es subclasificar la clase base `AxiosWrapper`:

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

Cuando se pasa el parámetro `baseURL` en la configuración de `axios`, todas las rutas solicitadas se agregarán a él.

```javascript
const apiInstance = new API({
  config: {
    baseURL: '/api/v2',
  },
});
```