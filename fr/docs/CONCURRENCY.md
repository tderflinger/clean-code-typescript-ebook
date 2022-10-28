## Opérations Concurrentes

### Préférer les promesses aux "callbacks"

Les callbacks ne sont pas propres et provoquent des quantités excessives
d'imbrication *(l'enfer des callbacks)*. Il existe des utilitaires qui transforment
les fonctions existantes en utilisant le style du callback en une version qui renvoie
des promesses (pour Node.js,
voir [`util.promisify`](https://nodejs.org/dist/latest-v8.x/docs/api/util.html#util_util_promisify_original),
pour un usage général, voir [pify](https://www.npmjs.com/package/pify), [es6-promisify](https://www.npmjs.com/package/es6-promisify))

**Mal:**

```ts
import { get } from 'request';
import { writeFile } from 'fs';

function downloadPage(url: string, saveTo: string, callback: (error: Error, content?: string) => void) {
  get(url, (error, response) => {
    if (error) {
      callback(error);
    } else {
      writeFile(saveTo, response.body, (error) => {
        if (error) {
          callback(error);
        } else {
          callback(null, response.body);
        }
      });
    }
  });
}

downloadPage('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', 'article.html', (error, content) => {
  if (error) {
    console.error(error);
  } else {
    console.log(content);
  }
});
```

**Bien:**

```ts
import { get } from 'request';
import { writeFile } from 'fs';
import { promisify } from 'util';

const write = promisify(writeFile);

function downloadPage(url: string, saveTo: string): Promise<string> {
  return get(url)
    .then(response => write(saveTo, response));
}

downloadPage('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', 'article.html')
  .then(content => console.log(content))
  .catch(error => console.error(error));
```

Les promesses prennent en charge quelques méthodes d'assistance qui aident à rendre le code plus concis:

| Modèle                  | Description                                |
| ------------------------ | -----------------------------------------|
| `Promise.resolve(value)` | Convertit une valeur en promesse résolue.|
| `Promise.reject(error)`  | Convertit une erreur en une promesse rejetée.  |
| `Promise.all(promises)`  | Renvoie une nouvelle promesse qui est remplie avec un tableau de valeurs de réalisation pour les promesses ou les refus passés avec la raison de la première promesse qui rejette. |
| `Promise.race(promises)`| Renvoie une nouvelle promesse qui est remplie/rejetée avec le résultat/l'erreur de la première promesse réglée à partir du tableau des promesses passées.|

`Promise.all` est particulièrement utile lorsqu'il est nécessaire d'exécuter
des tâches en parallèle. `Promise.race` facilite l'implémentation de choses comme
les délais d'attente pour les promesses.

### Async/Await sont encore plus propres que les promesses

Avec la syntaxe `async`/`await`, vous pouvez écrire du code beaucoup plus propre et
plus compréhensible que les promesses enchaînées. Dans une fonction préfixée par
le mot clé `async`, vous avez un moyen de dire au runtime de JavaScript de suspendre
l'exécution du code sur le mot clé`await` (lorsqu'il est utilisé sur une promesse).

**Mal:**

```ts
import { get } from 'request';
import { writeFile } from 'fs';
import { promisify } from 'util';

const write = util.promisify(writeFile);

function downloadPage(url: string, saveTo: string): Promise<string> {
  return get(url).then(response => write(saveTo, response));
}

downloadPage('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', 'article.html')
  .then(content => console.log(content))
  .catch(error => console.error(error));
```

**Bien:**

```ts
import { get } from 'request';
import { writeFile } from 'fs';
import { promisify } from 'util';

const write = promisify(writeFile);

async function downloadPage(url: string, saveTo: string): Promise<string> {
  const response = await get(url);
  await write(saveTo, response);
  return response;
}

// somewhere in an async function
try {
  const content = await downloadPage('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', 'article.html');
  console.log(content);
} catch (error) {
  console.error(error);
}
```
