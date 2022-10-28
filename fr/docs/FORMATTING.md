## Formatage du Code

Le formatage est subjectif. Comme beaucoup de règles ici, il n'y a pas de règle
stricte que vous devez suivre. Le but principal est *NE PAS DISCUTER* sur le formatage.
Il existe des tonnes d'outils pour automatiser cela. Utilisez-en un! C'est une perte
de temps et d'argent pour les ingénieurs de discuter du formatage. La règle générale
à suivre est de *conserver des règles de formatage cohérentes*.

Pour TypeScript, il existe un outil puissant appelé [TSLint](https://palantir.github.io/tslint/).
Il s'agit d'un outil d'analyse statique qui peut vous aider à améliorer considérablement
la lisibilité et la maintenabilité de votre code. Il existe des configurations
TSLint prêtes à l'emploi que vous pouvez référencer dans vos projets:

- [TSLint Config Standard](https://www.npmjs.com/package/tslint-config-standard) - règles de style standard

- [TSLint Config Airbnb](https://www.npmjs.com/package/tslint-config-airbnb) - guide de style Airbnb

- [TSLint Clean Code](https://www.npmjs.com/package/tslint-clean-code) - Les règles TSLint inspirées du [Clean Code: A Handbook of Agile Software Craftsmanship](https://www.amazon.ca/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)

- [TSLint react](https://www.npmjs.com/package/tslint-react) - règles sur les peluches liées à React et JSX

- [TSLint + Prettier](https://www.npmjs.com/package/tslint-config-prettier) - règles de "lint" pour [Prettier](https://github.com/prettier/prettier) formateur de code

- [ESLint rules for TSLint](https://www.npmjs.com/package/tslint-eslint-rules) - règles ESLint pour TypeScript

- [Immutable](https://www.npmjs.com/package/tslint-immutable) - règles pour désactiver la mutation dans TypeScript

Reportez-vous également à cette excellente source
[TypeScript StyleGuide and Coding Conventions](https://basarat.gitbook.io/typescript/styleguide).

### Utiliser une capitalisation cohérente

La capitalisation vous en dit long sur vos variables, fonctions, etc. Ces règles
sont subjectives, donc votre équipe peut choisir ce qu'elle veut. Le fait est,
peu importe ce que vous choisissez tous, juste *soyez cohérent*.

**Mal:**

```ts
const DAYS_IN_WEEK = 7;
const daysInMonth = 30;

const songs = ['Back In Black', 'Stairway to Heaven', 'Hey Jude'];
const Artists = ['ACDC', 'Led Zeppelin', 'The Beatles'];

function eraseDatabase() {}
function restore_database() {}

type animal = { /* ... */ }
type Container = { /* ... */ }
```

**Bien:**

```ts
const DAYS_IN_WEEK = 7;
const DAYS_IN_MONTH = 30;

const SONGS = ['Back In Black', 'Stairway to Heaven', 'Hey Jude'];
const ARTISTS = ['ACDC', 'Led Zeppelin', 'The Beatles'];

function eraseDatabase() {}
function restoreDatabase() {}

type Animal = { /* ... */ }
type Container = { /* ... */ }
```

Utiliser `PascalCase` de préférence pour les noms de classe, d'interface, de type et d'espace de noms.
Utiliser `camelCase` de préférence pour les variables, les fonctions et les membres de la classe.

### Les "callers" et "callees" doivent être proches

Si une fonction en appelle une autre, gardez ces fonctions verticalement fermées
dans le fichier source. Idéalement, gardez le "caller" juste au-dessus du "callee".
Nous avons tendance à lire le code de haut en bas, comme un journal. Pour cette raison,
faites lire votre code de cette façon.

**Mal:**

```ts
class PerformanceReview {
  constructor(private readonly employee: Employee) {
  }

  private lookupPeers() {
    return db.lookup(this.employee.id, 'peers');
  }

  private lookupManager() {
    return db.lookup(this.employee, 'manager');
  }

  private getPeerReviews() {
    const peers = this.lookupPeers();
    // ...
  }

  review() {
    this.getPeerReviews();
    this.getManagerReview();
    this.getSelfReview();

    // ...
  }

  private getManagerReview() {
    const manager = this.lookupManager();
  }

  private getSelfReview() {
    // ...
  }
}

const review = new PerformanceReview(employee);
review.review();
```

**Bien:**

```ts
class PerformanceReview {
  constructor(private readonly employee: Employee) {
  }

  review() {
    this.getPeerReviews();
    this.getManagerReview();
    this.getSelfReview();

    // ...
  }

  private getPeerReviews() {
    const peers = this.lookupPeers();
    // ...
  }

  private lookupPeers() {
    return db.lookup(this.employee.id, 'peers');
  }

  private getManagerReview() {
    const manager = this.lookupManager();
  }

  private lookupManager() {
    return db.lookup(this.employee, 'manager');
  }

  private getSelfReview() {
    // ...
  }
}

const review = new PerformanceReview(employee);
review.review();
```

### Organiser les importations (imports)

Avec des instructions d'importation propres et faciles à lire, vous pouvez rapidement
voir les dépendances du code actuel. Assurez-vous d'appliquer les bonnes pratiques
suivantes pour les instructions `import`:

- Les déclarations d'importation doivent être classées par ordre alphabétique et regroupées.
- Les importations non utilisées doivent être supprimées.
- Les importations nommées doivent être classées par ordre alphabétique (c'est-à-dire `import {A, B, C} de 'foo';`)
- Les sources d'importation doivent être classées par ordre alphabétique dans les groupes, c'est-à-dire: `import * as foo from 'a'; importer *comme barre de 'b';`
- Les groupes d'importations sont délimités par des lignes blanches.
- Les groupes doivent respecter l'ordre suivant:
  - Polyfills (c'est-à-dire `import 'reflect-metadata';`)
  - modules intégrés de Node (c'est-à-dire `import fs from 'fs';`)
  - modules externes (c'est-à-dire `import {query} from 'itiriri';`)
  - modules internes (c'est-à-dire `import {UserService} from 'src/services/userService';`)
  - modules d'un répertoire parent (c'est-à-dire `import foo from '../foo'; import qux from '../../foo/qux';`)
  - modules du même répertoire ou d'un répertoire frère (c'est-à-dire `import bar from './bar'; import baz from './bar/baz';`)

**Mal:**

```ts
import { TypeDefinition } from '../types/typeDefinition';
import { AttributeTypes } from '../model/attribute';
import { ApiCredentials, Adapters } from './common/api/authorization';
import fs from 'fs';
import { ConfigPlugin } from './plugins/config/configPlugin';
import { BindingScopeEnum, Container } from 'inversify';
import 'reflect-metadata';
```

**Bien:**

```ts
import 'reflect-metadata';

import fs from 'fs';
import { BindingScopeEnum, Container } from 'inversify';

import { AttributeTypes } from '../model/attribute';
import { TypeDefinition } from '../types/typeDefinition';

import { ApiCredentials, Adapters } from './common/api/authorization';
import { ConfigPlugin } from './plugins/config/configPlugin';
```

### Utiliser des alias de TypeScript

Créez des importations plus jolies en définissant les chemins d'accès et les attributs
baseUrl dans la section `compilerOptions` dans le fichier `tsconfig.json`.

Cela évitera de longs chemins relatifs lors des importations.

**Mal:**

```ts
import { UserService } from '../../../services/UserService';
```

**Bien:**

```ts
import { UserService } from '@services/UserService';
```

```js
// tsconfig.json
...
  "compilerOptions": {
    ...
    "baseUrl": "src",
    "paths": {
      "@services": ["services/*"]
    }
    ...
  }
...
```
