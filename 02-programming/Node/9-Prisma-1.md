# Prisma 1

Prisma est un ORM (Object Relational Mapper) nouvelle génération qui permet à nos script JS exécutés via Node.js de communiquer avec une base de donnée ou un serveur GraphQL.  
L'avantage d'un ORM est qu'il est générique et qu'il peut supporter plusieuress bases de données, notamment Postgresql.  
De plus en tant qu'ORM nous pouvons récupérer directement des objets Javascripts suite à nos requetes à notre base de donnée. Il effectue un mapping entre résultat de requêtes SQL vers des objets Javascript.

## Installation

https://www.prisma.io/docs/getting-started/setup-prisma/add-to-existing-project/relational-databases-typescript-postgres

Prisma est un package Node.js, il faudra donc installer Prisma via `yarn` dans un projet Node.js déjà existant (par exemple notre application express).

```zsh
mkdir hello-prisma
cd hello-prisma
yarn init -y
yarn add prisma
```

Il faut ensuite initialiser Prisma et lui fournir les informations de connections à notre base de donnée:

```zsh
npx prisma init
```

Un fichier \_prisma/schema.prisma\_ sera généré:

```js
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}
```

Par défaut la base de donnée est bien `postgresql` et l'url vers notre base de donnée est récupérée via
une variable d'environment.  
Il nous faut donc un fichier `.env` qui assignera la variable d'environement `DATABASE_URL`.  
Le fichier `.env` est également auto-généré il faudra le modifier avec les informations de connexion à notre base de donnée:
_.env_:

```zsh
DATABASE_URL="postgresql://db_user:strongpassword123@localhost:5432/db_hello_prisma?schema=public"
```

**ATTENTION A BIEN CREER UN FICHIER `.gitignore` QUI EXCLURA CE FICHIER `.env` CAR IL POSSEDE DES INFORMATIONS CRITIQUES QUI NE DOIVENT JAMAIS SE RETROUVER SUR GITHUB**

## Generating models with instrospection

Nous pouvons créer notre base de donnée depuis Prisma, mais nous allons dans ce cours partir de l'hypothèse qu'une base de donnée existe déjà.  
Pour cela il faut la créer:

```zsh
createdb -U db_user db_hello_prisma
psql -d db_hello_prisma -U db_user -a -f db_hello_prisma.sql
```

Avec le fichier _db_hello_prisma.sql_ comme ci dessous:

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY NOT NULL,
  login VARCHAR(30) NOT NULL UNIQUE,
  "firstName" VARCHAR(255),
  "lastName" VARCHAR(255),
  email VARCHAR(255) UNIQUE NOT NULL
);

CREATE TABLE posts (
  id SERIAL PRIMARY KEY NOT NULL,
  title VARCHAR(255) NOT NULL,
  "createdAt" TIMESTAMP NOT NULL DEFAULT now(),
  content TEXT,
  published BOOLEAN NOT NULL DEFAULT false,
  "authorId"  INTEGER NOT NULL REFERENCES users(id)
);

CREATE TABLE profiles (
  id SERIAL PRIMARY KEY NOT NULL,
  bio TEXT,
  "userId" INTEGER UNIQUE NOT NULL REFERENCES users(id)
);
```

Prisma à besoin de récupérer les tables de notre base de données afin de les traduire en `Prisma data model`.

```zsh
npx prisma introspect
```

En consultant le fichier `prisma/schema.prisma` vous remarquerez que de nouveaux modèles ont été ajoutés.
Le fichier `prisma/schema.prisma` est le `Prisma data model`, il sert à configurer le Prisma client qui effectuera les requêtes à notre base de donnée.

## Prisma client

client installation:

```zsh
yarn add @prisma/client
```

client generation:

```zsh
npx prisma generate
```

Le Prisma client généré est la librairie que nous utiliserons pour nous connecter à notre base de donnée.
Si les tables de notre base de donnée sont modifiées il faudra de nouveau générer le Prisma client.

## Query our database

Nous pouvons désormais interroger notre base de donnée depuis nos scripts javascript.  
Il nous faudra pour cela importer le Prisma client et créer une instance de ce client que nous utiliserons dans notre code:
_app.js_

```js
// import
const { PrismaClient } = require('@prisma/client')

// instanciation d'un nouveau prisma client
const prisma = new PrismaClient()

