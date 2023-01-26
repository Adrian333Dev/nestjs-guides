# Data Validation

**Useful links**:

- [Validation](https://docs.nestjs.com/techniques/validation)

---

## Validation Pipe

The ValidationPipe provides a convenient way of enforcing validation rules for all incoming client payloads. You can specify these rules by using simple annotations in your DTO!

- To use the ValidationPipe, you need to install the `class-validator` and `class-transformer` packages:
`npm i --save class-validator class-transformer`

- To enable the ValidationPipe, you need to add it to the list of global pipes in the `main.ts` file:

```typescript
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe({
    // 👇 You can pass an options object to the ValidationPipe
    whitelist: true, // 👈 'whitelist' is going to remove any properties that are not defined in the DTO
    forbidNonWhitelisted: true, // 👈 'forbidNonWhitelisted' is going to throw an error if the client sends a property that is not defined in the DTO
    transform: true, // 👈 'transform' is going to transform the payload to the DTO instance
  }));
  await app.listen(3000);
}
bootstrap();
```

- Also install `@nestjs/mapped-types` package which provides a set of utility functions to help you create DTOs.
`npm i --save @nestjs/mapped-types`

## DTOs

The Data Transfer Object (DTO) is a pattern that is used to transfer data between processes. In Nest, DTOs are used to define the structure of the data that is passed to a controller's route handler.

**Example**:

```typescript
import { IsString, IsNotEmpty } from 'class-validator';

export class CreateBoardDto {
  @IsString()
  @IsNotEmpty()
  title: string; // 👈 title is a string and is required

  @IsString()
  @IsNotEmpty()
  description: string;
}
```

- Also You can use `PartialType()` to create a new DTO class from an existing one and make all of its properties optional.

```typescript
import { PartialType } from '@nestjs/mapped-types';
import { CreateBoardDto } from './create-board.dto';

export class UpdateBoardDto extends PartialType(CreateBoardDto) {}
```
