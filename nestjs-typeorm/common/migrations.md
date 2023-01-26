# Setting up Migrations

Migrations are a way to manage changes to your database schema over time. They are a way to version control your database schema. Migrations are a way to keep your database schema in sync with your application code.

**Example:** As an example usecase, let's say we want to rename the `title` column to `name` in the `Task` entity. We will create a migration file for this.

## Steps to setup migrations

- Create `typeorm-cli.config.ts` file in the root of your project.
- Add the following code to the file:

```typescript
import { DataSource } from 'typeorm';

export default new DataSource({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'pass123',
  database: 'postgres',
  entities: [],
  migrations: [],
});
```

- Change the column names in the `Task` entity to `name` instead of `title`.

```typescript
@Entity()
export class Task {

  @Column()
  name: string; // Change from title to name
}
```

- Create a TypeORM migration by running the following command:

```bash
npx typeorm migration:create src/migrations/<migration-name>
```

- Go to the generated migration file and update the `up` and `down` methods as follows:

```typescript
import { MigrationInterface, QueryRunner } from 'typeorm';

export class <migration-name> implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`ALTER TABLE "task" RENAME COLUMN "title" TO "name"`);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`ALTER TABLE "task" RENAME COLUMN "name" TO "title"`);
  }
}
```

- Open `typeorm-cli.config.ts` file and add the Generated migration file to the `migrations` array.

```typescript
import { DataSource } from 'typeorm';

export default new DataSource({
  // ...
  migrations: [<migration-name>], // Example: [TaskRefactor126183]
});
```

- Build the project by running the following command:

```bash
npm run build
```

- Run the migration by running the following command:

```bash
npx typeorm migration:run -d dist/typeorm-cli.config.js
```

- **If you want to revert the changes back, run the following command:**

```bash
npx typeorm migration:revert -d dist/typeorm-cli.config.js
```

**Note:** Don't forget to change the `title` column name back to `name` in the `Task` entity.

---

## Another way to setup migrations

TypeORM provides a way to setup migrations automaticly using the CLI.

- Add the entities you want to track to the `entities` array in the `typeorm-cli.config.ts` file.

```typescript
export default new DataSource({
  // ...
  entities: [Board, Task] // Example
  // ...
});
```

- Run the following command to generate a migration file:

```bash
npx typeorm migration:generate src/migrations/<migration-name> -d dist/typeorm-cli.config
```

- Now open `typeorm-cli.config.ts` file and add the Generated migration file to the `migrations` array.

```typescript
export default new DataSource({
  // ...
  migrations: [<migration-name>], // Example: [SchemaSync162183]
});
```

- Run the migration by running the following command:

```bash
npx typeorm migration:run -d dist/typeorm-cli.config.js
```

---

**Links:**

- [NestJS Course](https://learn.nestjs.com/courses/591712/lectures/23241322)
