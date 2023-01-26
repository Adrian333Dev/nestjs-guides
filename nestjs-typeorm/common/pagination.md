# Implementing Pagination

- Create a `Dto` for pagination

```typescript pagination.dto.ts
import { IsOptional, IsPositive } from 'class-validator';

export class PaginationQueryDto {
  @IsOptional()
  @IsPositive()
  limit: number;

  @IsOptional()
  @IsPositive()
  offset: number;
}
```

- In `main.ts` update the `ValidationPipe`

```typescript main.ts
/* main.ts - useGlobalPipes addition */
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
    transform: true,
    forbidNonWhitelisted: true,
    transformOptions: {
      enableImplicitConversion: true, // 👈 Convert query parameters to the correct type
    },
  }),
);
```

- In `boards.service.ts` update the `getAllBoards` method

```typescript boards.service.ts
import { PaginationQueryDto } from './dto/pagination.dto';

@Injectable()
export class BoardsService {
  async getAllBoards(paginationQuery: PaginationQueryDto): Promise<Board[]> {
    const { limit, offset } = paginationQuery;
    return this.boardRepository.find({
      take: limit, // 👈 number of records to return
      skip: offset, // 👈 number of records to skip
    });
  }
}

```

- In `boards.controller.ts` use `@Query()` to access query parameters and pass them to the service

```typescript boards.controller.ts
import { /* ... */ Query } from '@nestjs/common';
import { PaginationQueryDto } from './dto/pagination.dto';

@Controller('boards')
export class BoardsController {

  constructor(
    private readonly boardsService: BoardsService, // Don't forget inject the service
  ) {}

  @Get()
  getAll(@Query() paginationQuery: PaginationQueryDto) {
    return this.boardsService.getAllBoards(paginationQuery);
  }
}
```
