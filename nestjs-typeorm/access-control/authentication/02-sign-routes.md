# Implementing Sign-in and Sign-up Routes

In this section, we’ll be implementing routes that are essential to our authentication workflows

- **Sign-in:** which will authenticate a user by verifying their "credentials" (such as username/password, or an identity token from an Identity Provider) and manage authenticated state (by issuing a portable token, such as a JWT)
- **Sign-up:** that will let users register into the system

## Steps

- Install necessary packages for validation

  ```bash
  npm i class-validator class-transformer
  ```

---

- Register `ValidationPipe` globally in `main.ts`
  
  ```ts
  import { ValidationPipe } from '@nestjs/common';

  async function bootstrap() {
    // ...
    app.useGlobalPipes(new ValidationPipe());
    // ...
  }
  ```

---

- Generate `authentication` controller and service using Nest CLI

  ```bash
  nest g controller iam/authentication
  nest g service iam/authentication
  ```

- Generate `sign-in` and `sign-up` DTOs using Nest CLI

  ```bash
  nest g class iam/authentication/dto/sign-in.dto --no-spec
  nest g class iam/authentication/dto/sign-up.dto --no-spec
  ```

---

- Update the **DTOs** as follows:

  ```ts signup.dto.ts
  import { IsEmail, MinLength, IsNotEmpty } from 'class-validator';

  export class SignUpDto {
  @IsNotEmpty()
  @IsEmail()
  email: string;

  @IsNotEmpty()
  @MinLength(10)
  password: string;
  }
  ```

  ```ts signin.dto.ts
  import { IsEmail, MinLength, IsNotEmpty } from 'class-validator';

  export class SignInDto {
  @IsNotEmpty()
  @IsEmail()
  email: string;

  @IsNotEmpty()
  @MinLength(10)
  password: string;
  }
  ```

---

- Open newly generated `authentication.service.ts` and update it as follows:

  ```ts
  import {
  ConflictException,
  Injectable,
  UnauthorizedException,

  } from '@nestjs/common';
  import { InjectRepository } from '@nestjs/typeorm';
  import { Repository } from 'typeorm';
  import { User } from '../../users/entities/user.entity';
  import { HashingService } from '../hashing/hashing.service';
  import { SignInDto } from './dto/sign-in.dto';
  import { SignUpDto } from './dto/sign-up.dto';

  @Injectable()
  export class AuthenticationService {
    constructor(
      @InjectRepository(User) private readonly usersRepository: Repository<User>, // 👈 Inject Users Repository
      private readonly hashingService: HashingService, // 👈 Inject Hashing Service to hash passwords
    ) {}

    // 👇 Implement sign-up
    async signUp(signUpDto: SignUpDto) {
      try {
        const user = new User();
        user.email = signUpDto.email;
        user.password = await this.hashingService.hash(signUpDto.password); // 👈 Hash password before saving

        await this.usersRepository.save(user);
      } catch (err) {
        // Because email is unique, we can get a unique violation error
        // Since we're using PostgreSQL, we can check the error code to see if it's a unique violation
        const pgUniqueViolationErrorCode = '23505'; // 👈 PostgreSQL unique violation error code, sould be moved to a separate file
        if (err.code === pgUniqueViolationErrorCode) throw new ConflictException();
        throw err;
      }
    }

  async signIn(signInDto: SignInDto) {
    const user = await this.usersRepository.findOneBy({
      email: signInDto.email,
    });
    if (!user) {
      throw new UnauthorizedException('User does not exists');
    }
    const isEqual = await this.hashingService.compare(
      signInDto.password,
      user.password,
    );
    if (!isEqual) {
      throw new UnauthorizedException('Password does not match');
    }
    // TODO: We'll fill this gap in the next lesson
    return true;
    }
  }
  ```
