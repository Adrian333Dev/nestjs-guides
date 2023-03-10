# Protecting our routes with a Guard

## Auth Guard

In this example we gonna create a guard that will check if the user is authenticated or not. If the user is not authenticated, the guard will deny the access to the route.

- Generate a new guard

```bash
nest g guard iam/authentication/guards/access-token
```

- Define the `User Key` as a constant in `src/iam/iam.constants.ts`:

```ts
export const REQUEST_USER_KEY = 'user';
```

- Open the newly created guard and add the following code:

```ts
import {
  CanActivate,F
  ExecutionContext,
  Inject,
  Injectable,
  UnauthorizedException,
} from '@nestjs/common';
import { ConfigType } from '@nestjs/config';
import { JwtService } from '@nestjs/jwt';
import { Observable } from 'rxjs';
import jwtConfig from '../../config/jwt.config';
import { Request } from 'express';
import { REQUEST_USER_KEY } from '../../iam.constants';

@Injectable()
export class AccessTokenGuard implements CanActivate {
  constructor(
    private readonly jwtService: JwtService, // ð inject JwtService
    @Inject(jwtConfig.KEY) // ð inject jwtConfig
    private readonly jwtConfiguration: ConfigType<typeof jwtConfig>,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    // ð¡ NOTE: For GraphQL applications, youâd have to use the 
    // wrapper GqlExecutionContext here instead.
    const request = context.switchToHttp().getRequest(); // ð get request
    const token = this.extractTokenFromHeader(request); // ð extract token from header
    if (!token) throw new UnauthorizedException(); // ð throw if token is not present

    // ð if you are using HTTP only cookies approach, the code would look like this:
    // const token = request.cookies['access_token'];
    // if (!token) throw new UnauthorizedException();
    
    try {
      // ð verify token
      const payload = await this.jwtService.verifyAsync(
        token,
        this.jwtConfiguration,
      );

      /**
       * It's common to assign decoded payload to the request.user property.
       * This way, you can access the user in your endpoints or guards.
       */
      request[REQUEST_USER_KEY] = payload;
      console.log(payload);
    } catch {
      throw new UnauthorizedException();
    }
    return true;
  }

  // ð helper method to extract token from header
  private extractTokenFromHeader(request: Request): string | undefined {
    const [_, token] = request.headers.authorization?.split(' ') ?? [];
    return token;
  }
}
```

- Next, open `src/iam/iam.module.ts` and register the guard:

```ts
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';
import { AccessTokenGuard } from './guards/access-token.guard';

@Module({
  // ...
  providers: [
    // ð register guard
    {
      provide: APP_GUARD,
      useClass: AccessTokenGuard,
    },
  ],
  // ...
})
```

**Note:** Ypu can bind the guard to individual routes or to the entire module.
