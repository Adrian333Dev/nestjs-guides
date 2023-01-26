# Connecting to the PostgreSQL database with TypeORM

TypeORM is a great ORM for Node.js. It supports many different databases, including PostgreSQL. In this tutorial, we will connect to a PostgreSQL database using TypeORM.

## Prerequisites

- [ ] [Docker](https://docs.docker.com/get-docker/) should be installed and running on your machine.

## Table of Contents

- [ ] [Docker Compose](#docker-compose)
- [ ] [Adding TypeORM to the project](#adding-typeorm-to-the-project)
- [ ] [Creating a TypeORM Entity](#creating-a-typeorm-entity)
- [ ] [Using Repository to Access the Database](#using-repository-to-access-the-database)

---

### Docker Compose

We will use Docker Compose to run a PostgreSQL database in a Docker container. Create a `docker-compose.yml` file in the root of your project and add the following content:

```yaml
version: "3"
services:
  db:
    image:  postgres
    restart: always 
    ports:
      - "5432:5432"
    environment:
       POSTGRES_PASSWORD: pass123
```

**Start** the container by running `docker-compose up -d` in the root of your project.
**Stop** the container by running `docker-compose down` in the root of your project.

---

### Adding TypeORM to the project

Install the necessary packages by running `npm install @nestjs/typeorm typeorm pg`.

Add the `TypeOrmModule` to the `imports` array in the `AppModule`:

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres', // type of our database
      host: 'localhost', // database host
      port: 5432, // database host
      username: 'postgres', // username
      password: 'pass123', // user password
      database: 'postgres', // name of our database,
      autoLoadEntities: true, // models will be loaded automatically 
      synchronize: true, // your entities will be synced with the database(recommended: disable in prod)
    }),
    // ...
  ],
  // ...
})
export class AppModule {}
```

---

### Creating a TypeORM Entity

Generate a resource if you haven't already:

Example: `nest g resource resources/board --no-spec` - `--no-spec` will prevent the CLI from generating a test file.

Go to the `board.entity.ts` file and add the following content:

```typescript
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity('boards') // 👈 sql table === 'boards', by default it was 'board'
export class Board {
  @PrimaryGeneratedColumn({ name: 'board_id'})
  boardId: number;

  @Column()
  title: string;

  @Column()
  description: string;
}
```

**Notes**:

- Don't forget to add the `@Entity()` decorator to the `Board` class. The `@Entity()` decorator takes a string argument which is the name of the table in the database.
- Also don't forget to register the `Board` entity in the `board.module.ts` file:

  ```typescript
  @Module({
    imports: [TypeOrmModule.forFeature([Board])], // 👈
    controllers: [BoardController],
    providers: [BoardService],
  })
  export class BoardModule {}
  ```

---

### Using Repository to Access the Database

In the `board.service.ts` file, inject the `Board` repository and add CRUD methods:

```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Board } from './board.entity';

@Injectable()
export class BoardService {
  constructor(
    @InjectRepository(Board) // 👈 inject the Board entity
    private boardRepository: Repository<Board>, // 👈 inject the Board repository
  ) {}

  async getAllBoards() {
    const boards = await this.boardRepository.find(); // 👈 get all boards from the database
    return boards;
  }

  async findBoardById(boardId: number) {
    const board = await this.boardRepository.findOne(boardId); // 👈 get a board by id from the database
    if (!board) throw new  NotFoundException(`Board with ID "${boardId}" not found`); // 👈 throw an error if the board is not found
    return board;
  }

  async createBoard(createBoardDto: CreateBoardDto) {
    const board = await this.boardRepository.create(createBoardDto); // 👈 create a new board
    return await this.boardRepository.save(board); // 👈 save the board to the database and return it
  }

  async updateBoard(boardId: number, updateBoardDto: UpdateBoardDto) {
    const board = await this.boardRepository.preload({ 
      boardId,
      ...updateBoardDto,
    }); // 👈 'preload' method creates a new entity based on the object passed to it
    if (!board) throw new  NotFoundException(`Board with ID "${boardId}" not found`); // 👈 throw an error if the board is not found
    return await this.boardRepository.save(board);
  }

  async deleteBoard(boardId: number) {
    const board = await this.findBoardById(boardId); // 👈 get the board by id
    return await this.boardRepository.remove(board);
  }
}
```
