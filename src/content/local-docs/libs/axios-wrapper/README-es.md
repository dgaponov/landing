<p align="center">
  <a href="README.md">English</a> |
  <a href="README-zh.md">简体中文</a> |
  <a href="README-de.md">Deutsch</a> |
  <a href="README-fr.md">Français</a> |
  <span>Español</span>
</p>

# Envoltorio de Axios
1
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

La configuración del recolector de solicitudes es un objeto:

```json
{
  "collectErrors": 10,
  "collectRequests": 10
}
```

### Métodos básicos

El envoltorio proporciona métodos HTTP como `get`, `head`, `put`, `post` y `delete`.

Los métodos `get` y `head` tienen la firma `(url, params, options)`; `put`, `post` y el método `delete` tienen la firma `(url, data, params, options)`.

El argumento `params` representa los parámetros de la cadena de consulta, mientras que `options` son las configuraciones de la solicitud.

Actualmente se admiten 4 configuraciones de solicitud:

- `concurrentId (string)`: ID de solicitud opcional
- `collectRequest (bool)`: bandera opcional que indica si la solicitud debe registrarse (por defecto `true`)
- `requestConfig (object)`: configuración opcional con parámetros personalizados de la solicitud
- `headers (object)`: objeto opcional con encabezados personalizados de la solicitud.
- `timeout (number)`: tiempo de espera opcional para la solicitud
- `onDownloadProgress (function)`: callback opcional para procesar el progreso de descarga de archivos

### Encabezados

El método `setDefaultHeader({name (string), value (string), methods (array)})` permite agregar un encabezado de solicitud predeterminado.

Los argumentos `name` y `value` son obligatorios; el argumento opcional `methods` especifica todos los métodos que reciben estos encabezados predeterminados (por defecto, todos los métodos los reciben).

### CSRF

El método `setCSRFToken` permite especificar un token CSRF, que se agregará a todas las solicitudes `put`, `post` y `delete`.

### Solicitudes concurrentes

A veces es mejor cancelar la solicitud en curso si sus resultados ya no son necesarios. Para lograr esto, se debe pasar el ID `concurrentId` en las `options` de la solicitud. Cuando ocurra la siguiente solicitud con el mismo `concurrentId`, la solicitud anterior con ese ID se cancelará.

También se puede cancelar una solicitud manualmente invocando el método `cancelRequest(concurrentId)`.

### Recolección de solicitudes

Es posible configurar la recolección de solicitudes en el almacenamiento local usando la opción `collector`. Almacena todas las solicitudes y errores por separado. La siguiente `apiInstance` mantendrá las 10 solicitudes más recientes (tanto exitosas como no) y las 10 solicitudes erróneas más recientes.

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

El uso sugerido es subclasificar la clase base `AxiosWrapper`:

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

Cuando se pasa el parámetro `baseURL` en la configuración de `axios`, todas las rutas de las solicitudes se agregarán a él.

```javascript
const apiInstance = new API({
  config: {
    baseURL: '/api/v2',
  },
});
```