## Functions

### Function arguments (2 or fewer ideally)

Limiting the number of function parameters is incredibly important because it makes testing your function easier.
Having more than three leads to a combinatorial explosion where you have to test tons of different cases with each separate argument.  

One or two arguments is the ideal case, and three should be avoided if possible. Anything more than that should be consolidated.
Usually, if you have more than two arguments then your function is trying to do too much.
In cases where it's not, most of the time a higher-level object will suffice as an argument.  

Consider using object literals if you are finding yourself needing a lot of arguments.  

To make it obvious what properties the function expects, you can use the [destructuring](https://basarat.gitbook.io/typescript/future-javascript/destructuring) syntax.
This has a few advantages:

1. When someone looks at the function signature, it's immediately clear what properties are being used.

2. It can be used to simulate named parameters.

3. Destructuring also clones the specified primitive values of the argument object passed into the function. This can help prevent side effects. Note: objects and arrays that are destructured from the argument object are NOT cloned.

4. TypeScript warns you about unused properties, which would be impossible without destructuring.

**Bad:**

```ts
function createMenu(title: string, body: string, buttonText: string, cancellable: boolean) {
  // ...
}

createMenu('Foo', 'Bar', 'Baz', true);
```

**Good:**

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

You can further improve readability by using [type aliases](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-aliases):

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

### Functions should do one thing

This is by far the most important rule in software engineering. When functions do more than one thing, they are harder to compose, test, and reason about. When you can isolate a function to just one action, it can be refactored easily and your code will read much cleaner. If you take nothing else away from this guide other than this, you'll be ahead of many developers.

**Bad:**

```ts
function emailActiveClients(clients: Client[]) {
  clients.forEach((client) => {
    const clientRecord = database.lookup(client);
    if (clientRecord.isActive()) {
      email(client);
    }
  });
}
```

**Good:**

```ts
function emailActiveClients(clients: Client[]) {
  clients.filter(isActiveClient).forEach(email);
}

function isActiveClient(client: Client) {
  const clientRecord = database.lookup(client);
  return clientRecord.isActive();
}
```

### Function names should say what they do

**Bad:**

```ts
function addToDate(date: Date, month: number): Date {
  // ...
}

const date = new Date();

// It's hard to tell from the function name what is added
addToDate(date, 1);
```

**Good:**

```ts
function addMonthToDate(date: Date, month: number): Date {
  // ...
}

const date = new Date();
addMonthToDate(date, 1);
```

### Functions should only be one level of abstraction

When you have more than one level of abstraction your function is usually doing too much. Splitting up functions leads to reusability and easier testing.

**Bad:**

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

**Good:**

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

### Remove duplicate code

Do your absolute best to avoid duplicate code.
Duplicate code is bad because it means that there's more than one place to alter something if you need to change some logic.  

Imagine if you run a restaurant and you keep track of your inventory: all your tomatoes, onions, garlic, spices, etc.
If you have multiple lists that you keep this on, then all have to be updated when you serve a dish with tomatoes in them.
If you only have one list, there's only one place to update!  

Oftentimes you have duplicate code because you have two or more slightly different things, that share a lot in common, but their differences force you to have two or more separate functions that do much of the same things. Removing duplicate code means creating an abstraction that can handle this set of different things with just one function/module/class.  

Getting the abstraction right is critical, that's why you should follow the [SOLID](#solid) principles. Bad abstractions can be worse than duplicate code, so be careful! Having said this, if you can make a good abstraction, do it! Don't repeat yourself, otherwise, you'll find yourself updating multiple places anytime you want to change one thing.

**Bad:**

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

**Good:**

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

function showEmployeeList(employee: (Developer | Manager)[]) {
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

You may also consider adding a union type, or common parent class if it suits your abstraction.
```ts
class Developer {
  // ...
}

class Manager {
  // ...
}

type Employee = Developer | Manager

function showEmployeeList(employee: Employee[]) {
  // ...
  });
}

```

You should be critical about code duplication. Sometimes there is a tradeoff between duplicated code and increased complexity by introducing unnecessary abstraction. When two implementations from two different modules look similar but live in different domains, duplication might be acceptable and preferred over extracting the common code. The extracted common code, in this case, introduces an indirect dependency between the two modules.


### Set default objects with Object.assign or destructuring

**Bad:**

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

**Good:**

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

Or, you could use the spread operator:

```ts
function createMenu(config: MenuConfig) {
  const menuConfig = {
    title: 'Foo',
    body: 'Bar',
    buttonText: 'Baz',
    cancellable: true,
    ...config,
  };

  // ...
}
```
The spread operator and `Object.assign()` are very similar.
The main difference is that spreading defines new properties, while `Object.assign()` sets them. More detailed, the difference is explained in [this](https://stackoverflow.com/questions/32925460/object-spread-vs-object-assign) thread.

Alternatively, you can use destructuring with default values:

```ts
type MenuConfig = { title?: string, body?: string, buttonText?: string, cancellable?: boolean };

function createMenu({ title = 'Foo', body = 'Bar', buttonText = 'Baz', cancellable = true }: MenuConfig) {
  // ...
}

createMenu({ body: 'Bar' });
```

To avoid any side effects and unexpected behavior by passing in explicitly the `undefined` or `null` value, you can tell the TypeScript compiler to not allow it.
See [`--strictNullChecks`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html#--strictnullchecks) option in TypeScript.


### Don't use flags as function parameters

Flags tell your user that this function does more than one thing.
Functions should do one thing. Split out your functions if they are following different code paths based on a boolean.

**Bad:**

```ts
function createFile(name: string, temp: boolean) {
  if (temp) {
    fs.create(`./temp/${name}`);
  } else {
    fs.create(name);
  }
}
```

**Good:**

```ts
function createTempFile(name: string) {
  createFile(`./temp/${name}`);
}

function createFile(name: string) {
  fs.create(name);
}
```

### Avoid Side Effects (part 1)

A function produces a side effect if it does anything other than take a value in and return another value or values.
A side effect could be writing to a file, modifying some global variable, or accidentally wiring all your money to a stranger.  

Now, you do need to have side effects in a program on occasion. Like the previous example, you might need to write to a file.
What you want to do is to centralize where you are doing this. Don't have several functions and classes that write to a particular file.
Have one service that does it. One and only one.  

The main point is to avoid common pitfalls like sharing state between objects without any structure, using mutable data types that can be written to by anything, and not centralizing where your side effects occur. If you can do this, you will be happier than the vast majority of other programmers.

**Bad:**

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

**Good:**

```ts
const name = 'Robert C. Martin';

function toBase64(text: string): string {
  return btoa(text);
}

const encodedName = toBase64(name);
console.log(name);
```

### Avoid Side Effects (part 2)

Browsers and Node.js process only JavaScript, therefore any TypeScript code has to be compiled before running or debugging.  In JavaScript, some values are unchangeable (immutable) and some are changeable (mutable). Objects and arrays are two kinds of mutable values so it's important to handle them carefully when they're passed as parameters to a function. A JavaScript function can change an object's properties or alter the contents of an array which could easily cause bugs elsewhere.

Suppose there's a function that accepts an array parameter representing a shopping cart. If the function makes a change in that shopping cart array - by adding an item to purchase, for example - then any other function that uses that same `cart` array will be affected by this addition. That may be great, however it could also be bad. Let's imagine a bad situation:

The user clicks the "Purchase" button which calls a `purchase` function that spawns a network request and sends the `cart` array to the server. Because of a bad network connection, the `purchase` function has to keep retrying the request. Now, what if in the meantime the user accidentally clicks an "Add to Cart" button on an item they don't actually want before the network request begins? If that happens and the network request begins, then that purchase function will send the accidentally added item because the `cart` array was modified.

A great solution would be for the `addItemToCart` function to always clone the `cart`, edit it, and return the clone. This would ensure that functions that are still using the old shopping cart wouldn't be affected by the changes.

Two caveats to mention to this approach:

1. There might be cases where you actually want to modify the input object, but when you adopt this programming practice you will find that those cases are pretty rare. Most things can be refactored to have no side effects! (see [pure function](https://en.wikipedia.org/wiki/Pure_function))

2. Cloning big objects can be very expensive in terms of performance. Luckily, this isn't a big issue in practice because there are [great libraries](https://github.com/immutable-js/immutable-js) that allow this kind of programming approach to be fast and not as memory intensive as it would be for you to manually clone objects and arrays.

**Bad:**

```ts
function addItemToCart(cart: CartItem[], item: Item): void {
  cart.push({ item, date: Date.now() });
};
```

**Good:**

```ts
function addItemToCart(cart: CartItem[], item: Item): CartItem[] {
  return [...cart, { item, date: Date.now() }];
};
```

### Don't write to global functions

Polluting globals is a bad practice in JavaScript because you could clash with another library and the user of your API would be none-the-wiser until they get an exception in production. Let's think about an example: what if you wanted to extend JavaScript's native Array method to have a `diff` method that could show the difference between two arrays? You could write your new function to the `Array.prototype`, but it could clash with another library that tried to do the same thing. What if that other library was just using `diff` to find the difference between the first and last elements of an array? This is why it would be much better to just use classes and simply extend the `Array` global.

**Bad:**

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

**Good:**

```ts
class MyArray<T> extends Array<T> {
  diff(other: T[]): T[] {
    const hash = new Set(other);
    return this.filter(elem => !hash.has(elem));
  };
}
```

### Favor functional programming over imperative programming

Favor this style of programming when you can.

**Bad:**

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

**Good:**

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


### Encapsulate conditionals

**Bad:**

```ts
if (subscription.isTrial || account.balance > 0) {
  // ...
}
```

**Good:**

```ts
function canActivateService(subscription: Subscription, account: Account) {
  return subscription.isTrial || account.balance > 0;
}

if (canActivateService(subscription, account)) {
  // ...
}
```

### Avoid negative conditionals

**Bad:**

```ts
function isEmailNotUsed(email: string): boolean {
  // ...
}

if (isEmailNotUsed(email)) {
  // ...
}
```

**Good:**

```ts
function isEmailUsed(email: string): boolean {
  // ...
}

if (!isEmailUsed(email)) {
  // ...
}
```

### Avoid conditionals

This seems like an impossible task. Upon first hearing this, most people say, "how am I supposed to do anything without an `if` statement?" The answer is that you can use polymorphism to achieve the same task in many cases. The second question is usually, "well that's great but why would I want to do that?" The answer is a previous clean code concept we learned: a function should only do one thing. When you have classes and functions that have `if` statements, you are telling your user that your function does more than one thing. Remember, just do one thing.

**Bad:**

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

**Good:**

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

### Avoid type checking

TypeScript is a strict syntactical superset of JavaScript and adds optional static type checking to the language.
Always prefer to specify types of variables, parameters and return values to leverage the full power of TypeScript features.
It makes refactoring more easier.

**Bad:**

```ts
function travelToTexas(vehicle: Bicycle | Car) {
  if (vehicle instanceof Bicycle) {
    vehicle.pedal(currentLocation, new Location('texas'));
  } else if (vehicle instanceof Car) {
    vehicle.drive(currentLocation, new Location('texas'));
  }
}
```

**Good:**

```ts
type Vehicle = Bicycle | Car;

function travelToTexas(vehicle: Vehicle) {
  vehicle.move(currentLocation, new Location('texas'));
}
```

### Don't over-optimize

Modern browsers do a lot of optimization under-the-hood at runtime. A lot of times, if you are optimizing then you are just wasting your time. There are good [resources](https://github.com/petkaantonov/bluebird/wiki/Optimization-killers) for seeing where optimization is lacking. Target those in the meantime, until they are fixed if they can be.

**Bad:**

```ts
// On old browsers, each iteration with uncached `list.length` would be costly
// because of `list.length` recomputation. In modern browsers, this is optimized.
for (let i = 0, len = list.length; i < len; i++) {
  // ...
}
```

**Good:**

```ts
for (let i = 0; i < list.length; i++) {
  // ...
}
```

### Remove dead code

Dead code is just as bad as duplicate code. There's no reason to keep it in your codebase.
If it's not being called, get rid of it! It will still be safe in your version history if you still need it.

**Bad:**

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

**Good:**

```ts
function requestModule(url: string) {
  // ...
}

const req = requestModule;
inventoryTracker('apples', req, 'www.inventory-awesome.io');
```

### Use iterators and generators

Use generators and iterables when working with collections of data used like a stream.  
There are some good reasons:

- decouples the callee from the generator implementation in a sense that callee decides how many
items to access
- lazy execution, items are streamed on-demand
- built-in support for iterating items using the `for-of` syntax
- iterables allow implementing optimized iterator patterns

**Bad:**

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

**Good:**

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

There are libraries that allow working with iterables in a similar way as with native arrays, by
chaining methods like `map`, `slice`, `forEach` etc. See [itiriri](https://www.npmjs.com/package/itiriri) for
an example of advanced manipulation with iterables (or [itiriri-async](https://www.npmjs.com/package/itiriri-async) for manipulation of async iterables).

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

