# Interceptors

Interceptors have a set of useful capabilities which are inspired by the [Aspect Oriented Programming](https://en.wikipedia.org/wiki/Aspect-oriented_programming) (AOP) technique. They make it possible to:

- bind extra logic before / after method execution
- transform the result returned from a function
- transform the exception thrown from a function
- extend the basic function behavior
- completely override a function depending on specific conditions (e.g., for caching purposes)

## Example

As an example let's create a interceptor that will wrap our response in `data` property.

- Generate a new interceptor

```bash
nest g interceptor common/interceptors/wrap-response
```

- Open the generated file `src/common/interceptors/wrap-response.interceptor.ts` and modify it as follows:

```typescript
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class WrapResponseInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before...');

    return next.handle().pipe(map(data => ({ data })));
  }
}
```

- Finally register the interceptor globally in the `src/main.ts` file:

```typescript
import { WrapResponseInterceptor } from './common/interceptors/wrap-response.interceptor';

async function bootstrap() {
  // ...
  app.useGlobalInterceptors(new WrapResponseInterceptor());
  // ...
}
```

## Handling Timeouts with Interceptors

Another technique useful for Interceptors is to **extend** the basic function behavior by applying RxJS operators to the response stream.

To help us learn about this concept by example - let’s imagine that we need to handle **timeouts** for all of our route requests.

When an endpoint does not return anything after a certain period of time, we need to terminate the request, and send back an error message.

---

Let's create a new interceptor that will handle timeouts for all of our requests, When an endpoint does not return anything after a certain period of time, we need to terminate the request, and send back an error message.

- Generate a new interceptor

```bash
nest g interceptor common/interceptors/timeout
```

- Open the generated file `src/common/interceptors/timeout.interceptor.ts` and modify it as follows:

```typescript
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
  RequestTimeoutException,
} from '@nestjs/common';
import { Observable, throwError, TimeoutError } from 'rxjs';
import { catchError, timeout } from 'rxjs/operators';

@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(3000),
      catchError(err => {
        if (err instanceof TimeoutError) return throwError(new RequestTimeoutException());
        return throwError(err);
      }),
    );
  }
}
```

- To simulate a timeout, we can use the `setTimeout()` function in our controller:

```typescript board.controller.ts
// ...
@Get()
findAll() {
  await new Promise(resolve => setTimeout(resolve, 5000));
  return this.boardService.findAll();
}
```
