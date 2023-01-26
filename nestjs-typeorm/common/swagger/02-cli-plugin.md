# Enabling CLI Plugin

- Add the `@nestjs/swagger` plugin to our application `nest-cli.json`

```json nest-cli.json
"compilerOptions": {
  "deleteOutDir": true,
  "plugins": ["@nestjs/swagger/plugin"] // 👈
}
```

**NOTE:** To Correctly reflect the dtos you need import `PartialType` from `@nestjs/swagger` instead of `@nestjs/mapped-types` in your all `update` dtos.

As an example:

```typescript update-user.dto.ts
import { PartialType } from '@nestjs/swagger'; // 👈 instead of @nestjs/mapped-types
import { CreateUserDto } from './create-user.dto';

export class UpdateUserDto extends PartialType(CreateUserDto) {}
```
