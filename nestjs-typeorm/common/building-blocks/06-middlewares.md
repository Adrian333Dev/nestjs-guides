# Middlewares

Middleware functions have access to the request and response objects, and are not specifically tied to any method, but rather to a specified route **PATH**.

**Middleware functions can perform the following tasks:**

- executing code
- making changes to the request and the response objects.
- ending the request-response cycle.
- Or even calling the **next** middleware function in the call stack.

When working with middleware, if the current middleware function does not END the request-response cycle, it must call the `next()` method, which passes control to the next middleware function.

Otherwise, the request will be left hanging - and never complete.

## Example

Let's create a middleware whic just logs to the console.

- Generate a new middleware

```bash
nest g middleware common/middleware/logging
```

- Open the generated file `src/common/middleware/logging.middleware.ts` and replace the content with the following:

```typescript
import {
  Injectable,
  NestMiddleware,
} from '@nestjs/common';

@Injectable()
export class LoggingMiddleware implements NestMiddleware {
  use(req: any, res: any, next: () => void) {
    console.time('Request-response time');
    console.log('Hi from middleware!');
    
    res.on('finish', () => console.timeEnd('Request-response time'));
    next(); 
  }
}
```

- Go to the module where you want to register the middleware, in our case `shared.module.ts` and add the following:

```typescript
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggingMiddleware } from './middleware/logging.middleware';

@Module({
  // ...
})
export class SharedModule implements NestModule { // 👈 Make sure to implement the NestModule interface
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggingMiddleware)
      .forRoutes('*');
  }
}
```
