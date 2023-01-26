
# Diving Into Custom Providers

Nest provides a number of built-in providers, but sometimes you need to create your own. In this chapter, we'll learn how to create custom providers.

## Table of Contents

- [Diving Into Custom Providers](#diving-into-custom-providers)
  - [Table of Contents](#table-of-contents)
  - [Links](#links)
  - [Example advanced use-cases where we might need Custom Providers](#example-advanced-use-cases-where-we-might-need-custom-providers)
  - [List of Custom Providers](#list-of-custom-providers)
    - [1. Value Based Providers](#1-value-based-providers)
    - [2. Non-class-based Provider Tokens](#2-non-class-based-provider-tokens)
    - [3. Class Provider](#3-class-provider)
    - [4. Factory Provider](#4-factory-provider)
    - [5. Leverage Async Providers](#5-leverage-async-providers)
    - [6. Create a Dynamic Module](#6-create-a-dynamic-module)
    - [7. Control the Scope of a Provider](#7-control-the-scope-of-a-provider)

---

## Links

- [NestJS Course](https://learn.nestjs.com/courses/591712/lectures/18346941)
- [NestJS Docs](https://docs.nestjs.com/fundamentals/custom-providers)

---

## Example advanced use-cases where we might need Custom Providers

1. Creating a custom instance of our provider instead of having Nest instantiate the class for us
2. Or let’s say we want to reuse an existing class in a second dependency
3. How about if we want to override a class with a mock version for testing
4. And lastly, what if we want to use a **Strategy Pattern** in which we can provide an abstract class and interchange the real implementation (or actual class that is to be used) based on different conditions

---

## List of Custom Providers

### 1. Value Based Providers

This syntax is useful for providing constant values. Let's say we want to provide mock data in our **Boards Service** for testing purposes. We can do so by using the `useValue` syntax.

```typescript boards.module.ts

class MockBoardsService {} // 👈 class with mock data

@Module({
  imports: [TypeOrmModule.forFeature([Board, List])],
  controllers: [BoardsController],
  providers: [
    BoardsService,
    {
      provide: BoardsService, // 👈 provide the mock data
      useValue: new MockBoardsService(), // 👈 when we inject BoardsService, provider will return this mock data
    },
  ],
})
export class BoardsModule {}
```

---

### 2. Non-class-based Provider Tokens

Sometimes we might want to use a non-class-based provider token. For example, we might want to use a string or a symbol as a provider token. We can do so by using the `useValue` syntax.

```typescript boards.module.ts

const BOARDS_VISIBILITY = 'BOARDS_VISIBILITY'; // 👈 It's better practice to the token in separate file

@Module({
  imports: [TypeOrmModule.forFeature([Board, List])],
  controllers: [BoardsController],
  providers: [
    BoardsService,
    {
      provide: BOARDS_VISIBILITY, // 👈 non-class-based provider token
      useValue: ['public', 'private'], // 👈 when we inject 'BOARDS_VISIBILITY', provider will return this array
    },
  ],
})
export class BoardsModule {}
```

**Note:** When we inject a non-class-based provider token, we need to use the `@Inject` decorator.
  
  ```typescript boards.service.ts
   @Injectable()
    export class BoardsService {
      constructor(
        @Inject(BOARDS_VISIBILITY) private boardsVisibility: string[], // 👈 inject the non-class-based provider token
      ) {
        console.log(boardsVisibility); // 👈 ['public', 'private']
      }
    }
  ```

---

### 3. Class Provider

Another way to create a custom provider is by using `useClass` syntax. This syntax allows us to dynamically determine a class that a token should resolve to. Let's say we want to provide a different implementation of the **ConfigService** based on the environment. We can do so by using the `useClass` syntax.

```typescript boards.module.ts
class ConfigService {}
class DevConfigService {}
class ProdConfigService {}

@Module({
  // ...
  providers: [
    {
      provide: ConfigService, // 👈 provide the ConfigService
      useClass:
        process.env.NODE_ENV === 'development' // 👈 when we inject ConfigService, provider will return DevConfigService
          ? DevConfigService
          : ProdConfigService,
    },
  ],
})
export class BoardsModule {}
```

---

### 4. Factory Provider

The `useFactory` syntax allows us to create providers dynamically, which is useful for creating providers that depend on other providers.
Let's convert the `BOARDS_VISIBILITY` provider to a factory provider.

```typescript boards.module.ts
const BOARDS_VISIBILITY = 'BOARDS_VISIBILITY';

// 👇 create a provider for illustration purposes
@Injectable()
export class BoardsVisibilityFactory {
  create() {
    // ... some logic
    return ['public', 'private'];
  }
}

@Module({
  // ...
  providers: [
    BoardsVisibilityFactory,
    {
      provide: BOARDS_VISIBILITY,
      inject: [BoardsVisibilityFactory], // 👈 by injecting the BoardsVisibilityFactory, we can use it in the "useFactory" function
      useFactory: (boardsVisibilityFactory: BoardsVisibilityFactory) => boardsVisibilityFactory.create(),
    },
  ],
})
export class BoardsModule {}
```

---

### 5. Leverage Async Providers

Some times we might need to delay the whole application startup until a certain asynchronous operation is completed. For example, we might want to connect to a database before the application starts. We can achieve this by using combination of `async/await` and `useFactory` syntax.

In our example let's say that **BOARDS_VISIBILITY** provider's data has to be asynchronously queried from the database before it can be used anywhere in the application.

```typescript boards.module.ts
import { Connection } from 'typeorm';

@Module({
  // ...
  providers: [
    {
      provide: BOARDS_VISIBILITY,
      useFactory: async (connection: Connection) => { 👈 // use async/await
        // 👇 query the database
        const boardsVisibility = await Promise.resolve(['public', 'private']);
        return boardsVisibility;
      },
      inject: [Connection], // 👈 inject the Connection
    },
  ],
})
```

---

### 6. Create a Dynamic Module

Sometimes we might want to create a module that is not statically defined. For example, we might want to create a module that is only imported when a certain condition is met. We can achieve this by using the `DynamicModule` interface.

Let's create a `database` module that can be passed configuration settings before it instantiates.

- Generate a new module called `database` using the CLI.

  ```bash
  nest g module database
  ```

- Update the `database.module.ts` file.

  ```typescript database.module.ts
  import { DynamicModule, Module } from '@nestjs/common';
  import { DataSource } from 'typeorm';

  @Module({})
  export class DatabaseModule {
    static register(options: DataSourceOptions): DynamicModule {
      return {
        module: DatabaseModule,
        providers: [
          {
            provide: 'CONNECTION',
            useValue: new DataSource(options).initiate(),
          },
        ],
      };
    }
  }
  ```

- Go to the module that you want to import the `database` module and update the `imports` array.

  ```typescript boards.module.ts
  import { DatabaseModule } from './database/database.module';

  @Module({
    imports: [
      DatabaseModule.register({
        type: 'postgres',
        host: 'localhost',
        port: 5432,
        username: process.env.DATABASE_USER,
        password: process.env.DATABASE_PASSWORD,
      })
    ],
    // ...
  })
  // ...
  ```

---

### 7. Control the Scope of a Provider

In NestJS, providers are singletons by default. This means that the same instance of a provider is shared across the entire application. However, we might want to create a provider that is not a singleton. For example, we might want to create a provider that is only available to a specific module. We can achieve this by using the `scope` property.

```typescript boards.module.ts
@Module({
  // ...
  providers: [
    {
      provide: BOARDS_VISIBILITY,
      useValue: ['public', 'private'],
      scope: Scope.REQUEST, // 👈 set the scope to "request"
    },
  ],
})
```

OR

```typescript boards.module.ts
@Injectable({ scope: Scope.REQUEST }) // 👈 set the scope to "request"
export class BoardsService {}
```

**Note:** Other available scopes are `Scope.TRANSIENT` and `Scope.DEFAULT`.
