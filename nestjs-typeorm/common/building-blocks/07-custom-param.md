# Custom Param Decorators

Decorators are simply functions that apply logic, NestJS provides a few decorators out of the box that you can use together with the HTTP route handlers to extract information from the request object. Additionally, you can create your own custom decorators.

## Example

Let's create a custom decorator that retrieves `request.protocol` from the request object.

- Create a new file `src/shared/decorators/protocol.decorator.ts` and add the following code:

```typescript
import {
  createParamDecorator,
  ExecutionContext,
} from '@nestjs/common';

export const Protocol = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.protocol;
  },
);
```

- To test the decorator, add it to the endpoint handler:

```typescript boards.controller.ts
// ...
@Get()
findAll(@Protocol() protocol: string) {
  console.log(protocol);
  return this.boardsService.findAll();
}
// ...
```

**NOTE:** You can also pass arguments to the decorator, which will be passed to the function that creates the decorator:

```typescript boards.controller.ts
@Get()
findAll(@Protocol('https') protocol: string) { // 👈
  console.log(protocol);
  return this.boardsService.findAll();
}
```

```typescript protocol.decorator.ts
export const Protocol = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => { // 👈 data: 'https'
    const request = ctx.switchToHttp().getRequest();
    return request.protocol === data ? 'secure' : 'insecure'; // 👈
  },
);
```
