# NodeKit

NodeKit est un kit d'outils simple pour vos applications, scripts et bibliothèques Node.js. Il fournit des fonctionnalités pour la journalisation, la télémétrie, la configuration et la gestion des erreurs, afin que vous puissiez disposer d'une base solide et familière dans vos différents projets.

## Pour commencer

Ajoutez la dépendance à votre projet :

```bash
npm install --save @gravity-ui/nodekit
```

Puis importez et initialisez NodeKit dans votre application :

```typescript
import {NodeKit} from '@gravity-ui/nodekit';

const nodeKit = new NodeKit();
nodekit.ctx.log('App is ready');
```

## Documentation

Consultez le répertoire `docs/` pour une documentation supplémentaire :

- [`docs/configuration.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/configuration.md) explique comment configurer NodeKit lui-même ainsi que vos applications basées sur NodeKit
- [`docs/contexts.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/contexts.md) décrit le concept des contextes NodeKit, la journalisation et le traçage
- [`docs/app-error.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/app-error.md) contient une description de la classe d'erreur personnalisée utile fournie par NodeKit pour vos applications
- [`docs/utils.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/utils.md) répertorie certaines fonctions utilitaires supplémentaires incluses avec NodeKit

## Contribution

### Pour commencer

Récupérez des copies du dépôt NodeKit et des applications d'exemple :

```bash
git clone git@github.com:gravity-ui/nodekit
git clone git@github.com:gravity-ui/nodekit-examples
```

Lie votre NodeKit à npm et lancez le compilateur :

```bash
cd nodekit && npm link && npm run dev
```

Ensuite, dans un autre terminal, allez dans les exemples, ouvrez celui qui vous intéresse, liez votre NodeKit là-bas, puis lancez l'application :

```bash
cd nodekit-examples/basic-app && npm i && npm link @gravity-ui/nodekit
npm run dev
```

À ce stade, vous pouvez apporter des modifications à la fois à NodeKit et à l'application de démonstration, et voir les résultats en temps réel.