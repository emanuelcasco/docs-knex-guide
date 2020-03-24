# Knex.ts practical guide

Practical guide to start using Knex in conjunction with TypeScript and not throw production trying.

## Index
<details><summary>Show more...</summary>
<p>

* [Index](#index)
* [Knex](#knex)
  + [QueryBuilder vs ORM](#query-builder-vs-orm)
* [Knex and TypeScript](#knex-y-typescript)
* [Installation](#installation)
  + [Database connection](#database-connection)
* [First steps](#first-steps)
  + [Select](#select)
    - [Where](#where)
    - [Join](#join)
  + [Data persistance](#data-persistance)
    - [Insert](#insert)
    - [Update](#update)
    - [Delete](#delete)
* [Migrations](#migrations)
  + [Create table](#create-table)
    - [Associations on schema](#associations-on-schema)
  + [Alter table](#alter-table)
  + [Timestamps helper](#timestamps-helper)


</p>
</details>

## Knex

[Knex.js](http://knexjs.org/) is an SQL QueryBuilder for Postgres, MSSQL, MySQL, MariaDB, SQLite3, Oracle, and Amazon Redshift designed to be flexible and portable. It features a promise interface for cleaner async flow control, a stream interface, full featured query and schema builders, transaction support, connection pooling and standardized responses between different query clients and dialects.

### QueryBuilder vs ORM

An ORM (Object-relational mapping) is a object/class within a language, i.e. JavaScript, that allows us to deal with data in the DB without dealing with the database itself (writing SQL queries). On the other hand, a QueryBuilder provides an API that is designed for constructing your queries using a language. Main difference between them is that ORMs enforces the creation of encapsulations representing your tables and offers an API to communicate with them meanwhile QueryBuilders help you to write "queries" using your favourite language.

## Knex y TypeScript

While knex is written in JavaScript, **officially supported TypeScript bindings are available** within the knex npm package, since it has `.d.ts` files in its repository that we can consume.

It's important 

It's important to keep in mind that Knex is a QueryBuilder, thus its API is extremely flexible and complex to type. According to knex's own documentation: **lack of type errors doesn't currently guarantee that the generated queries will be correct**.

Many of the APIs accept TRecord and TResult type parameters, using which we can specify the type of a row in the database table and the type of the result of the query respectively. However, we will generally use casting assuming the type of response. For these reason, **it is of utmost importance to accompany the typing of a good set of tests to ensure that there are no unexpected behaviors**.

> See: [Knex - TypeScript support](http://knexjs.org/#typescript-support)

## Installation

First, we need to install the knex library and the appropriate database driver. In our case, we'll be using [PostgreSQL](https://github.com/brianc/node-postgres) thus we need to install `pg` package:

```bash
$ npm install knex pg --save
```

### Database connection

Once installed both knex and driver, we have to create a knex instance connected to our database. We can do this as follows:

```typescript
import Knex from 'knex';

const knex = Knex({
  client: 'pg',
  connection: {
    host : '127.0.0.1',
    user : 'your_database_user',
    password : 'your_database_password',
    database : 'myapp_test'
  }
});
```

Now, we are ready to start working with knex!

## First steps

Although we provide a quick introduction to the QueryBuilder syntax, we recommend a thorough read of the [official Knex documentation](http://knexjs.org). In addition, is available a [Knex Cheatsheet](https://devhints.io/knex) more complete than this brief introduction.

### Select

#### Where

```typescript
knex
  .from('books')
  .select('title', 'author', 'year')
  .where('title', 'Hello') // or... where({ title: 'Hello' })
```

```typescript
  .where('title', 'Hello')
  .whereIn('id', [1, 2, 3])
  .whereNot(···)
  .whereNull('updated_at')
  .whereNotNull(···)
  .whereExists('updated_at')
  .whereNotExists(···)
  .whereBetween('votes', [1, 100])
  .whereNotBetween(···)
  .whereRaw('id = ?', [1])
```

> See: [Knex - Wheres](http://knexjs.org/#Builder-wheres)

#### Join

Basic example:

```typescript
knex
  .from('books')
  .join('contacts', 'users.id', 'contacts.id');
```

Other directions:

```typescript
  .leftJoin(···)
  .leftOuterJoin(···)
  .rightJoin(···)
  .rightOuterJoin(···)
  .outerJoin(···)
  .fullOuterJoin(···)
  .crossJoin(···)
  .joinRaw('natural full join table1')
```
> See: [Knex - QueryBuilder](http://knexjs.org/#Builder)

### Data persistance

#### Insert

```typescript
knex('users')
  .insert({ name: 'John' })
  .insert([
    { name: 'Starsky' },
    { name: 'Hutch' }
  ]);
```
> See: [Knex - Insert](http://knexjs.org/#Builder-insert)

#### Update

```typescript
knex('users')
  .where({ id: 2 })
  .update({ name: 'Homer' });
```
> See: [Knex - Update](http://knexjs.org/#Builder-update)

#### Delete

```typescript
knex('users')
  .where({ id: 2 })
  .del()
```
> See: [Knex - Delete](http://knexjs.org/#Builder-del)

## Migrations

### Create table

Table creation example:

```typescript

export async function up(knex: Knex): Promise<void> {
  const tableExist = await knex.schema.hasTable('banks');
  if (tableExist) return;
  await knex.schema.createTable('banks', (table: Knex.CreateTableBuilder) => {
    table
      .uuid('id')
      .primary()
      .defaultTo(knex.raw('uuid_generate_v4()'));
    table.string('name');
    table.timestamp('created_at').defaultTo(knex.fn.now());
    table.timestamp('updated_at').defaultTo(knex.fn.now());
  });
  await knex.raw(applyOnUpdateTrigger('banks'));
}

export async function down(knex: Knex): Promise<void> {
  await knex.raw(removeOnUpdateTrigger('banks'));
  await knex.schema.dropTableIfExists('banks');
}
```

> Disclaimer! Note we use `.hasTable` to check if table exists and then use `.createTable` to actually create it. Since `.createTableIfNotExists` actually just generates plain "CREATE TABLE IF NOT EXIST..." query it will not work correctly if there are any alter table queries generated for columns afterwards.

Columns:

```typescript
  table.increments('id')
  table.string('account_name')
  table.integer('age')
  table.float('age')
  table.decimal('balance', 8, 2)
  table.boolean('is_admin')
  table.date('birthday')
  table.time('created_at')
  table.timestamp('created_at').defaultTo(knex.fn.now())
  table.json('profile')
  table.jsonb('profile')
  table.uuid('id').primary()
```

Constraints:

```typescript
  table.unique('email')
  table.unique(['email', 'company_id'])
  table.dropUnique(···)
```

Indices:
```typescript
  table.foreign('company_id')
    .references('companies.id')
  table.dropForeign(···)
```

#### Associations on schema

```typescript
exports.up = async (knex: Knex): Promise<void> => {
  await knex.schema.createTable('accounts', (table: Knex.CreateTableBuilder) => {
    table
      .uuid('id')
      .primary()
      .defaultTo(knex.raw('uuid_generate_v4()'));
    // First, create column
    table.uuid('bank_id').notNullable();
    // Second, set created column as foreign key
    table.foreign('bank_id').references('banks.id');
  });
};
```
> See: [Knex - Schema builder](http://knexjs.org/#Schema)

### Alter table

```typescript
knex.schema.table('accounts', table => {
  // Create
  table.string('first_name')
  // Alter
  table.string('first_name').alter()
  table.renameColumn('admin', 'is_admin')
  // Drop
  table.dropColumn('admin')
  table.dropTimestamps('created_at')
});
```

```typescript
knex.schema
  // Modify table
  .renameTable('persons', 'people')
  .dropTable('persons')
  // Check existance
  .hasTable('users').then(exists => ···)
  .hasColumn('users', 'id').then(exists => ···)
```
> See: [Knex - Schema builder](http://knexjs.org/#Schema)

### Timestamps helper

Knex doesn't provide a helper to automatically update `updated_at` field on a table. To accomplish this, we need to implement our own solution using Postgres [function](https://www.postgresql.org/docs/12/sql-createfunction.html) and [triggers](https://www.postgresql.org/docs/12/sql-createtrigger.html).

This solution is based on the following [StackOverflow response](https://stackoverflow.com/questions/36728899/knex-js-auto-update-trigger).

Step 1, we have to add a function responsible for updating the timestamp when any column in the table is updated. We need to do that in a migration before adding it to any table:

```typescript
// this_is_a_migration_to_add_function.ts
export async function up(knex: Knex): Promise<void> {
  await knex.raw(`
    CREATE OR REPLACE FUNCTION on_update_timestamp()
    RETURNS trigger AS $$
    BEGIN
      NEW.updated_at = now();
      RETURN NEW;
    END;
    $$ language 'plpgsql';
  `);
}

export async function down(knex: Knex): Promise<void> {
  await knex.raw('DROP FUNCTION on_update_timestamp');
}
```

> Important! If you look closely we are setting timestamp column as `updated_at`. If it needs to be called differently you need to change it.

I hope we don't need to write to this file anymore in the future but, in order to keep structure of the project. We need to create two functions, one to attach the trigger to our table and the second to revert this change.

```typescript
// triggers.ts
export function applyOnUpdateTrigger(table: string): string {
  return `
    CREATE TRIGGER ${table}_updated_at
    BEFORE UPDATE ON ${table}
    FOR EACH ROW
    EXECUTE PROCEDURE on_update_timestamp();
  `;
}

export function removeOnUpdateTrigger(table: string): string {
  return `DROP TRIGGER ${table}_updated_at`;
}
```

Finally, we need to implement the trigger in a migration. You can use this either when you create tha table or after that. Just remember to implement the migration to revert the change:

```typescript
// this_is_another_migration_to_create_banks_table.ts
import { applyOnUpdateTrigger, removeOnUpdateTrigger } from '../triggers';

export async function up(knex: Knex): Promise<void> {
  const tableExist = await knex.schema.hasTable('banks');
  if (tableExist) return;
  await knex.schema.createTable('banks', (table: Knex.CreateTableBuilder) => {
    table
      .uuid('id')
      .primary()
      .defaultTo(knex.raw('uuid_generate_v4()'));
    table.string('name');
    table.timestamp('created_at').defaultTo(knex.fn.now());
    table.timestamp('updated_at').defaultTo(knex.fn.now());
  });
  await knex.raw(applyOnUpdateTrigger('banks'));
}

export async function down(knex: Knex): Promise<void> {
  await knex.raw(removeOnUpdateTrigger('banks'));
  await knex.schema.dropTableIfExists('banks');
}
```