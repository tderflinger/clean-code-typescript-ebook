## Objets et Structures de Données

### Utiliser des "getters" et "setters"

TypeScript supporte la syntaxe getter/setter.

L'utilisation de getters et de setters pour accéder aux données d'objets qui
englobent le comportement pourrait être meilleure que la simple recherche d'un
attribut sur un objet. "Pourquoi?" vous vous demander. Eh bien, voici une liste de raisons:

- Lorsque vous voulez faire plus que d'obtenir l’attribut d'un objet, vous n'avez
pas besoin de rechercher et de modifier chaque accesseur dans votre base de code.
- Valide de façon plus simple en utilisant le mot-clé `set`.
- Encapsule la représentation interne.
- Facile à ajouter un log des activités et la gestion des erreurs.
- Vous pouvez paresseusement charger les attributs de votre objet, disons l'obtenir
à partir d'un serveur.

**Mal:**

```ts
type BankAccount = {
  balance: number;
  // ...
}

const value = 100;
const account: BankAccount = {
  balance: 0,
  // ...
};

if (value < 0) {
  throw new Error('Cannot set negative balance.');
}

account.balance = value;
```

**Bien:**

```ts
class BankAccount {
  private accountBalance: number = 0;

  get balance(): number {
    return this.accountBalance;
  }

  set balance(value: number) {
    if (value < 0) {
      throw new Error('Cannot set negative balance.');
    }

    this.accountBalance = value;
  }

  // ...
}

// Now `BankAccount` encapsulates the validation logic.
// If one day the specifications change, and we need extra validation rule,
// we would have to alter only the `setter` implementation,
// leaving all dependent code unchanged.
const account = new BankAccount();
account.balance = 100;
```

### Faire en sorte que les objets aient des membres privés/protégés

TypeScript prend en charge les accesseurs `public` *(par défaut)*, `protected`
et `private` sur les membres de la classe.

**Mal:**

```ts
class Circle {
  radius: number;

  constructor(radius: number) {
    this.radius = radius;
  }

  perimeter() {
    return 2 * Math.PI * this.radius;
  }

  surface() {
    return Math.PI * this.radius * this.radius;
  }
}
```

**Bien:**

```ts
class Circle {
  constructor(private readonly radius: number) {
  }

  perimeter() {
    return 2 * Math.PI * this.radius;
  }

  surface() {
    return Math.PI * this.radius * this.radius;
  }
}
```

### Préférer l'immuabilité

Le système de types de TypeScript vous permet de marquer des attributs individuels
sur une interface/classe comme *readonly*. Cela vous permet de travailler de manière
fonctionnelle (une mutation inattendue est mauvaise). Pour les scénarios plus avancés,
il existe un type intégré `Readonly` qui prend un type`T` et marque toutes ses
attributs comme étant “readonly” à l'aide de types mappés
(voir [mapped types ou "types mappés"](https://www.typescriptlang.org/docs/handbook/advanced-types.html#mapped-types)).

**Mal:**

```ts
interface Config {
  host: string;
  port: string;
  db: string;
}
```

**Bien:**

```ts
interface Config {
  readonly host: string;
  readonly port: string;
  readonly db: string;
}
```

Dans le cas des tableaux, vous pouvez créer un tableau de *readonly* en utilisant
`ReadonlyArray<T>`.
N'autorisez pas les modifications telles que `push()` et `fill()`, mais plutôt
utilisez des fonctionnalités telles que `concat()` et `slice()` vu que celles-ci
ne modifient pas la valeur.

**Mal:**

```ts
const array: number[] = [ 1, 3, 5 ];
array = []; // error
array.push(100); // array will updated
```

**Bien:**

```ts
const array: ReadonlyArray<number> = [ 1, 3, 5 ];
array = []; // error
array.push(100); // error
```

Déclarer des paramètres de type "read-only" dans [TypeScript 3.4 est un peu plus facile](https://github.com/microsoft/TypeScript/wiki/What's-new-in-TypeScript#improvements-for-readonlyarray-and-readonly-tuples).

```ts
function hoge(args: readonly string[]) {
  args.push(1); // error
}
```

Préférer les [assertions "const"](https://github.com/microsoft/TypeScript/wiki/What's-new-in-TypeScript#const-assertions)
pour des valeurs littérales.

**Mal:**

```ts
const config = {
  hello: 'world'
};
config.hello = 'world'; // value is changed

const array  = [ 1, 3, 5 ];
array[0] = 10; // value is changed

// writable objects is returned
function readonlyData(value: number) {
  return { value };
}

const result = readonlyData(100);
result.value = 200; // value is changed
```

**Bien:**

```ts
// read-only object
const config = {
  hello: 'world'
} as const;
config.hello = 'world'; // error

// read-only array
const array  = [ 1, 3, 5 ] as const;
array[0] = 10; // error

// You can return read-only objects
function readonlyData(value: number) {
  return { value } as const;
}

const result = readonlyData(100);
result.value = 200; // error
```