const main = async () => {
  // Equivalent d'un SELECT * FROM users;
  const allUsers = await prisma.users.findMany()
  // Affichage du résultat
  console.log(allUsers)
}

// Execution de la fonction main
main()
  .catch((e) => {
    throw e
  })
  .finally(async () => {
    // Fermeture de la connection à la fin de l'exécution du script
    await prisma.$disconnect()
  })
```

La méthode `findMany` est l'équivalent d'un `SELECT`, elle permet de lire une base de donnée et de récupérer plusieurs résultats/lignes.
Notre base de donnée est vide, c'est pour cela que notre output est `[]`.
Créons une entrée dans notre base de donnée, ajoutons l'utilisatrice 'alice' grâce à la méthode `create`:

```js
async function main() {
  // Equivalent en SQL:
  // INSERT INTO users(login, "firstName", "lastName", email) VALUES ('alice', 'Alice', 'Euler', 'alice@mail.com');
  // Le resultat de l'insertion est automatiquement retourné
  const result = await prisma.users.create({
    data: {
      login: 'alice',
      firstName: 'Alice',
      lastName: 'Euler',
      email: 'alice@gmail.com',
    },
  })
  console.log(result)
```

Nous pouvons créer plusieurs entrées dans la même table avec `createMany`:

```js
const nbInserted = await prisma.users.createMany({
  data: [
    {
      login: 'bob',
      firstName: 'Bob',
      lastName: 'Durac',
      email: 'bob@mail.com',
    },
    {
      login: 'charlie',
      firstName: 'Charlie',
      lastName: 'Turing',
      email: 'charlie@mail.com',
    },
  ],
  // ignore les duplications
  skipDuplicates: true,
})
console.log(nbInserted)
```

Nous pouvons maintenant récupérer toutes les entrées de la table `users` avec:

```js
// users sera un tableau d'objets. Chacune des clefs de ces objets correspondra au nom d'une colonne.
const users = await prisma.users.findMany()
console.log(users)
```

output:

```zsh
[
  {
    id: 1,
    login: 'alice',
    firstName: 'Alice',
    lastName: 'Euler',
    email: 'alice@gmail.com'
  },
  {
    id: 2,
    login: 'bob',
    firstName: 'Bob',
    lastName: 'Durac',
    email: 'bob@mail.com'
  },
  {
    id: 3,
    login: 'charlie',
    firstName: 'Charlie',
    lastName: 'Turing',
    email: 'charlie@mail.com'
  }
]
```

Nous pouvons choisir de ne retenir que les colonnes qui nous intéressent avec la clef `select` et les colonnes qui nous intéressent set à `true` comme ci dessous:

```js
// Equivalent en SQL:
// SELECT login, "lastName", "firstName" FROM users;
const users = await prisma.users.findMany({
  select: {
    login: true,
    lastName: true,
    firstName: true,
  },
})
console.log(users)
```

Nous avons aussi la possibilité de filtrer les résultats en fonction de certains conditions avec la clef `where`:

```js
// Equivalent en SQL:
// SELECT login, "lastName", "firstName" FROM users WHERE id > 1;
const users = await prisma.users.findMany({
  const users = await prisma.users.findMany({
    where: {
      id: {
        gt: 1, // greater than
      },
    },
    select: {
      login: true,
      lastName: true,
      firstName: true,
    },
  })
})
console.log(users)
```

L'équivalent de `ORDER BY` et `LIMIT` est également disponible:

```js
// Equivalent en SQL:
// SELECT login, "lastName", "firstName" FROM users WHERE id > 1 ORDER BY id DESC LIMIT 1;
const users = await prisma.users.findMany({
  take: 1,
  orderBy: {
    id: 'desc',
  },
  where: {
    id: {
      gt: 1,
    },
  },
  select: {
    login: true,
    lastName: true,
    firstName: true,
  },
})
console.log(users)
```

## Raw queries

https://www.prisma.io/docs/concepts/components/prisma-client/raw-database-access#queryraw

Etant générique, et aussi assez récent, certaines fonctionnalités ne sont pas encore supportées sur Prisma.  
Pour cela il faudra donc utiliser des `raw queries`. Elles consistent à exécuter une requêtes SQL avec `$queryRaw`.

```js
const result = await prisma.$queryRaw('SELECT * FROM users WHERE id % 2 != 0;')
console.log(result)
```

Il est actuellement impossible d'exprimer cette requête SQL avec des filtres Prisma, il faudra donc passer par une `raw query`.
