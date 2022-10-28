## Variables

### Utiliser des noms de variables descriptifs

Distinguer les noms de façon que le lecteur sache ce que les différences offrent.

**Mal:**

```ts
function between<T>(a1: T, a2: T, a3: T): boolean {
  return a2 <= a1 && a1 <= a3;
}

```

**Bien:**

```ts
function between<T>(value: T, left: T, right: T): boolean {
  return left <= value && value <= right;
}
```


### Utiliser des noms de variables prononçables

Si vous ne pouvez pas le prononcer, vous ne pouvez pas en discuter sans ressembler à un idiot.

**Mal:**

```ts
type DtaRcrd102 = {
  genymdhms: Date;
  modymdhms: Date;
  pszqint: number;
}
```

**Bien:**

```ts
type Customer = {
  generationTimestamp: Date;
  modificationTimestamp: Date;
  recordId: number;
}
```

### Utiliser le même vocabulaire pour le même type de variable

**Mal:**

```ts
function getUserInfo(): User;
function getUserDetails(): User;
function getUserData(): User;
```

**Bien:**

```ts
function getUser(): User;
```

### Utiliser des noms consultables

Nous lirons plus de code que nous n'en aurons jamais écrit. Il est important que
le code que nous écrivons soit lisible et consultable. En *ne nommant pas* les
variables qui finissent par être significatives pour comprendre notre programme,
nous blessons nos lecteurs. Rendez vos noms consultables. Des outils comme
[TSLint](https://palantir.github.io/tslint/rules/no-magic-numbers/) peuvent aider
à identifier les constantes sans nom.

**Mal:**

```ts
// What the heck is 86400000 for?
setTimeout(restart, 86400000);
```

**Bien:**

```ts
// Declare them as capitalized named constants.
const MILLISECONDS_IN_A_DAY = 24 * 60 * 60 * 1000;

setTimeout(restart, MILLISECONDS_IN_A_DAY);
```

### Utiliser des variables explicatives

**Mal:**

```ts
declare const users: Map<string, User>;

for (const keyValue of users) {
  // iterate through users map
}
```

**Bien:**

```ts
declare const users: Map<string, User>;

for (const [id, user] of users) {
  // iterate through users map
}
```



### Évitez le mapping mental

Mieux vaut être explicite qu'implicite.
*La clarté est essentielle.*

**Mal:**

```ts
const u = getUser();
const s = getSubscription();
const t = charge(u, s);
```

**Bien:**

```ts
const user = getUser();
const subscription = getSubscription();
const transaction = charge(user, subscription);
```

### Ne pas ajouter de contexte inutile

Si votre nom de classe/type/objet vous dit quelque chose, ne répétez pas cela dans
le nom de votre variable.

**Mal:**

```ts
type Car = {
  carMake: string;
  carModel: string;
  carColor: string;
}

function print(car: Car): void {
  console.log(`${car.carMake} ${car.carModel} (${car.carColor})`);
}
```

**Bien:**

```ts
type Car = {
  make: string;
  model: string;
  color: string;
}

function print(car: Car): void {
  console.log(`${car.make} ${car.model} (${car.color})`);
}
```

### Utiliser des arguments par défaut au lieu d'un raccourci ou de conditions

Les arguments par défaut sont souvent plus propres que les raccourcis.

**Mal:**

```ts
function loadPages(count?: number) {
  const loadCount = count !== undefined ? count : 10;
  // ...
}
```

**Bien:**

```ts
function loadPages(count: number = 10) {
  // ...
}
```


### Utiliser enum pour documenter l'intention

Les énumérations peuvent vous aider à documenter l'intention du code. Par exemple,
lorsque on veut que les valeurs soient différentes plutôt que la valeur exacte de
celles-ci.

**Mal:**

```ts
const GENRE = {
  ROMANTIC: 'romantic',
  DRAMA: 'drama',
  COMEDY: 'comedy',
  DOCUMENTARY: 'documentary',
}

projector.configureFilm(GENRE.COMEDY);

class Projector {
  // declaration of Projector
  configureFilm(genre) {
    switch (genre) {
      case GENRE.ROMANTIC:
        // some logic to be executed
    }
  }
}
```

**Bien:**

```ts
enum GENRE {
  ROMANTIC,
  DRAMA,
  COMEDY,
  DOCUMENTARY,
}

projector.configureFilm(GENRE.COMEDY);

class Projector {
  // declaration of Projector
  configureFilm(genre) {
    switch (genre) {
      case GENRE.ROMANTIC:
        // some logic to be executed
    }
  }
}
```
