# Axios Wrapper
Cette bibliothèque fournit un wrapper pratique autour d'Axios, en ajoutant à ses fonctionnalités l'annulation automatique des requêtes concurrentes.

## Install

```shell
npm install --save-dev @gravity-ui/axios-wrapper
```

## API HTTP

### Paramètres du constructeur

##### config [optionnel]
La configuration d'une instance `axios`.

##### collector [optionnel]
La configuration du collecteur de requêtes est un objet :
```json
{
    "collectErrors": 10,
    "collectRequests": 10
}
```

### Méthodes de base
Le wrapper fournit les méthodes HTTP `get`, `head`, `put`, `post`, `delete`.

Les méthodes `get` et `head` ont la signature `(url, params, options)` ; `put`, `post`, tandis que la méthode `delete`
a la signature `(url, data, params, options)`.

L'argument `params` représente les paramètres de la chaîne de requête, tandis que `options` contient les paramètres de la requête.

Actuellement, 4 paramètres de requête sont pris en charge :
- `concurrentId (string)` : identifiant de requête optionnel
- `collectRequest (bool)` : indicateur optionnel indiquant si la requête doit être journalisée (par défaut `true`)
- `requestConfig (object)` : configuration optionnelle avec des paramètres de requête personnalisés
- `headers (object)` : objet optionnel avec des en-têtes de requête personnalisés.
- `timeout (number)` : délai d'attente optionnel pour la requête
- `onDownloadProgress (function)` : rappel optionnel pour le traitement de la progression du téléchargement de fichiers

### En-têtes
La méthode `setDefaultHeader({name (string), value (string), methods (array)})` permet d'ajouter un en-tête de requête par défaut.

Les arguments `name` et `value` sont requis, l'argument optionnel `methods` spécifie toutes les méthodes qui reçoivent ces en-têtes par défaut (par défaut, toutes les méthodes les reçoivent).

### CSRF
La méthode `setCSRFToken` permet de spécifier un jeton CSRF, qui sera ajouté à toutes les requêtes `put`, `post` et `delete`.

### Requêtes concurrentes
Parfois, il est préférable d'annuler une requête en cours si ses résultats ne sont plus nécessaires. Pour y parvenir, il suffit de passer l'identifiant `concurrentId` dans les `options` de la requête. Lorsque la prochaine requête avec le même `concurrentId` est effectuée, la requête précédente avec cet identifiant sera annulée.

Il est également possible d'annuler manuellement une requête en appelant la méthode `cancelRequest(concurrentId)`.

### Collecte des requêtes
Il est possible de configurer la collecte des requêtes dans le stockage local en utilisant l'option `collector`. Elle stocke toutes les requêtes et les erreurs séparément. L'instance `apiInstance` suivante conservera les 10 dernières requêtes (réussies ou non) et les 10 dernières requêtes erronées.
```javascript
const apiInstance = new API({
    collector: {
        collectErrors: 10,
        collectRequests: 10
    }
});
```

Pour obtenir les requêtes enregistrées, il faut appeler la méthode `getCollectedRequests`, qui retourne l'objet
`{errors: [...], requests: [...]}`.

### Utilisation
L'utilisation recommandée consiste à sous-classer la classe de base `AxiosWrapper` :
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

Lorsque le paramètre `baseURL` est passé dans la configuration `axios`, tous les chemins de requête seront ajoutés à celui-ci.
```javascript
const apiInstance = new API({
    config: {
        baseURL: '/api/v2'
    }
});
```