# Active User Decorator

Active User Decorator is a custom decorator that we will allow us to access the current user which is stored in the request object.

## Links

- To learn more about custom decorators, check out the [custom decorators lesson](https://docs.nestjs.com/custom-decorators).

## Steps

- Create an interface that will represent the current user in `iam/interfaces/active-user.interface.ts`:

```ts
import { Role } from '../../users/enums/role.enum';
import { PermissionType } from '../authorization/permission.type';

export interface IActiveUser {
  /**
   * The "subject" of the token. The value of this property is the user ID
   * that granted this token.
   */
  sub: number;

  /**
   * The subject's (user) email.
   */
  email: string;
}
```

- Next, create the decorator in `iam/decorators/active-user.decorator.ts`:

```ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';
import { REQUEST_USER_KEY } from '../iam.constants';
import { IActiveUser } from '../interfaces/active-user.interface';

export const ActiveUser = createParamDecorator(
  (field: keyof IActiveUser | undefined, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user: IActiveUser | undefined = request[REQUEST_USER_KEY];
    return field ? user?.[field] : user;
  },
);
```

- Finally, open the `authenticaton.service.ts` and make sure that payload passed to the **JWT service** fulfills the `IActiveUser` interface:

```ts authentication.service.ts
const accessToken = await this.jwtService.signAsync(
  {
    sub: user.id,
    email: user.email,
  } as ActiveUserData,
  {
    audience: this.jwtConfiguration.audience,
    issuer: this.jwtConfiguration.issuer,
    secret: this.jwtConfiguration.secret,
    expiresIn: this.jwtConfiguration.accessTokenTtl,
  },
);
```
