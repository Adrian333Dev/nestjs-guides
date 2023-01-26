# Using Metadata to Build Generic Guards or Interceptors

NestJS provides the ability to attach custom metadata to route handlers through the `@SetMetadata()` decorator.
The `@SetMetadata()` decorator takes two arguments: the metadata **key** and the metadata **value**.

Let's create a `public` metadata key that will be used to mark routes as public.

- Go to the controller where you want to mark a route as public, as an example, let's use the `app.controller.ts` file.

```typescript app.controller.ts
import { Controller, Get, SetMetadata } from "@nestjs/common";

@Controller()
export class AppController {
  @Get()
  getHello(): string {
    return "Hello World!";
  }

  @SetMetadata("isPublic", true)
  @Get("public")
  getPublic(): string {
    return "This is a public route";
  }
}
```

**Note**: However the example above is not good practice, Ideally, you should create your own custom decorator for this. So let's create a `@Public()` decorator.

- Create a new file `src/shared/decorators/public.decorator.ts` and add the following code:

```typescript public.decorator.ts
import { SetMetadata } from "@nestjs/common";

export const IS_PUBLIC_KEY = 'isPublic';

export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);
```

- Now, let's use the `@Public()` decorator in the `app.controller.ts` file:

```typescript app.controller.ts
import { Public } from "./shared/decorators/public.decorator";

@Controller()
export class AppController {
  // ...

  @Public()
  @Get("public")
  getPublic(): string {
    return "This is a public route";
  }
}
```
