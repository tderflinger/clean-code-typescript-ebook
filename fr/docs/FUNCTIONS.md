## Fonctions

### Paramètres d'une fonction (2 or moins idéalement)

Limiter le nombre de paramètres d'une fonction est extrêmement important car cela
facilite le test de votre fonction. En avoir plus de trois conduit à une explosion
combinatoire où chaque argument doit être testé séparément dans des tonnes de cas
différents.

Avoir un ou deux paramètres par fonction est idéal, et trois devraient être évités
si possible. Rien de plus que cela devrait être consolidé. Assez souvent, si vous
avez plus de deux arguments, votre fonction essaie d'en faire trop à la fois.
Dans les cas où ce n'est pas le cas, la plupart du temps, un objet de niveau
supérieur suffit comme argument.

Pensez à utiliser des objets littéraux si vous avez besoin de beaucoup plus de paramètres.

Pour rendre évidentes les attributs que la fonction attend, vous pouvez utiliser
la syntaxe de [déstructuration](https://basarat.gitbook.io/typescript/future-javascript/destructuring).
Cela présente quelques avantages :

1. Quand quelqu'un regarde la signature d’une fonction, il est immédiatement clair
lesquels des attributs sont en train d’être utilisées.

2. Il peut être utilisé pour simuler des paramètres avec des noms.

3. La déstructuration clone également les valeurs primitives spécifiées de l'objet
passé comme paramètre dans la fonction. Cela peut aider à prévenir les effets secondaires.
Remarque: les objets et les tableaux qui sont déstructurés à partir de l'objet
argument ne sont PAS clonés.

4. TypeScript vous avertit des attributs non-utilisés, qui seraient impossibles
sans déstructuration.

**Mal:**

```ts
function createMenu(title: string, body: string, buttonText: string, cancellable: boolean) {
  // ...
}

createMenu('Foo', 'Bar', 'Baz', true);
```

**Bien:**

```ts
function createMenu(options: { title: string, body: string, buttonText: string, cancellable: boolean }) {
  // ...
}

createMenu({
  title: 'Foo',
  body: 'Bar',
  buttonText: 'Baz',
  cancellable: true
});
```

Vous pouvez encore améliorer la lisibilité en utilisant [type aliases (ou type d'alias)](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-aliases):

```ts

type MenuOptions = { title: string, body: string, buttonText: string, cancellable: boolean };

function createMenu(options: MenuOptions) {
  // ...
}

createMenu({
  title: 'Foo',
  body: 'Bar',
  buttonText: 'Baz',
  cancellable: true
});
```

### Les fonctions devraient faire une chose

C'est de loin la règle la plus importante en ingénierie de logiciels. Lorsque
les fonctions font plus d'une chose, elles sont plus difficiles à composer, à
tester et à raisonner. Lorsque vous arrivez à isoler une fonction afin d’exécuter
une seule tâche, elle peut être facilement refactorisée et votre code sera beaucoup
plus net. Si vous ne prenez en compte ce qui est dit dans ce guide, vous serez en
avance sur de nombreux développeurs.

**Mal:**

```ts
function emailClients(clients: Client[]) {
  clients.forEach((client) => {
    const clientRecord = database.lookup(client);
    if (clientRecord.isActive()) {
      email(client);
    }
  });
}
```

**Bien:**

```ts
function emailClients(clients: Client[]) {
  clients.filter(isActiveClient).forEach(email);
}

function isActiveClient(client: Client) {
  const clientRecord = database.lookup(client);
  return clientRecord.isActive();
}
```

### Les noms de fonction doivent préciser ce qu'ils font

**Mal:**

```ts
function addToDate(date: Date, month: number): Date {
  // ...
}

const date = new Date();

// It's hard to tell from the function name what is added
addToDate(date, 1);
```

**Bien:**

```ts
function addMonthToDate(date: Date, month: number): Date {
  // ...
}

const date = new Date();
addMonthToDate(date, 1);
```

### Les fonctions ne doivent avoir qu'un niveau d'abstraction

Quand vous avez plus d'un niveau d'abstraction, votre fonction en fait généralement
trop. La division des fonctions conduit à une réutilisabilité et à des tests plus faciles.

**Mal:**

```ts
function parseCode(code: string) {
  const REGEXES = [ /* ... */ ];
  const statements = code.split(' ');
  const tokens = [];

  REGEXES.forEach((regex) => {
    statements.forEach((statement) => {
      // ...
    });
  });

  const ast = [];
  tokens.forEach((token) => {
    // lex...
  });

  ast.forEach((node) => {
    // parse...
  });
}
```

**Bien:**

```ts
const REGEXES = [ /* ... */ ];

function parseCode(code: string) {
  const tokens = tokenize(code);
  const syntaxTree = parse(tokens);

  syntaxTree.forEach((node) => {
    // parse...
  });
}

function tokenize(code: string): Token[] {
  const statements = code.split(' ');
  const tokens: Token[] = [];

  REGEXES.forEach((regex) => {
    statements.forEach((statement) => {
      tokens.push( /* ... */ );
    });
  });

  return tokens;
}

function parse(tokens: Token[]): SyntaxTree {
  const syntaxTree: SyntaxTree[] = [];
  tokens.forEach((token) => {
    syntaxTree.push( /* ... */ );
  });

  return syntaxTree;
}
```

### Eliminer la partie dupliquée du code

Faites de votre mieux pour éviter la duplication de certaines parties du votre code.
La duplication du code est mauvaise car cela signifie qu'il y a plus d'un endroit
pour modifier quelque chose si vous devez changer une logique.

Imaginez que vous dirigiez un restaurant et que vous gardiez une trace de votre
inventaire: toutes vos tomates, oignons, ail, épices, etc. Si vous avez plusieurs
listes sur lesquelles vous gardez cela, alors toutes doivent être mises à jour
lorsque vous servez un plat avec des tomates. Si vous n'avez qu'une seule liste,
il n'y a qu'un seul endroit à mettre à jour!

Souvent, vous avez du code en double parce que vous avez deux ou plusieurs choses
légèrement différentes, qui partagent beaucoup en commun, mais leurs différences
vous obligent à avoir deux ou plusieurs fonctions distinctes qui font à peu près
les mêmes choses. Supprimer le code en double signifie créer une abstraction qui
peut gérer cet ensemble de choses différentes avec une seule fonction/module/classe.

Avoir la correcte abstraction est essentiel, c'est pourquoi vous devez suivre les
principes [SOLID](#solid). Les mauvaises abstractions peuvent être pires que le
code en double, alors faites attention! Cela dit, si vous pouvez faire une bonne
abstraction, faites-le! Ne vous répétez pas, sinon vous vous retrouverez à actualiser
plusieurs endroits chaque fois que vous voulez changer une chose.

**Mal:**

```ts
function showDeveloperList(developers: Developer[]) {
  developers.forEach((developer) => {
    const expectedSalary = developer.calculateExpectedSalary();
    const experience = developer.getExperience();
    const githubLink = developer.getGithubLink();

    const data = {
      expectedSalary,
      experience,
      githubLink
    };

    render(data);
  });
}

function showManagerList(managers: Manager[]) {
  managers.forEach((manager) => {
    const expectedSalary = manager.calculateExpectedSalary();
    const experience = manager.getExperience();
    const portfolio = manager.getMBAProjects();

    const data = {
      expectedSalary,
      experience,
      portfolio
    };

    render(data);
  });
}
```

**Bien:**

```ts
class Developer {
  // ...
  getExtraDetails() {
    return {
      githubLink: this.githubLink,
    }
  }
}

class Manager {
  // ...
  getExtraDetails() {
    return {
      portfolio: this.portfolio,
    }
  }
}

function showEmployeeList(employee: Developer | Manager) {
  employee.forEach((employee) => {
    const expectedSalary = employee.calculateExpectedSalary();
    const experience = employee.getExperience();
    const extra = employee.getExtraDetails();

    const data = {
      expectedSalary,
      experience,
      extra,
    };

    render(data);
  });
}
```

Vous devez être critique sur la duplication de code. Parfois, il y a un compromis
entre le code dupliqué et une complexité accrue en introduisant une abstraction
inutile. Lorsque deux implémentations de deux modules différents se ressemblent
mais vivent dans des domaines différents, la duplication peut être acceptable et
préférable à l'extraction du code commun. Le code commun extrait dans ce cas
introduit une dépendance indirecte entre les deux modules.

### Définir les objets par défaut avec “Object.assign” ou déstructuration

**Mal:**

```ts
type MenuConfig = { title?: string, body?: string, buttonText?: string, cancellable?: boolean };

function createMenu(config: MenuConfig) {
  config.title = config.title || 'Foo';
  config.body = config.body || 'Bar';
  config.buttonText = config.buttonText || 'Baz';
  config.cancellable = config.cancellable !== undefined ? config.cancellable : true;

  // ...
}

createMenu({ body: 'Bar' });
```

**Bien:**

```ts
type MenuConfig = { title?: string, body?: string, buttonText?: string, cancellable?: boolean };

function createMenu(config: MenuConfig) {
  const menuConfig = Object.assign({
    title: 'Foo',
    body: 'Bar',
    buttonText: 'Baz',
    cancellable: true
  }, config);

  // ...
}

createMenu({ body: 'Bar' });
```

Également, vous pouvez utiliser la déstructuration avec des valeurs par défaut:

```ts
type MenuConfig = { title?: string, body?: string, buttonText?: string, cancellable?: boolean };

function createMenu({ title = 'Foo', body = 'Bar', buttonText = 'Baz', cancellable = true }: MenuConfig) {
  // ...
}

createMenu({ body: 'Bar' });
```

Pour éviter tout effet secondaire et tout comportement inattendu en transmettant
explicitement la valeur `undefined` ou `null`, vous pouvez dire au compilateur de
TypeScript de ne pas l'autoriser.
Consultez [`--strictNullChecks`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html#--strictnullchecks) l'option dans TypeScript.

### Ne pas utiliser pas des indicateurs comme paramètres de fonction

Les indicateurs indiquent à votre utilisateur que cette fonction fait plus d'une chose.
Les fonctions devraient faire une chose. Divisez vos fonctions si elles suivent des
chemins de code différents basés sur un booléen.

**Mal:**

```ts
function createFile(name: string, temp: boolean) {
  if (temp) {
    fs.create(`./temp/${name}`);
  } else {
    fs.create(name);
  }
}
```

**Bien:**

```ts
function createTempFile(name: string) {
  createFile(`./temp/${name}`);
}

function createFile(name: string) {
  fs.create(name);
}
```

### Éviter les effets secondaires (partie 1)

Une fonction produit un effet secondaire si elle fait autre chose que de prendre
une valeur et de renvoyer une ou plusieurs autres valeurs. Un effet secondaire
pourrait être d'écrire dans un fichier, de modifier une variable globale ou de
transférer accidentellement tout votre argent à un étranger.

Maintenant, vous devez avoir des effets secondaires dans un programme à l'occasion.
Comme dans l'exemple précédent, vous devrez peut-être écrire dans un fichier. Ce
que vous voulez faire, c'est de centraliser où vous faites cela. Ne pas avoir
plusieurs fonctions et classes qui écrivent dans un fichier particulier. Avoir
un service qui le fait. Seul et l'unique.

Le point principal est d'éviter les pièges courants comme le partage d'état entre
des objets sans aucune structure, l'utilisation de types de données mutables qui
peuvent être écrits par n'importe quoi, et ne pas centraliser où se produisent
vos effets secondaires. Si vous pouvez le faire, vous serez plus heureux que la
grande majorité des autres programmeurs.

**Mal:**

```ts
// Global variable referenced by following function.
let name = 'Robert C. Martin';

function toBase64() {
  name = btoa(name);
}

toBase64();
// If we had another function that used this name, now it'd be a Base64 value

console.log(name); // expected to print 'Robert C. Martin' but instead 'Um9iZXJ0IEMuIE1hcnRpbg=='
```

**Bien:**

```ts
const name = 'Robert C. Martin';

function toBase64(text: string): string {
  return btoa(text);
}

const encodedName = toBase64(name);
console.log(name);
```

### Éviter les effets secondaires (partie 2)

En JavaScript, les primitives sont passées par valeur et les objets et tableaux
sont passés par référence. Dans le cas d'objets et de tableaux, si votre fonction
modifie un tableau de panier d'achat, par exemple, en ajoutant un article à acheter,
alors toute autre fonction qui utilise ce tableau `cart` sera affectée par cet
ajout. C'est peut-être bien, mais ça peut aussi être mauvais. Imaginons une mauvaise situation:

L'utilisateur clique sur le bouton “Achat”, qui appelle une fonction `purchase`
qui génère une demande réseau et envoie le tableau `cart` au serveur. En raison
d'une mauvaise connexion réseau, la fonction d'achat doit continuer à réessayer
la demande. Maintenant, que se passe-t-il si, dans l'intervalle, l'utilisateur
clique accidentellement sur le bouton `addItemToCart` sur un article qu'il ne
souhaite pas avant le début de la demande réseau? Si cela se produit et que la
demande de réseau commence, alors cette fonction d'achat enverra l'article ajouté
accidentellement car il a une référence à un tableau de panier d'achat que la
fonction `addItemToCart` a modifié en ajoutant un article indésirable.

Une excellente solution serait que `addItemToCart` clone toujours le`cart`, le
modifie et renvoie le clone. Cela garantit qu'aucune autre fonction conservant
une référence du panier ne sera affectée par des modifications.

Deux avertissements à mentionner à cette approche:

1. Il peut y avoir des cas où vous souhaitez réellement modifier l'objet passée
comme paramètre, mais lorsque vous adoptez cette pratique de programmation, vous
constaterez que ces cas sont assez rares. La plupart des choses peuvent être
refactorisées pour n'avoir aucun effet secondaire!
(voir [fonction pure](https://en.wikipedia.org/wiki/Pure_function))

2. Le clonage de gros objets peut être très coûteux en termes de performances.
Heureusement, ce n'est pas un gros problème dans la pratique, car il existe d'excellentes
bibliothèques qui permettent à ce type d'approche de programmation d'être rapide
et moins gourmande en mémoire que pour le clonage manuel d'objets et de tableaux.

**Mal:**

```ts
function addItemToCart(cart: CartItem[], item: Item): void {
  cart.push({ item, date: Date.now() });
};
```

**Bien:**

```ts
function addItemToCart(cart: CartItem[], item: Item): CartItem[] {
  return [...cart, { item, date: Date.now() }];
};
```

### Ne pas écrire dans les fonctions globales

Polluer les fonctions globales est une mauvaise pratique en JavaScript car vous
pourriez entrer en conflit avec une autre bibliothèque et l'utilisateur de votre
API ne serait pas plus sage jusqu'à ce qu'il obtienne une exception en production.
Réfléchissons à un exemple: et si vous vouliez étendre la méthode native Array
de JavaScript pour avoir une méthode `diff` qui pourrait montrer la différence
entre deux tableaux? Vous pouvez écrire votre nouvelle fonction dans le
`Array.prototype`, mais elle pourrait entrer en conflit avec une autre bibliothèque
qui a essayé de faire la même chose. Et si cette autre bibliothèque utilisait
simplement `diff` pour trouver la différence entre le premier et le dernier élément
d'un tableau? C'est pourquoi il serait beaucoup mieux d'utiliser simplement des
classes et d'étendre simplement le global `Array`.

**Mal:**

```ts
declare global {
  interface Array<T> {
    diff(other: T[]): Array<T>;
  }
}

if (!Array.prototype.diff) {
  Array.prototype.diff = function <T>(other: T[]): T[] {
    const hash = new Set(other);
    return this.filter(elem => !hash.has(elem));
  };
}
```

**Bien:**

```ts
class MyArray<T> extends Array<T> {
  diff(other: T[]): T[] {
    const hash = new Set(other);
    return this.filter(elem => !hash.has(elem));
  };
}
```


### Privilégier la programmation fonctionnelle à la programmation impérative

Privilégiez ce style de programmation quand vous le pouvez.

**Mal:**

```ts
const contributions = [
  {
    name: 'Uncle Bobby',
    linesOfCode: 500
  }, {
    name: 'Suzie Q',
    linesOfCode: 1500
  }, {
    name: 'Jimmy Gosling',
    linesOfCode: 150
  }, {
    name: 'Gracie Hopper',
    linesOfCode: 1000
  }
];

let totalOutput = 0;

for (let i = 0; i < contributions.length; i++) {
  totalOutput += contributions[i].linesOfCode;
}
```

**Bien:**

```ts
const contributions = [
  {
    name: 'Uncle Bobby',
    linesOfCode: 500
  }, {
    name: 'Suzie Q',
    linesOfCode: 1500
  }, {
    name: 'Jimmy Gosling',
    linesOfCode: 150
  }, {
    name: 'Gracie Hopper',
    linesOfCode: 1000
  }
];

const totalOutput = contributions
  .reduce((totalLines, output) => totalLines + output.linesOfCode, 0);
```



### Encapsuler les conditions

**Mal:**

```ts
if (subscription.isTrial || account.balance > 0) {
  // ...
}
```

**Bien:**

```ts
function canActivateService(subscription: Subscription, account: Account) {
  return subscription.isTrial || account.balance > 0;
}

if (canActivateService(subscription, account)) {
  // ...
}
```

### Éviter les conditions négatives

**Mal:**

```ts
function isEmailNotUsed(email: string): boolean {
  // ...
}

if (isEmailNotUsed(email)) {
  // ...
}
```

**Bien:**

```ts
function isEmailUsed(email: string): boolean {
  // ...
}

if (!isEmailUsed(node)) {
  // ...
}
```

### Éviter les conditions

Cela semble être une tâche impossible. En entendant cela pour la première fois,
la plupart des gens disent: “Comment suis-je censé faire quoi que ce soit sans
une déclaration `if`?" La réponse est que vous pouvez utiliser le polymorphisme
pour réaliser la même tâche dans de nombreux cas. La deuxième question est
généralement, "c'est bien, mais pourquoi voudrais-je faire ça?" La réponse est
un concept de code propre précédent que nous avons appris: une fonction ne devrait
faire qu'une seule chose. Lorsque vous avez des classes et des fonctions qui ont
des instructions `if`, vous dites à votre utilisateur que votre fonction fait
plus d'une chose. N'oubliez pas, faites juste une chose.

**Mal:**

```ts
class Airplane {
  private type: string;
  // ...

  getCruisingAltitude() {
    switch (this.type) {
      case '777':
        return this.getMaxAltitude() - this.getPassengerCount();
      case 'Air Force One':
        return this.getMaxAltitude();
      case 'Cessna':
        return this.getMaxAltitude() - this.getFuelExpenditure();
      default:
        throw new Error('Unknown airplane type.');
    }
  }

  private getMaxAltitude(): number {
    // ...
  }
}
```

**Bien:**

```ts
abstract class Airplane {
  protected getMaxAltitude(): number {
    // shared logic with subclasses ...
  }

  // ...
}

class Boeing777 extends Airplane {
  // ...
  getCruisingAltitude() {
    return this.getMaxAltitude() - this.getPassengerCount();
  }
}

class AirForceOne extends Airplane {
  // ...
  getCruisingAltitude() {
    return this.getMaxAltitude();
  }
}

class Cessna extends Airplane {
  // ...
  getCruisingAltitude() {
    return this.getMaxAltitude() - this.getFuelExpenditure();
  }
}
```

### Éviter la vérification de type de donnée

TypeScript est un superset syntaxique strict de JavaScript et ajoute une vérification
de type statique facultative au langage.
Préférez toujours spécifier des types de variables, des paramètres et des valeurs
de retour pour exploiter toute la puissance des fonctionnalités de TypeScript.
Cela facilite la refactorisation.

**Mal:**

```ts
function travelToTexas(vehicle: Bicycle | Car) {
  if (vehicle instanceof Bicycle) {
    vehicle.pedal(currentLocation, new Location('texas'));
  } else if (vehicle instanceof Car) {
    vehicle.drive(currentLocation, new Location('texas'));
  }
}
```

**Bien:**

```ts
type Vehicle = Bicycle | Car;

function travelToTexas(vehicle: Vehicle) {
  vehicle.move(currentLocation, new Location('texas'));
}
```

### Ne pas trop optimiser

Les navigateurs modernes font beaucoup d'optimisation sous le capot lors de l'exécution.
Souvent, si vous optimisez, vous perdez simplement votre temps. Il existe de bonnes
[ressources](https://github.com/petkaantonov/bluebird/wiki/Optimization-killers)
pour voir où l'optimisation fait défaut. Ciblez ceux en attendant, jusqu'à ce
qu'ils soient corrigés s'ils le peuvent.

**Mal:**

```ts
// On old browsers, each iteration with uncached `list.length` would be costly
// because of `list.length` recomputation. In modern browsers, this is optimized.
for (let i = 0, len = list.length; i < len; i++) {
  // ...
}
```

**Bien:**

```ts
for (let i = 0; i < list.length; i++) {
  // ...
}
```

### Éliminer tout code qui ne s’utilise pas

Le code qui ne s’utilise pas est tout aussi mauvais que le code en double. Il n'y
a aucune raison de le conserver dans votre base de code. S'il n'est pas appelé,
débarrassez-vous-en! Il sera toujours sauvegardé en sécurité dans votre historique
de version si vous en avez toujours besoin.

**Mal:**

```ts
function oldRequestModule(url: string) {
  // ...
}

function requestModule(url: string) {
  // ...
}

const req = requestModule;
inventoryTracker('apples', req, 'www.inventory-awesome.io');
```

**Bien:**

```ts
function requestModule(url: string) {
  // ...
}

const req = requestModule;
inventoryTracker('apples', req, 'www.inventory-awesome.io');
```

### Utiliser des itérateurs et des générateurs

Utilisez des générateurs et des itérables lorsque vous travaillez avec des
collections de données utilisées comme un flux. Il y a quelques bonnes raisons:

- dissocie l'appelé de la mise en œuvre du générateur dans le sens où l'appelé
décide du nombre éléments à accéder
- exécution paresseuse, les éléments sont diffusés à la demande
- prise en charge intégrée pour l'itération d'éléments à l'aide de la syntaxe `for-of`
- les itérables permettent d'implémenter des modèles d'itérateurs optimisés

**Mal:**

```ts
function fibonacci(n: number): number[] {
  if (n === 1) return [0];
  if (n === 2) return [0, 1];

  const items: number[] = [0, 1];
  while (items.length < n) {
    items.push(items[items.length - 2] + items[items.length - 1]);
  }

  return items;
}

function print(n: number) {
  fibonacci(n).forEach(fib => console.log(fib));
}

// Print first 10 Fibonacci numbers.
print(10);
```

**Bien:**

```ts
// Generates an infinite stream of Fibonacci numbers.
// The generator doesn't keep the array of all numbers.
function* fibonacci(): IterableIterator<number> {
  let [a, b] = [0, 1];

  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

function print(n: number) {
  let i = 0;
  for (const fib of fibonacci()) {
    if (i++ === n) break;
    console.log(fib);
  }
}

// Print first 10 Fibonacci numbers.
print(10);
```

Il existe des bibliothèques qui permettent de travailler avec les itérables de
la même manière qu'avec les tableaux natifs, en des méthodes de chaînage comme
`map`, `slice`, `forEach` etc. Voir [itiriri](https://www.npmjs.com/package/itiriri)
pour un exemple de manipulation avancée avec les itérables
(ou [itiriri-async](https://www.npmjs.com/package/itiriri-async) pour la manipulation
des itérables asynchrones).

```ts
import itiriri from 'itiriri';

function* fibonacci(): IterableIterator<number> {
  let [a, b] = [0, 1];

  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

itiriri(fibonacci())
  .take(10)
  .forEach(fib => console.log(fib));
```
