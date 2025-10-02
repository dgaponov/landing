# NodeKit

NodeKit — это простой набор инструментов для ваших приложений, скриптов и библиотек на Node.js. Он предоставляет функциональность для логирования, телеметрии, конфигурации и обработки ошибок, чтобы у вас была знакомая основа в разных проектах.

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

Смотрите директорию `docs/` для дополнительной документации:

- [`docs/configuration.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/configuration.md) описывает, как настроить сам NodeKit и ваши приложения на его основе
- [`docs/contexts.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/contexts.md) объясняет концепцию контекстов NodeKit, логирование и трассировку
- [`docs/app-error.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/app-error.md) содержит описание полезного пользовательского класса ошибок, который предоставляет NodeKit для ваших приложений
- [`docs/utils.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/utils.md) перечисляет дополнительные вспомогательные функции, входящие в состав NodeKit

## Вклад в разработку

### Начало работы

Склонируйте репозитории NodeKit и примеров приложений:

```bash
git clone git@github.com:gravity-ui/nodekit
git clone git@github.com:gravity-ui/nodekit-examples
```

Создайте символическую ссылку для nodekit в npm и запустите компилятор:

```bash
cd nodekit && npm link && npm run dev
```

Затем, в другом терминале, перейдите в папку с примерами, выберите интересующий вас, создайте там ссылку на nodekit и запустите приложение:

```bash
cd nodekit-examples/basic-app && npm i && npm link @gravity-ui/nodekit
npm run dev
```

Теперь вы можете вносить изменения как в NodeKit, так и в демонстрационное приложение, и видеть результаты в реальном времени.