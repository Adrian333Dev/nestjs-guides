# Decorating Model Properties

NestJS provides us with `ApiProperty` decorator that allows us to decorate our model properties with metadata. This metadata is then used to generate Swagger documentation.

## Basic Usage

Let's take a look at a simple example:

```typescript
import { ApiProperty } from '@nestjs/swagger';

/**
 * @ApiProperty decorator useful to *override* 
 * information automatically inferred from the @nestjs/swagger plugin
 */
export class CreateBoardDto {
  @ApiProperty({ description: 'The title of the board' })
  title: string;

  @ApiProperty({ description: 'The description of the board' })
  description: string;
}
```
