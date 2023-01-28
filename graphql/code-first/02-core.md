# Diving into GraphQL

## Table of Contents

- [Diving into GraphQL](#diving-into-graphql)
  - [Table of Contents](#table-of-contents)
    - [Resolvers and Object Types](#resolvers-and-object-types)
    - [GraphQL Schemas, Types, and Scalars](#graphql-schemas-types-and-scalars)
  - [⬅️ Previous | ➡️ Next](#️-previous--️-next)

---

### Resolvers and Object Types

- Generate coffee entity: `nest g class coffees/entities/coffee.entity --no-spec`

- Update `coffee.entity.ts` as follows:

  ```ts
  import { ObjectType } from '@nestjs/graphql';

  @ObjectType() // 👈
  export class Coffee { // 👈 make sure Entity is removed
  id: number;
  name: string;
  brand: string;
  flavors: string[];
  }
  ```

- Generate coffees module: `nest g module`
- Generate coffees resolver: `nest g resolver coffees`

- Update `coffees.resolver.ts` as follows:

  ```ts
  import { Resolver } from '@nestjs/graphql';

  @Resolver()
  export class CoffeesResolver {
    @Query(() => [Coffee], { name: 'coffees' }) // 👈
    async findAll() {
      return [];
    }
  }
  ```

- To start the server run `npm run start` and navigate to `http://localhost:3000/graphql`

- To query the server run the following query in the postman, insomnia or graphql playground:

  ```ts
  // ------
  // 💡 GraphQL Query BREAKDOWN
  // If we look at the query we’ve written here:
  { // 👈 we start off with a special “root” object.
    coffees { // 👈 We’re then selecting the “coffees” field on that
      // ☝️ Next, when it comes to the object that gets returned by “coffees”, we’re selecting:
      id // 👈
      name // 👈
      brand // 👈
      flavors // 👈
    }
  }
  ```

---

### GraphQL Schemas, Types, and Scalars

You can take a look at the auto-generated schema by going to `src/schema.gql`

---

## [⬅️ Previous](../01-graphql/README.md) | [➡️ Next](../03-graphql-queries/README.md)
