# Creating our first GraphQL Application

- Create new nest application

```bash
nest new graphql-coffee
```

- Install necessary dependencies

```bash
npm i @nestjs/graphql @nestjs/apollo graphql apollo-server-express
```

- Add GraphQLModule to the `app.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { GraphQLModule, ApolloDriverConfig } from '@nestjs/graphql';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver, // 👈 Because we are using Apollo set the driver to ApolloDriver
      /**
       * The autoSchemaFile option tells the module to generate a schema file
       * Also you can specify the path to the schema file, by default it will be generated in the memory
       */
      autoSchemaFile: join(process.cwd(), 'src/schema.gql'),
    }),
  ],
})
export class AppModule {}
```

- Enable GraphQL CLI plugin in `nest-cli.json`

```json
{
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "plugins": ["@nestjs/graphql"]
  }
}
```
