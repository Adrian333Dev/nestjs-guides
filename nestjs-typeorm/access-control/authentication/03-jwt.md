# JWT

**JSON Web Tokens (or JWT’s)** are an open standard used to share security information between two parties — a client and a server.  

Each JWT contains encoded JSON objects, including a set of claims. JWTs are signed using a cryptographic algorithm to ensure that the claims cannot be altered after the token has been issued.
JWTs can be signed using a secret (with the HMAC algorithm) or a public/private key pair using RSA or ECDSA.

Learn more about JWTs in general: <https://jwt.io/>

## Steps

- Install necessary packages for validation

  ```bash
  npm i @nestjs/jwt @nestjs/config
  ```

- Open `src/app.module.ts` and add the following imports:

  ```ts
  import { ConfigModule } from '@nestjs/config';

  @Module({
    imports: [
      // ...
      ConfigModule.forRoot(), // 👈 import ConfigModule and initialize it
    ],
  })
  ```

- Create `.env` file in the root directory and add the following code:
  
  ```env
  JWT_SECRET=YOUR_SECRET_KEY_HERE # 👈 replace with your own secret key
  JWT_TOKEN_AUDIENCE=localhost:3000
  JWT_TOKEN_ISSUER=localhost:3000
  JWT_ACCESS_TOKEN_TTL=3600
  ```

- Create `src/iam/config/jwt.config.ts` and add the following code:

  ```ts
  import { registerAs } from '@nestjs/config';

  export default registerAs('jwt', () => {
    return {
      secret: process.env.JWT_SECRET,
      audience: process.env.JWT_TOKEN_AUDIENCE,
      issuer: process.env.JWT_TOKEN_ISSUER,
      accessTokenTtl: parseInt(process.env.JWT_ACCESS_TOKEN_TTL ?? '3600', 10),
    };
  });
  ```

- Next open the `iam.module.ts` file and register `JwtModule` and `ConfigModule`:

  ```ts
  import { JwtModule } from '@nestjs/jwt';
  import { ConfigModule } from '@nestjs/config';
  import jwtConfig from './config/jwt.config';

  @Module({
    imports: [
      // ...
      JwtModule.registerAsync(jwtConfig.asProvider()), // 👈 register JwtModule
      ConfigModule.forFeature(jwtConfig), // 👈 register ConfigModule
    ],
  })
  ```

- Now navigate to `src/iam/authentications/authentications.service.ts` and update the code to the following:

  ```ts
  import {
    ConflictException,
    Inject,
    Injectable,
    UnauthorizedException,
  } from '@nestjs/common';
  import { ConfigType } from '@nestjs/config';
  import { JwtService } from '@nestjs/jwt';
  import { InjectRepository } from '@nestjs/typeorm';
  import { Repository } from 'typeorm';
  import { User } from '../../users/entities/user.entity';
  import jwtConfig from '../config/jwt.config';
  import { HashingService } from '../hashing/hashing.service';
  import { SignInDto } from './dto/sign-in.dto';
  import { SignUpDto } from './dto/sign-up.dto';

  @Injectable()
  export class AuthenticationService {
    constructor(
      @InjectRepository(User) private readonly usersRepository: Repository<User>,
      private readonly hashingService: HashingService,
      private readonly jwtService: JwtService, // 👈 inject JwtService
      @Inject(jwtConfig.KEY) // 👈 inject jwtConfig
      private readonly jwtConfiguration: ConfigType<typeof jwtConfig>,
    ) {}

    async signUp(signUpDto: SignUpDto) {
      try {
        const user = new User();
        user.email = signUpDto.email;
        user.password = await this.hashingService.hash(signUpDto.password);

        await this.usersRepository.save(user);
      } catch (err) {
        const pgUniqueViolationErrorCode = '23505';
        if (err.code === pgUniqueViolationErrorCode) throw new ConflictException();
        
        throw err;
      }
    }

    async signIn(signInDto: SignInDto) {
      const user = await this.usersRepository.findOneBy({
        email: signInDto.email,
      });
      if (!user) throw new UnauthorizedException('User does not exists');
      
      const isEqual = await this.hashingService.compare(
        signInDto.password,
        user.password,
      );
      if (!isEqual) throw new UnauthorizedException('Password does not match');
      
      const accessToken = await this.jwtService.signAsync( // 👈 Generate JWT
        {
          sub: user.id,
          email: user.email,
        },
        {
          audience: this.jwtConfiguration.audience,
          issuer: this.jwtConfiguration.issuer,
          secret: this.jwtConfiguration.secret,
          expiresIn: this.jwtConfiguration.accessTokenTtl,
        },
      );
      return {
        accessToken,
      };
    }
  }
  ```

**Note:** There are safer ways to send the JWT to the client, for Example by using the HTTPOnly cookie. But for now, we will just return the JWT as a response.

  ```ts auth.controller.ts - Optional
  import { Response } from 'express';

  // ...
  @Post('sign-in')
  async signIn(
    @Body() signInDto: SignInDto,
    @Res() response: Response,
  ) {
  const accessToken = await this.authService.signIn(signInDto);
  response.cookie('accessToken', accessToken, {
    secure: true,
    httpOnly: true,
    sameSite: true,
  });
  }
  // ...
  ```

## [⬅️ Previous](./02-sign-routes.md) | [➡️ Next](./04-guards.md)
