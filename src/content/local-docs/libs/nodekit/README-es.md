# NodeKit

NodeKit es un kit de herramientas sencillo para tus aplicaciones, scripts y bibliotecas de Node.js. Proporciona funcionalidades para el registro de logs, telemetría, configuración y manejo de errores, de modo que puedas tener una base familiar en tus diferentes proyectos.

## Getting started

Añade la dependencia a tu proyecto:

```bash
npm install --save @gravity-ui/nodekit
```

Luego, importa e inicializa NodeKit en tu aplicación:

```typescript
import {NodeKit} from '@gravity-ui/nodekit';

const nodeKit = new NodeKit();
nodekit.ctx.log('App is ready');
```

## Documentation

Consulta el directorio `docs/` para documentación adicional:

- [`docs/configuration.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/configuration.md) especifica cómo puedes configurar tanto NodeKit en sí como tus aplicaciones basadas en NodeKit
- [`docs/contexts.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/contexts.md) describe el concepto de contextos de NodeKit, logging y tracing
- [`docs/app-error.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/app-error.md) contiene la descripción de la clase de error personalizada útil que NodeKit proporciona para tus aplicaciones
- [`docs/utils.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/utils.md) enumera algunas funciones auxiliares adicionales que vienen empaquetadas con NodeKit

## Contributing

### Getting started

Obtén copias del repositorio de NodeKit y las aplicaciones de ejemplo:

```bash
git clone git@github.com:gravity-ui/nodekit
git clone git@github.com:gravity-ui/nodekit-examples
```

Enlaza tu NodeKit a npm e inicia el compilador:

```bash
cd nodekit && npm link && npm run dev
```

Luego, en otra terminal, ve al directorio de ejemplos, abre el que te interese, enlaza tu NodeKit allí y luego inicia la app:

```bash
cd nodekit-examples/basic-app && npm i && npm link @gravity-ui/nodekit
npm run dev
```

En este punto, puedes realizar cambios tanto en NodeKit como en la app de demostración, y ver los resultados en tiempo real.