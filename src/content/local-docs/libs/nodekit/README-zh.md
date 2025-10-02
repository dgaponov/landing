# NodeKit

NodeKit 是一个简单的工具包，适用于您的 Node.js 应用、脚本和库。它提供了日志记录、遥测、配置和错误处理功能，让您在不同项目中拥有熟悉的基础架构。

## 快速开始

将依赖项添加到您的项目中：

```bash
npm install --save @gravity-ui/nodekit
```

然后在您的应用中导入并初始化 NodeKit：

```typescript
import {NodeKit} from '@gravity-ui/nodekit';

const nodeKit = new NodeKit();
nodekit.ctx.log('App is ready');
```

## 文档

请查看 `docs/` 目录中的附加文档：

- [`docs/configuration.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/configuration.md) 说明了如何配置 NodeKit 本身以及基于 NodeKit 的应用
- [`docs/contexts.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/contexts.md) 介绍了 NodeKit 上下文、日志记录和追踪的概念
- [`docs/app-error.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/app-error.md) 包含了 NodeKit 为您的应用提供的实用自定义错误类的描述
- [`docs/utils.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/utils.md) 列出了与 NodeKit 捆绑的一些附加辅助函数

## 贡献指南

### 快速开始

获取 NodeKit 仓库和示例应用的副本：

```bash
git clone git@github.com:gravity-ui/nodekit
git clone git@github.com:gravity-ui/nodekit-examples
```

将您的 NodeKit 链接到 npm 并启动编译器：

```bash
cd nodekit && npm link && npm run dev
```

然后，在另一个终端中，转到示例目录，打开您感兴趣的示例，将您的 NodeKit 链接到那里，然后启动应用：

```bash
cd nodekit-examples/basic-app && npm i && npm link @gravity-ui/nodekit
npm run dev
```

此时，您可以同时修改 NodeKit 和演示应用，并实时查看结果。