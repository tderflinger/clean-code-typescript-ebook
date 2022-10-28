## Classes

### Les classes doivent être petites

La taille de la classe est mesurée par sa responsabilité. Suivant le
*Principe de Responsabilité Unique*, une classe doit être petite.

**Mal:**

```ts
class Dashboard {
  getLanguage(): string { /* ... */ }
  setLanguage(language: string): void { /* ... */ }
  showProgress(): void { /* ... */ }
  hideProgress(): void { /* ... */ }
  isDirty(): boolean { /* ... */ }
  disable(): void { /* ... */ }
  enable(): void { /* ... */ }
  addSubscription(subscription: Subscription): void { /* ... */ }
  removeSubscription(subscription: Subscription): void { /* ... */ }
  addUser(user: User): void { /* ... */ }
  removeUser(user: User): void { /* ... */ }
  goToHomePage(): void { /* ... */ }
  updateProfile(details: UserDetails): void { /* ... */ }
  getVersion(): string { /* ... */ }
  // ...
}

```

**Bien:**

```ts
class Dashboard {
  disable(): void { /* ... */ }
  enable(): void { /* ... */ }
  getVersion(): string { /* ... */ }
}

// split the responsibilities by moving the remaining methods to other classes
// ...
```

### Haute cohésion et faible couplage

La cohésion définit le degré de relation entre les membres de la classe. Idéalement,
tous les champs d'une classe doivent être utilisés par chaque méthode. On dit alors
que la classe est au maximum cohérente. En pratique, cela n'est cependant pas toujours
possible, ni même conseillé. Vous devez cependant préférer une cohésion élevée.

Le couplage fait référence à la façon dont les parents sont liés ou dépendants
entre eux. Les classes sont dites à faible couplage si les changements dans l'un
d'entre eux n'affectent pas l'autre.

Une bonne conception logicielle a une **haute cohésion** et un **faible couplage**.

**Mal:**

```ts
class UserManager {
  // Bad: each private variable is used by one or another group of methods.
  // It makes clear evidence that the class is holding more than a single responsibility.
  // If I need only to create the service to get the transactions for a user,
  // I'm still forced to pass and instance of `emailSender`.
  constructor(
    private readonly db: Database,
    private readonly emailSender: EmailSender) {
  }

  async getUser(id: number): Promise<User> {
    return await db.users.findOne({ id });
  }

  async getTransactions(userId: number): Promise<Transaction[]> {
    return await db.transactions.find({ userId });
  }

  async sendGreeting(): Promise<void> {
    await emailSender.send('Welcome!');
  }

  async sendNotification(text: string): Promise<void> {
    await emailSender.send(text);
  }

  async sendNewsletter(): Promise<void> {
    // ...
  }
}
```

**Bien:**

```ts
class UserService {
  constructor(private readonly db: Database) {
  }

  async getUser(id: number): Promise<User> {
    return await this.db.users.findOne({ id });
  }

  async getTransactions(userId: number): Promise<Transaction[]> {
    return await this.db.transactions.find({ userId });
  }
}

class UserNotifier {
  constructor(private readonly emailSender: EmailSender) {
  }

  async sendGreeting(): Promise<void> {
    await this.emailSender.send('Welcome!');
  }

  async sendNotification(text: string): Promise<void> {
    await this.emailSender.send(text);
  }

  async sendNewsletter(): Promise<void> {
    // ...
  }
}
```

### Préférer la composition à l'héritage

Comme indiqué dans [Design Patterns](https://en.wikipedia.org/wiki/Design_Patterns)
du “Gang of Four”, *vous devriez préférer la composition à l'héritage* où vous le
pouvez. Il y a beaucoup de bonnes raisons d'utiliser l'héritage et beaucoup de
bonnes raisons d'utiliser la composition. Le point principal de cette maxime est
que si votre esprit va instinctivement à l'héritage, essayez de penser si la
composition pourrait mieux modéliser votre problème. Dans certains cas, c'est possible.

Vous vous demandez peut-être alors "quand dois-je utiliser l'héritage?" Cela
dépend de votre problème, mais c'est une liste décente où l'héritage a plus de
sens que la composition:

1. Votre héritage représente une relation "est-une" et non une relation "a-une"
(Humain-> Animal vs Utilisateur-> Détails de l'utilisateur).

2. Vous pouvez réutiliser le code des classes de base (les humains peuvent se
déplacer comme tous les animaux).

3. Vous souhaitez apporter des modifications globales aux classes dérivées en
modifiant une classe de base. (Modifiez la dépense calorique de tous les animaux
lorsqu'ils se déplacent).

**Mal:**

```ts
class Employee {
  constructor(
    private readonly name: string,
    private readonly email: string) {
  }

  // ...
}

// Bad because Employees "have" tax data. EmployeeTaxData is not a type of Employee
class EmployeeTaxData extends Employee {
  constructor(
    name: string,
    email: string,
    private readonly ssn: string,
    private readonly salary: number) {
    super(name, email);
  }

  // ...
}
```

**Bien:**

```ts
class Employee {
  private taxData: EmployeeTaxData;

  constructor(
    private readonly name: string,
    private readonly email: string) {
  }

  setTaxData(ssn: string, salary: number): Employee {
    this.taxData = new EmployeeTaxData(ssn, salary);
    return this;
  }

  // ...
}

class EmployeeTaxData {
  constructor(
    public readonly ssn: string,
    public readonly salary: number) {
  }

  // ...
}
```

### Utiliser le chaînage des méthodes

Ce modèle est très utile et couramment utilisé dans de nombreuses bibliothèques.
Il permet à votre code d'être expressif et moins verbeux. Pour cette raison,
utilisez le chaînage de méthodes et regardez à quel point votre code sera propre.

**Mal:**

```ts
class QueryBuilder {
  private collection: string;
  private pageNumber: number = 1;
  private itemsPerPage: number = 100;
  private orderByFields: string[] = [];

  from(collection: string): void {
    this.collection = collection;
  }

  page(number: number, itemsPerPage: number = 100): void {
    this.pageNumber = number;
    this.itemsPerPage = itemsPerPage;
  }

  orderBy(...fields: string[]): void {
    this.orderByFields = fields;
  }

  build(): Query {
    // ...
  }
}

// ...

const queryBuilder = new QueryBuilder();
queryBuilder.from('users');
queryBuilder.page(1, 100);
queryBuilder.orderBy('firstName', 'lastName');

const query = queryBuilder.build();
```

**Bien:**

```ts
class QueryBuilder {
  private collection: string;
  private pageNumber: number = 1;
  private itemsPerPage: number = 100;
  private orderByFields: string[] = [];

  from(collection: string): this {
    this.collection = collection;
    return this;
  }

  page(number: number, itemsPerPage: number = 100): this {
    this.pageNumber = number;
    this.itemsPerPage = itemsPerPage;
    return this;
  }

  orderBy(...fields: string[]): this {
    this.orderByFields = fields;
    return this;
  }

  build(): Query {
    // ...
  }
}

// ...

const query = new QueryBuilder()
  .from('users')
  .page(1, 100)
  .orderBy('firstName', 'lastName')
  .build();
```
