# Prisma 2

## schema.prisma

La configuration de notre client prisma est dans le fichier _prisma/schema.prisma_.  
Nous devrons y retrouver les blocks suivants:

- `datasource` qui correspondra au type de la base de donnée à laquelle on se souhaite se connecter ainsi que son url de connexion
- `generator` pour customiser la génération de notre client Prisma
- des `model` qui définiront nos modèles de données. Ces modèles contiendront des attributs typés.
  Les `model` contiendront également toutes les relations entre nos différentes tables.
  les `model` sont directement mappés aux tables de notre base de données.

### datasource

```js
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

le `provider` correspond au type de notre base de donnée.  
l'`url` peut être inscrite directement, pas recommandé, ou encore mieux, importée depuis une variable d'environment.
Dans l'exemple précédent il nous faudra un fichier _.env_ qui contiendra la variable `DATABASE_URL` à laquelle sera assignée l'url de connexion à notre base de donnée:

```zsh
DATABASE_URL="postgresql://db_user:strongpassword123@localhost:5432/mydb?schema=public"
```

### generator

Le block `generator` détermine comment notre client Prisma sera généré, il est recommandé d'utiliser:

```js
generator client {
  provider = "prisma-client-js"
}
```

### model

concepts: https://www.prisma.io/docs/concepts/components/prisma-schema/data-model
reference: https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference

Notre database peut être définie directement depuis Prisma dans le fichier _prisma/schema.prisma_, via des modèles
Un Prisma model est directement mappé à une table d'une base de donnée.  
Un Prisma model possède un nom et contiendra différents attributs qui seront typés, ils correspondront aux colonnes de notre table dans la base de donnée.  
Un Prisma model possède également les relations entre les models, c'est à dire les clefs étrangères entre nos tables.
Un Prisma model est défini avec le keyworld `model` suivi de préférence par un nom en PascalCase et entre accolade seront définis les champs de notre modèle ainsi que les relations qu'il possède avec les autres modèles.

```dart
model User {
  id               Int       @id @default(autoincrement())
  username         String    @unique @db.VarChar(30)
  email            String    @unique
  createdAt        DateTime  @default(now())
  apiKey           ApiKey?
  active           Boolean   @default(true)
  sentMessages     Message[] @relation("SentMessages")
  receivedMessages Message[] @relation("ReceivedMessages")

  @@map(name: "user")
}

model ApiKey {
  id     Int    @id @default(autoincrement())
  user   User   @relation(fields: [userId], references: [id])
  userId Int
  key    String @unique

  @@map(name: "api_key")
}

model Message {
  id        Int      @id @default(autoincrement())
  src       User     @relation("SentMessages", fields: [srcId], references: [id])
  srcId     Int
  dst       User     @relation("ReceivedMessages", fields: [dstId], references: [id])
  dstId     Int
  content   String
  createdAt DateTime @default(now())

  @@map(name: "message")
}
```

Afin de créer/mettre à jour notre base de donnée avec notre schéma il faut entrer la commande:

```zsh
npx prisma migrate dev --name init
```

Nos modèles ci dessus se nomme `User`, `ApiKey` et `Message`, afin de suivre la convention Postgresql, les noms des tables Postgresql correspondent au nom des modèles, mais en minuscules. Cette conversion est effectuée avec l'attribut `@@map`.  
Chaque modèle doit posséder au moins un champ avec un identifiant unique, c'est à dire qu'au moins un des champs de notre modèles devra posséder l'un des décorateurs:

```text
@unique
@@unique
@id
@@id
```

`@id` est équivalent à `PRIMARY KEY`.  
Chacun des champs de notre modèle devra posséder un type défini dans Prisma: https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference#model-field-scalar-types.  
Ce type sera automatiquement convertit en un type natif de notre base de donnée, mais nous pouvons préciser quel sera le type exactement de notre champ avec un `Native database type attribute`, comme pour:

```js
username  String   @unique @db.VarChar(30)
```

`username` ne sera pas de type `text` dans notre base de donnée mais de type `character varying(30)`.

Une valeur par défaut peut être assigné via le modifier `@default`, et afin de forcer un champ a être absolument unique dans toute la table nous pouvons utiliser le modifier `@unique`.  
Par défaut aucun champ ne peut être vide/null, pour avoir l'équivalent de `NOT NULL` d'une base de donnée Postgresql il faut rajouter un `?` juste après le type.

```js
age Int?
```

Tous les types Prisma sont listé dans l'api réference: https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference#model-field-scalar-types

Les relations fields servent à créer des relations entre les modèles. Leur type n'est pas un `scalar type`, comme listé ci dessus, mais un autre modèle.  
Chaque relation entre deux modèles est exprimée avec 2 relations fields, un pour chacune des tables qui sont en relation. Ces fields auront comme type la table qu'ils reférencent.
Il faudra absolument que l'une des tables ajoute `@relation` pour spécifier quels sont les champs liés entre les 2 tables.
Le nom de ces relations pourra ensuite être utilisé par notre client Prisma lors de nos requêtes.

## Prisma studio

Une interface graphique web pour visualisé notre base de donnée

```zsh
npx prisma studio
```
