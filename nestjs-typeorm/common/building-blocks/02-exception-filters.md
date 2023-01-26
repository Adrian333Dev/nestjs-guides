# Exception filters

Exception filters are responsible for handling and processing unhanded exceptions that might occur in the application.

## Links

* [NestJS Docs](https://docs.nestjs.com/exception-filters#exception-filters)

## Example

Let's create an Exception Filter that is resposible for catching exceptions that are an instance of `HttpException` Class, and implementing a custom response for them.

* Generate a new filter class using the CLI:

```bash
nest g filter shared/filters/http-exception
```

* Open the generated file `src/shared/filters/http-exception.filter.ts` and add the following code:

```typescript
import { Catch, HttpException, ExceptionFilter, ArgumentsHost } from "@nestjs/common";
import { Response } from 'express';

@Catch(HttpException) // 👈 Catches any HttpException thrown in the application

/* 
  👇 This class implements the ExceptionFilter interface and is used to handle HttpExceptions thrown in the application.
*/
export class HttpExceptionFilter<T extends HttpException> implements ExceptionFilter {

  // 👇 This method is called when an HttpException is thrown
  catch(exception: T, host: ArgumentsHost) {

    // 👇 Switch to the http context and get the response object
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    // 👆

    // 👇 Get the status code from the exception
    const status = exception.getStatus();

    // 👇 Get the response from the exception
    const exceptionResponse = exception.getResponse();

    /* 
    1. Check if the response is a string or an object
    2. If it's a string, create an error object with the message
    3. If it's an object, use it as the error object 👇 
    */
    const error =
      typeof response === 'string'
        ? { message: exceptionResponse }
        : (exceptionResponse as object);
    // 👆
    
    // 👇 Send the error as a JSON response with the status code and a timestamp
    response.status(status).json({
      ...error,
      timestamp: new Date().toISOString(),
    });
  }
}
```

* Lastly, we need to bind this `ExceptionFilter` to the application globally. Open the `src/main.ts` file and add the following code:

```typescript
import { HttpExceptionFilter } from './shared/filters/http-exception.filter';

async function bootstrap() {
  // ...
  app.useGlobalFilters(new HttpExceptionFilter()); // 👈 Binds the HttpExceptionFilter to the application
  // ...
}
