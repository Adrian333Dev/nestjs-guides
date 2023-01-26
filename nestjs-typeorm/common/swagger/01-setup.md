# Introducing the Swagger Module

In these next lessons we’ll be looking at how we can integrate and automatically generate OpenAPI documentation for our NestJS applications.

We’ll be taking advantage of all the latest tools, and Nest plugins to help automate and simplify every aspect of the process.

One of the best ways to document our application is to use the OpenAPI specification. The OpenAPI specification is a language-agnostic definition format used to describe **RESTful APIs**.

An OpenAPI document allows us to describe our entire API, including:

- Available operations (endpoints)
- Operation parameters: Input and output for each operation
- Authentication methods
- Contact information, license, terms of use and other information.
- … and much more ...

The OpenAPI specification is maintained by the [OpenAPI Initiative](https://www.openapis.org/), which is a consortium of the world’s leading API providers, including Google, IBM, Microsoft, and PayPal.

## Installing and Setting Up Swagger

- Install neccessary dependencies

```bash
npm install --save @nestjs/swagger swagger-ui-express
```

- Open `main.ts` and add the following code:

```typescript
import { NestFactory } from '@nestjs/core';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger'; 👈
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  /* ----------------- Swagger ----------------- */ 👇
  const options = new DocumentBuilder()
     .setTitle('PMA')
     .setDescription('Project Management Application')
     .setVersion('1.0')
     .build();
  const document = SwaggerModule.createDocument(app, options);
  SwaggerModule.setup('api', app, document); // 👈 Define the route for the Swagger UI
  /* ----------------- Swagger ----------------- */ 👆

  await app.listen(3000);
}
```

- To view the Swagger UI, navigate to `http://localhost:3000/api` in your browser.
