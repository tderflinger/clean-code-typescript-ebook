## SOLID

### Principe de Responsabilité Unique ou "Single Responsibility Principle (SRP)"

Comme indiqué dans Clean Code, "Il ne devrait jamais y avoir plus d'une raison
pour qu'une classe change". Il est tentant d'emballer une classe avec beaucoup
de fonctionnalités, comme lorsque vous ne pouvez emporter qu'une seule valise
pendant votre vol. Le problème avec cela est que votre classe ne sera pas
conceptuellement cohérente et cela lui donnera de nombreuses raisons de changer.
Il est important de minimiser le nombre de fois que vous devez changer de classe.
C'est important car si trop de fonctionnalités sont dans une classe et que vous
en modifiez une partie, il peut être difficile de comprendre comment cela affectera
les autres modules dépendants de votre base de code.

**Mal:**

```ts
class UserSettings {
  constructor(private readonly user: User) {
  }

  changeSettings(settings: UserSettings) {
    if (this.verifyCredentials()) {
      // ...
    }
  }

  verifyCredentials() {
    // ...
  }
}
```

**Bien:**

```ts
class UserAuth {
  constructor(private readonly user: User) {
  }

  verifyCredentials() {
    // ...
  }
}


class UserSettings {
  private readonly auth: UserAuth;

  constructor(private readonly user: User) {
    this.auth = new UserAuth(user);
  }

  changeSettings(settings: UserSettings) {
    if (this.auth.verifyCredentials()) {
      // ...
    }
  }
}
```

### Principe ouvert/fermé ou "Open/Closed Principle (OCP)"

Comme l'a déclaré Bertrand Meyer, "les entités logicielles (classes, modules,
fonctions, etc.) devraient être ouvertes pour extension, mais fermées pour modification."
Mais qu'est-ce que cela signifie? Ce principe stipule essentiellement que vous
devez autoriser les utilisateurs à ajouter de nouvelles fonctionnalités sans modifier le code existant.

**Mal:**

```ts
class AjaxAdapter extends Adapter {
  constructor() {
    super();
  }

  // ...
}

class NodeAdapter extends Adapter {
  constructor() {
    super();
  }

  // ...
}

class HttpRequester {
  constructor(private readonly adapter: Adapter) {
  }

  async fetch<T>(url: string): Promise<T> {
    if (this.adapter instanceof AjaxAdapter) {
      const response = await makeAjaxCall<T>(url);
      // transform response and return
    } else if (this.adapter instanceof NodeAdapter) {
      const response = await makeHttpCall<T>(url);
      // transform response and return
    }
  }
}

function makeAjaxCall<T>(url: string): Promise<T> {
  // request and return promise
}

function makeHttpCall<T>(url: string): Promise<T> {
  // request and return promise
}
```

**Bien:**

```ts
abstract class Adapter {
  abstract async request<T>(url: string): Promise<T>;

  // code shared to subclasses ...
}

class AjaxAdapter extends Adapter {
  constructor() {
    super();
  }

  async request<T>(url: string): Promise<T>{
    // request and return promise
  }

  // ...
}

class NodeAdapter extends Adapter {
  constructor() {
    super();
  }

  async request<T>(url: string): Promise<T>{
    // request and return promise
  }

  // ...
}

class HttpRequester {
  constructor(private readonly adapter: Adapter) {
  }

  async fetch<T>(url: string): Promise<T> {
    const response = await this.adapter.request<T>(url);
    // transform response and return
  }
}
```

### Principe de substitution de Liskov ou "Liskov Substitution Principle (LSP)"

C'est un terme effrayant pour un concept très simple. Il est formellement défini
comme "Si S est un sous-type de T, alors les objets de type T peuvent être remplacés
par des objets de type S (c'est-à-dire que les objets de type S peuvent remplacer
des objets de type T) sans altérer aucune des attributs souhaitables de ce
programme (correction, tâche effectuée, etc.)." C'est une définition encore plus effrayante.

La meilleure explication est que si vous avez une classe parent et une classe enfant,
la classe parent et la classe enfant peuvent être utilisées de manière interchangeable
sans obtenir de résultats incorrects. Cela peut encore prêter à confusion, alors
jetons un coup d'œil à l'exemple classique de Square-Rectangle. Mathématiquement,
un carré est un rectangle, mais si vous le modélisez en utilisant la relation "is-a"
via l'héritage, vous rencontrez rapidement des problèmes.

**Mal:**

```ts
class Rectangle {
  constructor(
    protected width: number = 0,
    protected height: number = 0) {

  }

  setColor(color: string): this {
    // ...
  }

  render(area: number) {
    // ...
  }

  setWidth(width: number): this {
    this.width = width;
    return this;
  }

  setHeight(height: number): this {
    this.height = height;
    return this;
  }

  getArea(): number {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  setWidth(width: number): this {
    this.width = width;
    this.height = width;
    return this;
  }

  setHeight(height: number): this {
    this.width = height;
    this.height = height;
    return this;
  }
}

function renderLargeRectangles(rectangles: Rectangle[]) {
  rectangles.forEach((rectangle) => {
    const area = rectangle
      .setWidth(4)
      .setHeight(5)
      .getArea(); // BAD: Returns 25 for Square. Should be 20.
    rectangle.render(area);
  });
}

const rectangles = [new Rectangle(), new Rectangle(), new Square()];
renderLargeRectangles(rectangles);
```

