# Adding Public Routes

In this section, we will add a public route to our application. This route will be accessible to all users, even those who are not logged in.

## Steps

- Create new enum that will indicate the type of authentication required for a route in `iam/enums/auth-type.enum.ts`:

```ts
export enum AuthType {
  Bearer, // Default Strategy
  None, // No Authentication Required
}
```

- Next create auth decorator that will let us choose the strategy for the route or an entire controller in `iam/decorators/auth.decorator.ts`:

```ts
import { SetMetadata } from '@nestjs/common';
import { AuthType } from '../enums/auth-type.enum';

export const AUTH_TYPE_KEY = 'authType';

export const Auth = (...authTypes: AuthType[]) =>
  SetMetadata(AUTH_TYPE_KEY, authTypes);
```

- Generate the new guard by running the following command:

```bash
nest g guard iam/authentication/guards/authentication
```

- Update the guard to use the new decorator in `iam/authentication/guards/authentication.guard.ts`:

```ts
import {
  CanActivate,
  ExecutionContext,
  Injectable,
  UnauthorizedException,
} from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { Observable } from 'rxjs';
import { AUTH_TYPE_KEY } from '../decorators/auth.decorator';
import { AuthType } from '../enums/auth-type.enum';
import { AccessTokenGuard } from './access-token.guard';

@Injectable()
export class AuthenticationGuard implements CanActivate {
  private static readonly defaultAuthType = AuthType.Bearer; // 👈 Set default auth type
  private readonly authTypeGuardMap: Record<
    AuthType,
    CanActivate | CanActivate[]
  > = {
    [AuthType.Bearer]: this.accessTokenGuard,
    [AuthType.None]: { canActivate: () => true },
  }; // 👈 Map auth type to guard

  constructor(
    private readonly reflector: Reflector, // 👈 Gives us access to the underlying metadata
    private readonly accessTokenGuard: AccessTokenGuard, // 👈 Our default guard
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const authTypes = this.reflector.getAllAndOverride<AuthType[]>(
      AUTH_TYPE_KEY,
      [context.getHandler(), context.getClass()],
    ) ?? [AuthenticationGuard.defaultAuthType];
    const guards = authTypes.map((type) => this.authTypeGuardMap[type]).flat();
    let error = new UnauthorizedException();

    for (const instance of guards) {
      const canActivate = await Promise.resolve(
        instance.canActivate(context),
      ).catch((err) => {
        error = err;
      });

      if (canActivate) {
        return true;
      }
    }
    throw error;
  }
}
```

- Now open `iam.module.ts` and replace the `AccessTokenGuard` with the new `AuthenticationGuard`:

```ts
@Module({
  providers: [
    // ...
    {
      provide: APP_GUARD,
      useClass: AuthenticationGuard,
    },
    AccessTokenGuard, // 👈 Make sure to register access token guard as a provider
  ],
})
```

- To use the new decorator, open `auth.controller.ts` and apply the `Auth` decorator to the whole controller or to individual routes:

```ts auth.controller.ts
import { Auth } from 'iam/decorators/auth.decorator';
import { AuthType } from 'iam/enums/auth-type.enum';

@Auth(AuthType.None) // 👈 Apply decorator to the whole controller
@Controller('auth')
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Auth(AuthType.Bearer) // 👈 Apply decorator to individual routes
  @Get('me')
  async getMe(@Request() req) {
    return req.user;
  }
}
```
