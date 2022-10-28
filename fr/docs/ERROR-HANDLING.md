## Gestion des Erreurs

Les erreurs lancées sont une bonne chose! Elles signifient que le runtime a
réussi à identifier quand quelque chose dans votre programme a mal tourné et
qu'il vous informe en arrêtant la fonction exécution sur la pile actuelle, tuant
le processus (dans Node), et vous notifiant dans la console avec une trace de pile.

### Utiliser toujours "Error" pour lancer ou rejeter une erreur

JavaScript, ainsi que TypeScript, vous permet de `throw` ou "lancer" n'importe quel
objet. Une promesse peut également être rejetée avec n'importe quel objet de motif.
Il est conseillé d'utiliser la syntaxe `throw` avec un type`Error`. C'est parce que
votre erreur peut être interceptée dans un code de niveau supérieur avec une syntaxe `catch`.
Il serait très déroutant d’y attraper un message de chaîne et
[débogage plus douloureux](https://basarat.gitbook.io/typescript/type-system/exceptions#always-use-error).
Pour la même raison, vous devez rejeter les promesses avec des types `Error`.

**Mal:**

```ts
function calculateTotal(items: Item[]): number {
  throw 'Not implemented.';
}

function get(): Promise<Item[]> {
  return Promise.reject('Not implemented.');
}
```

**Bien:**

```ts
function calculateTotal(items: Item[]): number {
  throw new Error('Not implemented.');
}

function get(): Promise<Item[]> {
  return Promise.reject(new Error('Not implemented.'));
}

// or equivalent to:

async function get(): Promise<Item[]> {
  throw new Error('Not implemented.');
}
```

L'avantage de l'utilisation des types `Error` est qu'il est supporté par la
syntaxe`try/catch/finally` et implicitement toutes les erreurs ont l'attribut
`stack` qui est très puissante pour le débogage. Il existe également d'autres
alternatives, pour ne pas utiliser la syntaxe `throw` et renvoyer à la place
toujours des objets d'erreur personnalisés. TypeScript rend cela encore plus
facile. Prenons l'exemple suivant:

```ts
type Result<R> = { isError: false, value: R };
type Failure<E> = { isError: true, error: E };
type Failable<R, E> = Result<R> | Failure<E>;

function calculateTotal(items: Item[]): Failable<number, 'empty'> {
  if (items.length === 0) {
    return { isError: true, error: 'empty' };
  }

  // ...
  return { isError: false, value: 42 };
}
```

Pour l'explication détaillée de cette idée, reportez-vous à la
[publication d'origine](https://medium.com/@dhruvrajvanshi/making-exceptions-type-safe-in-typescript-c4d200ee78e9).

### Ne pas ignorer pas les erreurs capturées

Ne rien faire avec une erreur détectée ne vous donne pas la possibilité de corriger
ou de réagir à cette erreur. L'enregistrement de l'erreur sur la console (`console.log`)
n'est pas beaucoup mieux car il arrive souvent qu'elle se perde dans un océan de
choses imprimées sur la console. Si vous enveloppez un morceau de code dans un
`try/catch` cela signifie que vous pensez qu'une erreur peut s'y produire et que
vous devez donc avoir un plan, ou créer un chemin de code, pour quand il se produit.

**Mal:**

```ts
try {
  functionThatMightThrow();
} catch (error) {
  console.log(error);
}

// or even worse

try {
  functionThatMightThrow();
} catch (error) {
  // ignore error
}
```

**Bien:**

```ts
import { logger } from './logging'

try {
  functionThatMightThrow();
} catch (error) {
  logger.log(error);
}
```

### Ne pas ignorer pas les promesses rejetées

Pour la même raison, vous ne devez pas ignorer les erreurs interceptées de `try/catch`.

**Mal:**

```ts
getUser()
  .then((user: User) => {
    return sendEmail(user.email, 'Welcome!');
  })
  .catch((error) => {
    console.log(error);
  });
```

**Bien:**

```ts
import { logger } from './logging'

getUser()
  .then((user: User) => {
    return sendEmail(user.email, 'Welcome!');
  })
  .catch((error) => {
    logger.log(error);
  });

// or using the async/await syntax:

try {
  const user = await getUser();
  await sendEmail(user.email, 'Welcome!');
} catch (error) {
  logger.log(error);
}
```
