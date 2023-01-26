# Using Tags to Group Resources

Tags are a way to group resources together. For example, you might have a set of resources that are all related to a particular entity in your application.  You can use tags to group these resources together.

## Basic Usage

Let's take a look at a simple example, open any of the `*.controller.ts` files and add the following code:

```typescript board.controller.ts
import { ApiTags } from '@nestjs/swagger'; // 👈 import the decorator

@ApiTags('boards') // 👈 add the decorator
@Controller('boards')
export class BoardsController {
  constructor(private readonly boardsService: BoardsService) {}

  @Get()
  findAll(): Board[] {
    return this.boardsService.findAll();
  }
}
```

**Note:** You can also group methods together by using the `@ApiTags` decorator on the method.w
