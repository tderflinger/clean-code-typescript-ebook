## Commentaires

L'utilisation de commentaires est une indication de non-expression sans eux.
Le code devrait être la seule source de vérité.

> Ne commentez pas le mauvais code - réécrivez-le.
> — *Brian W. Kernighan and P. J. Plaugher*

### Utiliser de préférence un code explicite plutôt que des commentaires

Les commentaires sont des excuses, pas une exigence. Un bon code *surtout* se documente.

**Mal:**

```ts
// Check if subscription is active.
if (subscription.endDate > Date.now) {  }
```

**Bien:**

```ts
const isSubscriptionActive = subscription.endDate > Date.now;
if (isSubscriptionActive) { /* ... */ }
```

### Ne pas laisser pas de code commenté dans votre base de code

Le contrôle de version existe pour une raison. Laissez l'ancien code dans votre historique.

**Mal:**

```ts
type User = {
  name: string;
  email: string;
  // age: number;
  // jobPosition: string;
}
```

**Bien:**

```ts
type User = {
  name: string;
  email: string;
}
```

### Ne pas avoir de commentaires dans l'archive de base

Rappelez-vous, utilisez le contrôle de version! Ce n'est pas nécessaire garder des
codes non utilisés ou commentés, et surtout de commentaires dans l'archive de base.
Utilisez le `git log` pour obtenir l'historique!

**Mal:**

```ts
/**
 * 2016-12-20: Removed monads, didn't understand them (RM)
 * 2016-10-01: Improved using special monads (JP)
 * 2016-02-03: Added type-checking (LI)
 * 2015-03-14: Implemented combine (JR)
 */
function combine(a: number, b: number): number {
  return a + b;
}
```

**Bien:**

```ts
function combine(a: number, b: number): number {
  return a + b;
}
```

### Éviter les marqueurs de position

Ils ajoutent généralement du "noise" au code. Laissez les fonctions et les noms
de variables ainsi que l'indentation et le formatage appropriés donner la structure
visuelle à votre code.
La plupart des IDE prennent en charge la fonction de pliage de code qui vous permet
de réduire/développer des blocs de code (voir  Visual Studio Code
[régions de pliage](https://code.visualstudio.com/updates/v1_17#_folding-regions)).

**Mal:**

```ts
////////////////////////////////////////////////////////////////////////////////
// Client class
////////////////////////////////////////////////////////////////////////////////
class Client {
  id: number;
  name: string;
  address: Address;
  contact: Contact;

  ////////////////////////////////////////////////////////////////////////////////
  // public methods
  ////////////////////////////////////////////////////////////////////////////////
  public describe(): string {
    // ...
  }

  ////////////////////////////////////////////////////////////////////////////////
  // private methods
  ////////////////////////////////////////////////////////////////////////////////
  private describeAddress(): string {
    // ...
  }

  private describeContact(): string {
    // ...
  }
};
```

**Bien:**

```ts
class Client {
  id: number;
  name: string;
  address: Address;
  contact: Contact;

  public describe(): string {
    // ...
  }

  private describeAddress(): string {
    // ...
  }

  private describeContact(): string {
    // ...
  }
};
```

### Les commentaires TODO

Lorsque vous vous rendez compte que vous devez laisser des notes dans le code pour
des améliorations ultérieures, faites-le en utilisant les commentaires `// TODO`.
La plupart des IDE ont un support spécial pour ce genre de commentaires afin que
vous puissiez parcourir rapidement la liste complète des "todos".

Gardez cependant à l'esprit qu'un commentaire *TODO* n'est pas une excuse pour un mauvais code.

**Mal:**

```ts
function getActiveSubscriptions(): Promise<Subscription[]> {
  // ensure `dueDate` is indexed.
  return db.subscriptions.find({ dueDate: { $lte: new Date() } });
}
```

**Bien:**

```ts
function getActiveSubscriptions(): Promise<Subscription[]> {
  // TODO: ensure `dueDate` is indexed.
  return db.subscriptions.find({ dueDate: { $lte: new Date() } });
}
```
