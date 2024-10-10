### Migrations

Migrations are an essential part of managing a database schema over time. They allow developers to define changes to the database structure, such as creating tables, modifying existing columns, or removing tables. Using migrations ensures that the database schema stays in sync with the application code, especially when working in teams or deploying applications to production.

In this section, we will explore how to handle migrations using **TypeORM**, **Prisma**, and **Mongoose**.

#### 1. TypeORM Migrations

TypeORM provides a robust migration system that allows you to create, run, and revert migrations. The migrations are typically written in TypeScript and allow you to define changes to the database schema programmatically.

##### Creating a Migration

To create a new migration, you can use the TypeORM CLI. First, ensure you have the CLI installed:

```bash
npm install typeorm -g
```

Then, generate a new migration:

```bash
typeorm migration:create -n MigrationName
```

This command creates a new migration file in the `migrations` directory.

##### Writing a Migration

In the generated migration file, you will implement the `up` and `down` methods. The `up` method contains the logic to apply the migration, while the `down` method contains the logic to revert it.

```typescript
import { MigrationInterface, QueryRunner, Table } from 'typeorm';

export class CreateUsersTable1645789000000 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(
      new Table({
        name: 'users',
        columns: [
          {
            name: 'id',
            type: 'int',
            isPrimary: true,
            isGenerated: true,
          },
          {
            name: 'username',
            type: 'varchar',
            isUnique: true,
          },
          {
            name: 'email',
            type: 'varchar',
          },
          {
            name: 'password',
            type: 'varchar',
          },
        ],
      }),
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable('users');
  }
}
```

In this example:
- The `up` method creates a new `users` table with the specified columns.
- The `down` method removes the `users` table, effectively reverting the migration.

##### Running Migrations

To run your migrations, execute the following command:

```bash
typeorm migration:run
```

This command applies any pending migrations to the database.

##### Reverting Migrations

To revert the last migration, you can use the command:

```bash
typeorm migration:revert
```

This command will execute the `down` method of the most recent migration.

#### 2. Prisma Migrations

Prisma offers a powerful migration system integrated with its schema file. Changes to the Prisma schema can be automatically converted into SQL migration scripts.

##### Creating a Migration

To create a migration in Prisma, use the following command after making changes to your `schema.prisma` file:

```bash
npx prisma migrate dev --name MigrationName
```

This command generates a new migration file in the `prisma/migrations` directory and applies it to your development database.

##### Writing a Migration

Prisma automatically generates migration files based on the changes you make to the schema. Each migration file contains the SQL necessary to apply the changes to the database.

For example, if you change your `User` model to add a `createdAt` timestamp:

```prisma
model User {
  id        Int      @id @default(autoincrement())
  username  String   @unique
  email     String   @unique
  password  String
  createdAt DateTime @default(now()) // New field
}
```

After running the migration command, Prisma will create an SQL migration file to add the `createdAt` column to the `users` table.

##### Running Migrations

Migrations are automatically applied when you run the `migrate dev` command, but you can also apply pending migrations to a production database using:

```bash
npx prisma migrate deploy
```

This command applies all pending migrations to the production database without creating new migration files.

##### Rolling Back Migrations

If you need to revert the last migration, you can use the command:

```bash
npx prisma migrate reset
```

This command will drop the database, recreate it, and reapply all migrations. Use this with caution, especially in production environments, as it will delete all data.

#### 3. Mongoose Migrations

Mongoose does not have a built-in migration system like TypeORM or Prisma, but you can manage migrations using libraries like **migrate-mongo** or custom scripts. This is important for maintaining version control over your MongoDB schema.

##### Using migrate-mongo

To use **migrate-mongo**, first, install it:

```bash
npm install migrate-mongo
```

##### Initializing migrate-mongo

Next, initialize it in your project:

```bash
npx migrate-mongo init
```

This command creates a `migrate-mongo-config.js` file and a `migrations` directory.

##### Creating a Migration

You can create a new migration file using:

```bash
npx migrate-mongo create MigrationName
```

This command generates a new migration file in the `migrations` directory.

##### Writing a Migration

In the generated migration file, you can define the `up` and `down` functions to apply and revert the migration.

```javascript
module.exports = {
  async up(db, client) {
    await db.collection('users').insertMany([
      { username: 'user1', email: 'user1@example.com', password: 'password1' },
      { username: 'user2', email: 'user2@example.com', password: 'password2' },
    ]);
  },

  async down(db, client) {
    await db.collection('users').deleteMany({ username: { $in: ['user1', 'user2'] } });
  },
};
```

##### Running Migrations

To apply the migrations, run:

```bash
npx migrate-mongo up
```

##### Rolling Back Migrations

To revert the last migration, you can use:

```bash
npx migrate-mongo down
```

This command will execute the `down` function of the most recent migration.

---

When you're ready, say "Next" to move on to **Best Practices**, where we will discuss strategies for effective data modeling, including naming conventions, performance considerations, and more.
