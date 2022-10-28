## Tests

Les tests sont plus importants que l'expédition. Si vous n'avez aucun test ou une
quantité insuffisante, chaque fois que vous expédiez du code, vous ne serez pas
sûr de ne rien casser.
Décider de ce qui constitue un montant adéquat appartient à votre équipe, mais
avoir une couverture à 100% (tous les relevés et succursales)
est la façon dont vous obtenez une confiance très élevée et une tranquillité
d'esprit de développeur Cela signifie qu'en plus d'avoir un excellent cadre de
test, vous devez également utiliser un bon [outil de couverture](https://github.com/gotwarlost/istanbul).

Il n'y a aucune excuse pour ne pas écrire de tests. Il existe [beaucoup de bons frameworks de test JS](http://jstherightway.org/#testing-tools) avec prise en charge des typages pour TypeScript, alors trouvez
celui que votre équipe préfère. Lorsque vous en trouvez un qui fonctionne pour
votre équipe, essayez de toujours écrire des tests pour chaque nouvelle
fonctionnalité/module que vous introduisez. Si votre méthode préférée est le
développement piloté par les tests (TDD), c'est très bien, mais le principal
est de vous assurer que vous atteignez vos objectifs de couverture avant de
lancer une fonctionnalité ou de refactoriser une fonctionnalité existante.

### Les trois lois du TDD

1. Vous n'êtes pas autorisé à écrire un code de production, sauf pour effectuer
un test unitaire ayant échoué.

2. Vous n'êtes pas autorisé à écrire plus d'un test unitaire que ce qui est
suffisant pour échouer; et les échecs de compilation sont des échecs.

3. Vous n'êtes pas autorisé à écrire plus de code de production qu'il n'en faut
pour réussir le test unitaire ayant échoué.



### Les règles de F.I.R.S.T

Les tests propres doivent suivre les règles:

- **Fast**: les tests *rapides* doivent être rapides car nous voulons les exécuter fréquemment.

- **Independent**: les tests *indépendants* ne devraient pas dépendre les uns des autres.
Ils doivent fournir la même sortie, qu'ils soient exécutés indépendamment ou tous
ensemble dans n'importe quel ordre.

- **Repeatable**: les tests doivent être reproductibles dans tous les environnements et
ne soyez pas une excuse pour pourquoi ils échouent.

- **Self-Validating**: un test doit répondre avec *Réussi* ou *Échoué*. Vous n'avez
pas besoin de comparer les fichiers journaux pour répondre si un test a réussi.

- **Timely**: des tests unitaires *opportuns* doivent être écrits avant le code de
production. Si vous écrivez des tests après le code de production, vous pourriez
trouver les tests d'écriture trop difficiles.


### Concept unique par test

Les tests doivent également suivre le *principe de responsabilité unique*. Faites
une seule assertion par test unitaire.

**Mal:**

```ts
import { assert } from 'chai';

describe('AwesomeDate', () => {
  it('handles date boundaries', () => {
    let date: AwesomeDate;

    date = new AwesomeDate('1/1/2015');
    assert.equal('1/31/2015', date.addDays(30));

    date = new AwesomeDate('2/1/2016');
    assert.equal('2/29/2016', date.addDays(28));

    date = new AwesomeDate('2/1/2015');
    assert.equal('3/1/2015', date.addDays(28));
  });
});
```

**Bien:**

```ts
import { assert } from 'chai';

describe('AwesomeDate', () => {
  it('handles 30-day months', () => {
    const date = new AwesomeDate('1/1/2015');
    assert.equal('1/31/2015', date.addDays(30));
  });

  it('handles leap year', () => {
    const date = new AwesomeDate('2/1/2016');
    assert.equal('2/29/2016', date.addDays(28));
  });

  it('handles non-leap year', () => {
    const date = new AwesomeDate('2/1/2015');
    assert.equal('3/1/2015', date.addDays(28));
  });
});
```

### Le nom du test doit révéler son intention

Lorsqu'un test échoue, son nom est la première indication de ce qui peut avoir mal tourné.

**Mal:**

```ts
describe('Calendar', () => {
  it('2/29/2020', () => {
    // ...
  });

  it('throws', () => {
    // ...
  });
});
```

**Bien:**

```ts
describe('Calendar', () => {
  it('should handle leap year', () => {
    // ...
  });

  it('should throw when format is invalid', () => {
    // ...
  });
});
```

