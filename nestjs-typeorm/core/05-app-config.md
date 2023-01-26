# Application Configuration

## Table of Contents

- [Application Configuration](#application-configuration)
  - [Table of Contents](#table-of-contents)
  - [Links](#links)
  - [Introducing the Config Module](#introducing-the-config-module)
  - [Custom Environment File Paths](#custom-environment-file-paths)
  - [Schema Validation](#schema-validation)
  - [Using the Config Service](#using-the-config-service)
  - [Custom Configuration Files](#custom-configuration-files)
  - [Configuration Namespaces and Partial Registration](#configuration-namespaces-and-partial-registration)
  - [Asynchronously Configure Dynamic Modules](#asynchronously-configure-dynamic-modules)

## Links

- [NestJS Course](https://learn.nestjs.com/courses/591712/lectures/23244110)
- [NestJS Docs](https://docs.nestjs.com/techniques/configuration)

---

## Introducing the Config Module

It’s a common best practice in the Node.js community to store these configuration variables as a part of the environment - in the Node.js global process.env object.
NestJS Config module helps make working with these environment variables, even simpler.

- Install the `@nestjs/config` package from npm

  ```bash
  npm i @nestjs/config
  ```

- Import the `ConfigModule` into the root module of our application

  ```typescript
  import { Module } from '@nestjs/common';
  import { ConfigModule } from '@nestjs/config';

  @Module({
    imports: [
      ConfigModule.forRoot(),
      // ...
    ],
  })
  export class AppModule {}
  ```

- Create a `.env` file in the root of our project and add the following variables

  ```env
  DATABASE_USER=postgres
  DATABASE_PASSWORD=pass123
  DATABASE_NAME=postgres
  DATABASE_PORT=5432
  DATABASE_HOST=localhost
  ```

**Notes:**

1. You need to change the values of the variables to match your database configuration.
2. Don’t forget to add the `*.env` to your `.gitignore` file. We don’t want to commit our environment variables to our repository.

- Now update `TypeOrmModule` to use the environment variables

  ```typescript
  import { Module } from '@nestjs/common';
  import { TypeOrmModule } from '@nestjs/typeorm';
  import { ConfigModule } from '@nestjs/config';

  @Module({
    imports: [
      ConfigModule.forRoot(),
      TypeOrmModule.forRoot({
        type: 'postgres',
        host: process.env.DATABASE_HOST,
        port: parseInt(process.env.DATABASE_PORT),
        username: process.env.DATABASE_USER,
        password: process.env.DATABASE_PASSWORD,
        database: process.env.DATABASE_NAME,
        autoLoadEntities: true,
        synchronize: true,
      }),
      // ...
    ],
  })
  export class AppModule {}
  ```
  
- Run the application and check that it works as expected

  ```bash
  npm run start:dev
  ```

---

## Custom Environment File Paths

- By default, the `ConfigModule` will look for a `.env` file in the root of our project. We can change this by passing a `path` property to the `forRoot()` method.

  ```typescript
  import { Module } from '@nestjs/common';
  import { TypeOrmModule } from '@nestjs/typeorm';
  import { ConfigModule } from '@nestjs/config';

  @Module({
    imports: [
      ConfigModule.forRoot({
        path: '.enviroment' // 👈 It will look for a .enviroment file in the root of our project instead of .env
      }),
      // ...
    ],
  })
  export class AppModule {}
  ```

- Also we can ignore the `.env` file by passing a `ignoreEnvFile` property to the `forRoot()` method. This is useful when we using **Provider UI's** such as [Heroku](https://www.heroku.com/), [Railway](https://railway.app/), etc.

  ```typescript
  @Module({
    imports: [
      ConfigModule.forRoot({
        ignoreEnvFile: true // 👈 It will ignore the .env file
      }),
      // ...
    ],
  })
  export class AppModule {}
  ```

---

## Schema Validation

The `nestjs/config` package, let's us take advantage of the `Joi` package to make sure any important environment variables are validated before our application starts.

- Install necessary packages

  ```bash
  npm install @hapi/joi
  npm install --save-dev @types/hapi__joi
  ```

- Update the `ConfigModule` to use the `validationSchema` property

  ```typescript
  import { Module } from '@nestjs/common';
  import { TypeOrmModule } from '@nestjs/typeorm';
  import { ConfigModule } from '@nestjs/config';
  import * as Joi from '@hapi/joi';

  @Module({
    imports: [
      ConfigModule.forRoot({
        validationSchema: Joi.object({
          DATABASE_USER: Joi.string().required(),
          DATABASE_PASSWORD: Joi.string().required(),
          DATABASE_NAME: Joi.string().required(),
          DATABASE_PORT: Joi.number().required().default(5432),
          DATABASE_HOST: Joi.string().required(),
        }),
      }),
      // ...
    ],
  })
  export class AppModule {}
  ```

- Now if we try to run our application without the environment variables, we will get an error.

---

## Using the Config Service

- We can inject the `ConfigService` into any of our providers and use it to access our environment variables.

  ```typescript
  import { Injectable } from '@nestjs/common';
  import { ConfigService } from '@nestjs/config';

  @Injectable()
  export class AppService {
    constructor(private configService: ConfigService) {}

    getHello(): string {
      const databaseHost = this.configService.get('DATABASE_HOST');
      return `Hello World! ${databaseHost}`;
    }
  }
  ```

---

## Custom Configuration Files

- We can also use the `ConfigModule` to load custom configuration files. For example, we can create a `config` folder in the root of our project and add a `app.config.ts` file.

  ```typescript config/app.config.ts
  export default () => ({
    environment: process.env.NODE_ENV || 'development',
    database: {
      host: process.env.DATABASE_HOST,
      port: parseInt(process.env.DATABASE_PORT, 10) || 5432
    }
  });
  ```

- Now we can import the `ConfigModule` and pass the `load` property to the `forRoot()` method.

  ```typescript
  import { Module } from '@nestjs/common';
  import { TypeOrmModule } from '@nestjs/typeorm';
  import { ConfigModule } from '@nestjs/config';
  import appConfig from './config/app.config';

  @Module({
    imports: [
      ConfigModule.forRoot({
        load: [appConfig],
      }),
      // ...
    ],
  })
  export class AppModule {}
  ```

- Now we can access the configuration values using the `ConfigService`

  ```typescript
  import { Injectable } from '@nestjs/common';
  import { ConfigService } from '@nestjs/config';

  @Injectable()
  export class AppService {
    constructor(private configService: ConfigService) {}

    getHello(): string {
      const databaseHost = this.configService.get('database.host');
      return `Hello World! ${databaseHost}`;
    }
  }
  ```

---

## Configuration Namespaces and Partial Registration

The `ConfigModule` allows us to register multiple configuration files and access them using namespaces.

- As an exmaple let's create a configuration file for **Board** module.

  ```typescript config/board.config.ts
  export default registerAs('board', () => ({
    maxColumns: process.env.BOARD_MAX_COLUMNS,
  }));
  ```

- Now open the `board.module.ts` file and import the `ConfigModule` and pass the newly created configuration file to the `forFeature()` method of the `ConfigModule`.

  ```typescript
  import { Module } from '@nestjs/common';
  import { TypeOrmModule } from '@nestjs/typeorm';
  import { ConfigModule } from '@nestjs/config';
  import boardConfig from '../config/board.config';

  @Module({
    imports: [
      ConfigModule.forFeature(boardConfig),
      // ...
    ],
  })
  export class BoardModule {}
  ```

- Now we can access the configuration values using the `ConfigService`

  ```typescript board.service.ts
  import { Injectable } from '@nestjs/common';
  import { ConfigService } from '@nestjs/config';

  @Injectable()
  export class BoardService {
    constructor(private configService: ConfigService) {}

    getBoardMaxColumns(): number {
      return this.configService.get('board.maxColumns');
    }
  }
  ```

**Note:** In this approach we don't have full type safety, and also it makes testing harder. So it's better to inject entire namespaces configuration objects directly into the providers.

  ```typescript board.service.ts
  import { Injectable } from '@nestjs/common';
  import { ConfigService } from '@nestjs/config';

  @Injectable()
  export class BoardService {
    constructor(
      @Inject(boardConfig.KEY)
      private boardConfig: ConfigType<typeof boardConfig>,
    ) {}
  }
  ```

---

## Asynchronously Configure Dynamic Modules

TypeORM requires us to use the `forRootAsync()` method to configure the `TypeOrmModule` asynchronously.

- Open the `app.module.ts` file and update the `TypeOrmModule` to use the `forRootAsync()` method.

  ```typescript
  import { Module } from '@nestjs/common';
  import { TypeOrmModule } from '@nestjs/typeorm';
  import { ConfigModule } from '@nestjs/config';
  import appConfig from './config/app.config';

  @Module({
    imports: [
      // ...
      TypeOrmModule.forRootAsync({
        useFactory: () => ({
          type: 'postgres',
          host: process.env.DATABASE_HOST,
          port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
          username: process.env.DATABASE_USER,
          password: process.env.DATABASE_PASSWORD,
          database: process.env.DATABASE_NAME,
          autoLoadEntities: true,
          synchronize: true,
        }),
      }),
      // ...
    ],
  })
  export class AppModule {}
  ```