**Bien:**

```ts
abstract class Shape {
  setColor(color: string): this {
    // ...
  }

  render(area: number) {
    // ...
  }

  abstract getArea(): number;
}

class Rectangle extends Shape {
  constructor(
    private readonly width = 0,
    private readonly height = 0) {
    super();
  }

  getArea(): number {
    return this.width * this.height;
  }
}

class Square extends Shape {
  constructor(private readonly length: number) {
    super();
  }

  getArea(): number {
    return this.length * this.length;
  }
}

function renderLargeShapes(shapes: Shape[]) {
  shapes.forEach((shape) => {
    const area = shape.getArea();
    shape.render(area);
  });
}

const shapes = [new Rectangle(4, 5), new Rectangle(4, 5), new Square(5)];
renderLargeShapes(shapes);
```

### Principe de ségrégation d'interface ou "Interface Segregation Principle (ISP)"

L'ISP déclare que "les clients ne devraient pas être obligés de dépendre d'interfaces
qu'ils n'utilisent pas.". Ce principe est très lié au principe de responsabilité unique.
Ce que cela signifie vraiment, c'est que vous devez toujours concevoir vos abstractions
de manière à ce que les clients qui utilisent les méthodes exposées n'obtiennent
pas tout le gâteau à la place. Cela implique également d'imposer aux clients la
charge de mettre en œuvre des méthodes dont ils n'ont pas réellement besoin.

**Mal:**

```ts
interface SmartPrinter {
  print();
  fax();
  scan();
}

class AllInOnePrinter implements SmartPrinter {
  print() {
    // ...
  }

  fax() {
    // ...
  }

  scan() {
    // ...
  }
}

class EconomicPrinter implements SmartPrinter {
  print() {
    // ...
  }

  fax() {
    throw new Error('Fax not supported.');
  }

  scan() {
    throw new Error('Scan not supported.');
  }
}
```

**Bien:**

```ts
interface Printer {
  print();
}

interface Fax {
  fax();
}

interface Scanner {
  scan();
}

class AllInOnePrinter implements Printer, Fax, Scanner {
  print() {
    // ...
  }

  fax() {
    // ...
  }

  scan() {
    // ...
  }
}

class EconomicPrinter implements Printer {
  print() {
    // ...
  }
}
```

### Principe d'inversion de dépendance ou "Dependency Inversion Principle (DIP)"

Ce principe énonce deux choses essentielles:

1. Les modules de haut niveau ne doivent pas dépendre de modules de bas niveau.
Les deux devraient dépendre d'abstractions.
2. Les abstractions ne devraient pas dépendre des détails. Les détails doivent
dépendre des abstractions.

Cela peut être difficile à comprendre au début, mais si vous avez travaillé avec
Angular, vous avez vu une implémentation de ce principe sous la forme d'une
injection de dépendance (DI). Bien qu'il ne s'agisse pas de concepts identiques,
DIP empêche les modules de haut niveau de connaître les détails de ses modules de
bas niveau et de les configurer. Il peut y parvenir grâce à DI. Un énorme avantage
de ceci est qu'il réduit le couplage entre les modules. Le couplage est un très
mauvais schéma de développement car il rend votre code difficile à refactoriser.

Le DIP est généralement obtenu en utilisant un conteneur d'inversion de contrôle
(IoC). Un exemple de conteneur IoC puissant pour TypeScript est
[InversifyJs](https://www.npmjs.com/package/inversify).

**Mal:**

```ts
import { readFile as readFileCb } from 'fs';
import { promisify } from 'util';

const readFile = promisify(readFileCb);

type ReportData = {
  // ..
}

class XmlFormatter {
  parse<T>(content: string): T {
    // Converts an XML string to an object T
  }
}

class ReportReader {

  // BAD: We have created a dependency on a specific request implementation.
  // We should just have ReportReader depend on a parse method: `parse`
  private readonly formatter = new XmlFormatter();

  async read(path: string): Promise<ReportData> {
    const text = await readFile(path, 'UTF8');
    return this.formatter.parse<ReportData>(text);
  }
}

// ...
const reader = new ReportReader();
await report = await reader.read('report.xml');
```

**Bien:**

```ts
import { readFile as readFileCb } from 'fs';
import { promisify } from 'util';

const readFile = promisify(readFileCb);

type ReportData = {
  // ..
}

interface Formatter {
  parse<T>(content: string): T;
}

class XmlFormatter implements Formatter {
  parse<T>(content: string): T {
    // Converts an XML string to an object T
  }
}


class JsonFormatter implements Formatter {
  parse<T>(content: string): T {
    // Converts a JSON string to an object T
  }
}

class ReportReader {
  constructor(private readonly formatter: Formatter) {
  }

  async read(path: string): Promise<ReportData> {
    const text = await readFile(path, 'UTF8');
    return this.formatter.parse<ReportData>(text);
  }
}

// ...
const reader = new ReportReader(new XmlFormatter());
await report = await reader.read('report.xml');

// or if we had to read a json report
const reader = new ReportReader(new JsonFormatter());
await report = await reader.read('report.json');
```
