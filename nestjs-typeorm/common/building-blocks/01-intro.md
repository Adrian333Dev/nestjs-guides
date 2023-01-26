# Building Blocks

**TODO:** This page is messed up, need to be completely rewritten.

## Introduction

In NestJS, we have 4 additional building blocks (for features) that we can use to extend the functionality of our application. These are:

- [Exception filters](./02-exception-filters.md)
- [Pipes](./03-pipes.md)
- [Guards](./04-guards.md)
- [Interceptors](./05-interceptors.md)

## Links

- [NestJS Docs](https://docs.nestjs.com)
- [NestJS Course](https://learn.nestjs.com/courses/591712/lectures/23246759)

---

## Binding Techniques

**Nest building blocks can be:**

1. **Globally**-scoped,
2. **Controller**-scoped,
3. **Method**-scoped,
4. And (the bonus 4th one) **Param-scoped** which as we said, is available to Pipes only.

**Note:** If you use more than one of the binding techniques, they're not going to override each other. Instead, they're going to be executed in the order they're defined.

- We can use the `app.useGlobalFilters()`, `app.useGlobalPipes()`, `app.useGlobalGuards()`, and `app.useGlobalInterceptors()` methods to bind a building block globally. In validation section, we've already seen how to use `app.useGlobalPipes()`.
- One big limitation of this approach is that we can't use dependency injection in our building blocks. This is because we setting it up outside of the Nest's DI container. One option we have is to set up a Pipe directly from inside a Nest module using custom provider.

  - Open the `AppModule` and add a new custom provider:

    ```ts
    import { Module, ValidationPipe } from '@nestjs/common';
    import { APP_PIPE } from '@nestjs/core';

    @Module({
      // ...
      providers: [ 
        {
          provide: APP_PIPE,
          useClass: ValidationPipe,
        },
        // ...
      ],
    })
    export class AppModule {}
    ```

- In the example above, we're injected the `ValidationPipe` into the `APP_PIPE` token. This way, we can use the `ValidationPipe` in all of our controllers, But what if we want to use a different Pipe in a specific controller? We can use the `@UsePipes()` decorator to override the global Pipe.

  - Open the `BoardsController` and add the `@UsePipes()` decorator:

    ```ts boards.controller.ts
    import { Controller, Get, Post, Body, UsePipes, ValidationPipe } from '@nestjs/common';
    import { CreateBoardDto } from './dto/create-board.dto';
    import { BoardsService } from './boards.service';
    import { Board } from './board.entity';

    @UsePipes(ValidationPipe) // 👈
    @Controller('boards')
    export class BoardsController {
      constructor(private boardsService: BoardsService) {}

      @Get()
      getAllBoards(): Promise<Board[]> {
        return this.boardsService.getAllBoards();
      }

      @Post()
      createBoard(@Body() createBoardDto: CreateBoardDto): Promise<Board> {
        return this.boardsService.createBoard(createBoardDto);
      }
    }
    ```

  - Now, the `ValidationPipe` will only be applied to the `BoardsController` and not to all of our controllers.

- Anoter option is to use the `@UsePipes()` decorator only for specific endpoint:
  
  ```ts boards.controller.ts

  @Controller('boards')
  export class BoardsController {
    constructor(private boardsService: BoardsService) {}

    @Get()
    getAllBoards(): Promise<Board[]> {
      return this.boardsService.getAllBoards();
    }

    @Post()
    @UsePipes(ValidationPipe) // 👈 only for this endpoint
    createBoard(@Body() createBoardDto: CreateBoardDto): Promise<Board> {
      return this.boardsService.createBoard(createBoardDto);
    }
  }
  ```

- And the 4th way which is only available to Pipes is to use the `@UsePipes()` decorator on a method parameter. This way, we can apply a Pipe to a specific parameter of a method.

  - Open the `BoardsController` and add the `@UsePipes()` decorator to the `createBoard()` method:

    ```ts boards.controller.ts
    import { Controller, Get, Post, Body, UsePipes, ValidationPipe } from '@nestjs/common';
    import { CreateBoardDto } from './dto/create-board.dto';
    import { BoardsService } from './boards.service';
    import { Board } from './board.entity';

    @Controller('boards')
    export class BoardsController {
      constructor(private boardsService: BoardsService) {}

      @Get()
      getAllBoards(): Promise<Board[]> {
        return this.boardsService.getAllBoards();
      }

      @Post()
      createBoard(
        @Body() createBoardDto: CreateBoardDto, // 👈
        @Body('title', ValidationPipe) title: string, // 👈 
      ): Promise<Board> {
        return this.boardsService.createBoard(createBoardDto);
      }
    }
    ```

  - Now, the `ValidationPipe` will only be applied to the `title` parameter of the `createBoard()` method.
