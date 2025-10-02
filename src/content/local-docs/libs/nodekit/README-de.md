# NodeKit

NodeKit ist ein einfaches Toolkit für Ihre Node.js-Apps, Skripte und Bibliotheken. Es bietet Funktionen für Logging, Telemetrie, Konfiguration und Fehlerbehandlung, damit Sie eine vertraute Grundlage in Ihren verschiedenen Projekten haben.

## Erste Schritte

Fügen Sie die Abhängigkeit zu Ihrem Projekt hinzu:

```bash
npm install --save @gravity-ui/nodekit
```

Importieren Sie dann NodeKit in Ihrer Anwendung und initialisieren Sie es:

```typescript
import {NodeKit} from '@gravity-ui/nodekit';

const nodeKit = new NodeKit();
nodekit.ctx.log('App is ready');
```

## Dokumentation

Lesen Sie das `docs/`-Verzeichnis für zusätzliche Dokumentation:

- [`docs/configuration.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/configuration.md) beschreibt, wie Sie NodeKit selbst und Ihre darauf basierenden Anwendungen konfigurieren können
- [`docs/contexts.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/contexts.md) erläutert das Konzept der NodeKit-Kontexte, Logging und Tracing
- [`docs/app-error.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/app-error.md) enthält eine Beschreibung der nützlichen benutzerdefinierten Fehlerklasse, die NodeKit für Ihre Anwendungen bereitstellt
- [`docs/utils.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/utils.md) listet einige zusätzliche Hilfsfunktionen auf, die mit NodeKit gebündelt sind

## Mitwirken

### Erste Schritte

Holen Sie sich Kopien des NodeKit-Repositorys und der Beispiele-Anwendungen:

```bash
git clone git@github.com:gravity-ui/nodekit
git clone git@github.com:gravity-ui/nodekit-examples
```

Verknüpfen Sie Ihr NodeKit mit npm und starten Sie den Compiler:

```bash
cd nodekit && npm link && npm run dev
```

Öffnen Sie dann in einem anderen Terminal die Beispiele, wählen Sie das aus, das Sie interessiert, verknüpfen Sie dort Ihr NodeKit und starten Sie die App:

```bash
cd nodekit-examples/basic-app && npm i && npm link @gravity-ui/nodekit
npm run dev
```

Zu diesem Zeitpunkt können Sie Änderungen sowohl an NodeKit als auch an der Demo-App vornehmen und die Ergebnisse in Echtzeit sehen.