# NodeKit

NodeKit — это простой набор инструментов для ваших приложений, скриптов и библиотек на Node.js. Он предоставляет функциональность для логирования, телеметрии, конфигурации и обработки ошибок, чтобы у вас была знакомая основа для разных проектов.

## Начало работы

Добавьте зависимость в ваш проект:

```bash
npm install --save @gravity-ui/nodekit
```

Затем импортируйте и инициализируйте NodeKit в вашем приложении:

```typescript
import {NodeKit} from '@gravity-ui/nodekit';

const nodeKit = new NodeKit();
nodekit.ctx.log('App is ready');
```

## Документация

Дополнительную документацию смотрите в директории `docs/`:

- [`docs/configuration.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/configuration.md) описывает, как настроить сам NodeKit и приложения на его основе
- [`docs/contexts.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/contexts.md) объясняет концепцию контекстов NodeKit, логирование и трассировку
- [`docs/app-error.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/app-error.md) содержит описание полезного пользовательского класса ошибок, который предоставляет NodeKit для ваших приложений
- [`docs/utils.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/utils.md) перечисляет дополнительные вспомогательные функции, входящие в состав NodeKit

## Вклад в проект

### Начало работы

Склонируйте репозиторий NodeKit и примеры приложений:

```bash
git clone git@github.com:gravity-ui/nodekit
git clone git@github.com:gravity-ui/nodekit-examples
```

Подключите ваш NodeKit к npm и запустите компилятор:

```bash
cd nodekit && npm link && npm run dev
```

Затем, в другом терминале, перейдите в папку с примерами, выберите интересующий вас, подключите там ваш NodeKit и запустите приложение:

```bash
cd nodekit-examples/basic-app && npm i && npm link @gravity-ui/nodekit
npm run dev
```

На этом этапе вы можете вносить изменения как в NodeKit, так и в демонстрационное приложение, и видеть результаты в реальном времени.