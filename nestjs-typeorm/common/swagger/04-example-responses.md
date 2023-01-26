# Adding Example Responses

The OpenAPI specification allows you to define example responses for each operation. This is useful for providing examples of the response body and headers. The examples are shown in the documentation and can be used to generate test cases.

The `NestJS/Swagger` provides us with `@ApiResponse` decorator that allows us to define example responses for each operation.

## Basic Usage

Let's take a look at a simple example, open any of the `*.controller.ts` files and add the following code:

```typescript board.controller.ts
import { ApiResponse } from '@nestjs/swagger';

@Controller('boards')
export class BoardsController {

  @ApiResponse({ status: 403, description: 'Forbidden.' })
  @Get()
  findAll(): Board[] {
    return this.boardsService.findAll();
  }
}
```

There is also a shorthand version of the `@ApiResponse` decorator that allows you to define the response status code and description:

```typescript board.controller.ts
import { ApiForbiddenResponse } from '@nestjs/swagger';

@Controller('boards')
export class BoardsController {

  @ApiForbiddenResponse({ description: 'Forbidden.' })
  @Get()
  findAll(): Board[] {
    return this.boardsService.findAll();
  }
}
```
