# Hashing Passwords

## Overview

Hashing passwords is a common practice in web applications. It is a way to store passwords in a database in a way that they cannot be easily read. This is important because if a database is compromised, the passwords are not readable.

## Hashing

In this section, we will use the `bcrypt` library to hash passwords. The `bcrypt` algorithm is a one-way algorithm. This means that it is not possible to reverse the hashing process.

- Install necessary packages

```bash
npm i bcrypt
npm i @types/bcrypt -D
```

- Generate **IAM module** & **Hashing / BCrypt files** using Nest CLI

```bash
nest g module iam
nest g service iam/hashing
nest g service iam/hashing/bcrypt --flat
```

- Go to `src/iam/hashing.service.ts` and add the following code

```ts
import { Injectable } from '@nestjs/common';

/**
 * 👇 Make this service @abstract as we will use it as an interface
 */
@Injectable()
export abstract class HashingService {
  // 👇 hash() method takes data as a string or buffer and returns hashed string
  abstract hash(data: string | Buffer): Promise<string>; 

  // 👇 compare() method takes two arguments, data and encrypted string and returns boolean
  abstract compare(data: string | Buffer, encrypted: string): Promise<boolean>;
}
```

- Go to `src/iam/hashing/bcrypt.service.ts` and add the following code

```ts
import { Injectable } from '@nestjs/common';
import { hash, genSalt, compare } from 'bcrypt'; // 👈 import bcrypt methods
import { HashingService } from './hashing.service'; // 👈 import HashingService

@Injectable()
export class BcryptService implements HashingService { // 👈 implement HashingService

  // 👇 hash() method takes data as a string or buffer and returns hashed string
  hash(data: string | Buffer): Promise<string> {
    const salt = await genSalt();
    return hash(data, salt);
  }

  // 👇 compare() method takes two arguments, data and encrypted string and returns boolean
  compare(data: string | Buffer, encrypted: string): Promise<boolean> {
    return compare(data, encrypted);
  }
}
```

- Go to `src/iam/iam.module.ts` and add the following code

```ts
import { Module } from '@nestjs/common';
import { BcryptService } from './hashing/bcrypt.service';
import { HashingService } from './hashing.service';

@Module({
  providers: [
    {
      provide: HashingService, // 👈 When HashingService is injected, use BcryptService instead
      useClass: BcryptService,
    },
  ],
  exports: [HashingService],
})
```
