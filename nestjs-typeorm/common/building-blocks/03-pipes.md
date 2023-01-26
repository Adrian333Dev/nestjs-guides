# Pipes

**Pipes have two typical use cases:**

- **transformation:** transform input data to the desired form (e.g., from string to integer)
- **validation:** evaluate input data and if valid, simply pass it through unchanged; otherwise, throw an exception

In both cases, pipes operate on the arguments being processed by a controller’s route handler.

NestJS triggers a pipe just before a method is invoked.

Pipes also receive the arguments meant to be passed on to the method. Any transformation or validation operation takes place at this time - afterwards the route handler is invoked with any (potentially) transformed arguments.

## Links

- [NestJS Docs](https://docs.nestjs.com/pipes#pipes)

## Example

Let's create a pipe that automatically parses any incoming String to an Integer for learning purposes.

- Generate a new pipe

```bash
nest g pipe shared/pipes/parse-int
```

- Open the generated file `src/shared/pipes/parse-int.pipe.ts` and replace the content with the following:

```typescript
import {
  ArgumentMetadata,
  BadRequestException,
  Injectable,
  PipeTransform,
} from '@nestjs/common';

@Injectable()
export class ParseIntPipe implements PipeTransform    {
  transform(value: string, metadata: ArgumentMetadata) {
    const val = parseInt(value, 10);
    if (isNaN(val)) {
      throw new BadRequestException(
        `Validation failed. "${val}" is not an integer.`,
      );
    }
    return val;
  }
}
```

- Now open go to the endpoint you want to use this pipe and add it to the `@Param()` decorator, in this case let's use it for the `id` parameter:

```typescript board.controller.ts
import { ParseIntPipe } from '../shared/pipes/parse-int.pipe';

@Controller('boards')
export class BoardsController {
  // ... other code
  @Get(':id')
  getBoardById(@Param('id', ParseIntPipe) id: number): Board {
    return this.boardsService.getBoardById(id);
  }
}
```
