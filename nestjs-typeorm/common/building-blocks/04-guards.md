# Guards

Guards have a single responsibility. They determine whether a given request will be handled by the route handler or not, depending on certain conditions (like permissions, roles, ACLs, etc.) present at run-time. This is often referred to as authorization.

## Links

- [NestJS Docs](https://docs.nestjs.com/guards#guards)

## 1. Example

Let's create a Guard that is responsible for checking if the user is authenticated or not. If the user is authenticated, the request will be handled by the route handler. If not, the request will be rejected.

- Generate a new guard class using the CLI:

```bash
nest g guard shared/guards/authenticated
```

- Open the generated file `src/shared/guards/authenticated.guard.ts` and add the following code:

```typescript
import { CanActivate, ExecutionContext, Injectable } from "@nestjs/common";

@Injectable()
export class AuthenticatedGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return request.isAuthenticated();
  }
}
```

- Open the file `src/app.module.ts` and add the following code:

```typescript
import { Module } from "@nestjs/common";
import { AuthenticatedGuard } from "./shared/guards/authenticated.guard";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService, AuthenticatedGuard],
})
export class AppModule {}
```

- Open the file `src/app.controller.ts` and add the following code:

```typescript
import { Controller, Get, UseGuards } from "@nestjs/common";
import { AuthenticatedGuard } from "./shared/guards/authenticated.guard";
import { AppService } from "./app.service";

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }

  @UseGuards(AuthenticatedGuard)
  @Get("protected")
  getProtected(): string {
    return "This is a protected route";
  }
}
```

## 2. Example

Let's create an API_KEY guard that is responsible for two things:

- Validated whether an API_KEY is present within an "Authorization" header.
- Whether the route being accessed is specified as "public".

- Generate a new guard class using the CLI:

```bash
nest g guard shared/guards/api-key
```

- Apply ApiKeyGuard globally in `main.ts`:

```typescript
import { ApiKeyGuard } from "./shared/guards/api-key.guard";

async function bootstrap() {
  // ...
  app.useGlobalGuards(new ApiKeyGuard()); // 👈 Add this line
  // ...
}
```

- Open the generated file `src/shared/guards/api-key.guard.ts` and add the following code:

```typescript
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { Reflector } from '@nestjs/core';
import { Request } from 'express';
import { Observable } from 'rxjs';
import { IS_PUBLIC_KEY } from '../decorators/public.decorator'; // 👈 Import the custom metadata

@Injectable()
export class ApiKeyGuard implements CanActivate {
  constructor(
    private readonly reflector: Reflector,
    private readonly configService: ConfigService,
  ) {}

  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    // 👇 In order to check if the route is public or not, we need to use the Reflector class, which is a built-in NestJS class that allows us to access custom metadata.
    const isPublic = this.reflector.get(IS_PUBLIC_KEY, context.getHandler());
    /* 
    NOTE: In the line above, we are using custom metadata to check if the route is public or not, to learn more about custom metadata, check out the "Custom Metadata" section in the "Decorators" building block.
    */
    if (isPublic) return true; // 👈 If the route is public return true without checking for the API_KEY
    
    // 👇 If the route is not public, check for the API_KEY
    const request = context.switchToHttp().getRequest<Request>(); 
    const authHeader = request.header('Authorization');
    return authHeader === this.configService.get('API_KEY');
  }
}
```

**Note:**

- The `process.env.API_KEY` is a placeholder for the actual API_KEY. You can use a `.env` file to store the API_KEY.
- Check [Custom Metadata](./custom-metadata.md) to learn more about custom metadata.
- Because we using dependency injection inside our guard, application will throw an error. To fix this, we need to add the `ApiKeyGuard` to the module providers. Open the module you want to add the guard, in this case `shared.module.ts` and add the following code:

```typescript src/shared/shared.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { APP_GUARD } from '@nestjs/core';
import { ApiKeyGuard } from './guards/api-key.guard';

@Module({
  // ...
  imports: [ConfigModule],
  providers: [{ provide: APP_GUARD, useClass: ApiKeyGuard }], // 👈 Add this line
  // ...
})
export class SharedModule {}
```

- Now opwn the `main.ts` file and remove the `app.useGlobalGuards(new ApiKeyGuard());` line, since we already added the guard to the module providers.

```typescript src/main.ts
async function bootstrap() {
  // ...
  // app.useGlobalGuards(new ApiKeyGuard()); // 👈 Remove this line
  // ...
}
```
