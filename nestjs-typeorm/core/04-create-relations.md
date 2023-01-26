# Creating Relations between Entities

## Introduction

**Relations** are associations established between two or more tables, based on common fields from each table, often involving primary and foreign keys.

**There are three types of relations:**

**One-to-one** - The first are one-to-one relations. In these relations every row in the primary table has one - and only one associated row in the foreign table. In TypeOrm, we define these types of relations with the `@OneToOne()` decorator.

**One-to-many or Many-to-one** - For these relations - every row in the primary table has one or more related rows in the foreign table. In TypeOrm, we define these types of relations with the  `@OneToMany()` and `@ManyToOne()` decorators.

**Many-to-many** - This is when every row in the primary table has many related rows in the foreign table, and every record in the foreign table has many related rows in the primary table. In TypeOrm, we define these types of relations with the  `@ManyToMany()` decorator.

---

### Many-to-many relations

Let's say we have a `User` entity and a `Board` entity. A user can be member of many boards, and a board can have many members. This is a many-to-many relation. We can define it like this:

```typescript board.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable } from 'typeorm';
import { User } from './user.entity';

@Entity()
export class Board {
  @PrimaryGeneratedColumn({name: 'board_id'})
  boardId: number;

  @Column()
  name: string;

  @JoinTable()  // 👈 Join the 2 tables - only the OWNER-side does this
  @ManyToMany(type => User, user => user.boards) // 👈 What is "boards" in the User entity?
  members: User[];
}
```

```typescript user.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany } from 'typeorm';
import { Board } from './board.entity';

@Entity()
export class User {
  @PrimaryGeneratedColumn({name: 'user_id'})
  userId: number;

  @Column()
  name: string;

  @ManyToMany(type => Board, board => board.members) // 👈 What is "members" in the Board entity?
  boards: Board[];
}
```

---

### One-to-many or Many-to-one relations

Let's say we have a `Board` entity and a `List` entity. A board can have many lists, and a list can only belong to one board. This is a one-to-many relation. We can define it like this:

```typescript board.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, JoinColumn, OneToMany } from 'typeorm';
import { List } from './list.entity';

@Entity()
export class Board {
  @PrimaryGeneratedColumn({ name: 'board_id' })
  boardId: number;

  @Column()
  name: string;

  @OneToMany(type => List, list => list.board)
  @JoinColumn() // ! 👈 Pay attention that we use @JoinColumn() here instead of @JoinTable()
  lists: List[];
}
```

```typescript list.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from 'typeorm';
import { Board } from './board.entity';

@Entity()
export class List {
  @PrimaryGeneratedColumn({ name: 'list_id' })
  listId: number;

  @Column()
  title: string;

  @ManyToOne(type => Board, board => board.lists)
  board: Board;
}
```

**NOTE:** Don't forget register the `List` entity in the `board.module.ts` file.

```typescript board.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([Board, List])], // 👈 Add the List entity here
  // ...
})
```

---

### One-to-one relations

Let's say we have a `User` entity and a `Profile` entity. A user can have one profile, and a profile can only belong to one user. This is a one-to-one relation. We can define it like this:

```typescript user.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, OneToOne, JoinColumn } from 'typeorm';
import { Profile } from './profile.entity';

@Entity()
export class User {
  @PrimaryGeneratedColumn({ name: 'user_id' })
  userId: number;

  @Column()
  name: string;

  @OneToOne(type => Profile, profile => profile.user)
  @JoinColumn() // ! 👈 Pay attention that we use @JoinColumn() here instead of @JoinTable()
  profile: Profile;
}
```

```typescript profile.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, OneToOne } from 'typeorm';
import { User } from './user.entity';

@Entity()
export class Profile {
  @PrimaryGeneratedColumn({ name: 'profile_id' })
  profileId: number;

  @Column()
  bio: string;

  @OneToOne(type => User, user => user.profile)
  user: User;
}
```

**NOTE:**

- Take note of the `@JoinTable()` decorator. This is only needed on the OWNER-side of the relation. The `@JoinTable()` decorator is used to specify the name of the join table and the column names of the primary and foreign keys in the join table. In this case, the join table is called `board_members_user` and the primary key is `boardId` and the foreign key is `userId`.
- `@JoinTable()` is a decorator that is used to create a **many-to-many** relationship between two entities, whereas `@JoinColumn()` is used to create a **one-to-one** or **many-to-one** relationship between two entities, **if you use it otherwise, unexpected results may occur**.
- **Don't forget** register all the entities in corresponding modules.

---

### Retrieving relations

We can retrieve relations by passing the **relations** option to the `find()` method. For example, to retrieve all the boards with their members, we can do this:

```typescript board.service.ts

@Injectable()
export class BoardService {
  constructor(
    @InjectRepository(Board)
    private boardRepository: Repository<Board>,
  ) {}

  async getAllBoards(): Promise<Board[]> {
    const boards = await this.boardRepository.find({ 
      relations: ['members'] // 👈 Retrieve the members of each board
    });
    return boards;
  }

  async getBoardById(id: number): Promise<Board> {
    const board = await this.boardRepository.findOne(id, {
      relations: ['members'] // 👈 Retrieve the members of the board
    });
    return board;
  }
}
```

---

### Using Cascading Inserts

Let's say we want to create a new task and add labels to it in one go. But not all the labels exist in the database. We want to create the labels that don't exist in the database. We can do this by using cascading inserts.

- First, we need to define the cascading inserts in the `Task` entity:

```typescript task.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable } from 'typeorm';
import { Label } from './label.entity';

@Entity()
export class Task {
  @PrimaryGeneratedColumn({ name: 'task_id' })
  taskId: number;

  @Column()
  title: string;

  @ManyToMany(type => Label, label => label.tasks, 
    { cascade: true } // 👈 Enable cascading inserts
  )
  @JoinTable()
  labels: Label[];
}
```

- Then, we create `preloadLabelsByName` method in the `TaskService` to preload the labels that don't exist in the database:

```typescript task.service.ts
@Injectable()
export class TaskService {
  constructor(
    @InjectRepository(Task)
    private taskRepository: Repository<Task>,
    @InjectRepository(Label)
    private labelRepository: Repository<Label>,
  ) {}

  async create(createTaskDto: CreateTaskDto) {
    const labels = await Promise.all(
      createTaskDto.labels.map(name => this.preloadLabelByName(name)), // 👈 Preload the labels
    );

    const task = this.taskRepository.create({
      ...createTaskDto,
      labels,
    });
    return this.taskRepository.save(task);
  }

  async update(taskId: number, updateTaskDto: UpdateTaskDto) {
    const labels = await Promise.all(
      updateTaskDto.labels.map(name => this.preloadLabelByName(name)), // 👈 Preload the labels
    );

    const task = await this.taskRepository.preload({
      taskId,
      ...updateTaskDto,
      labels,
    });
    if (!task) throw new NotFoundException(`Task #${taskId} not found`);
    
    return this.taskRepository.save(task);
  }

  private async preloadLabelByName(name: string): Promise<Label> {
    const existingLabel = await this.LabelRepository.findOne({ 
      where: { name } // 👈 notice the "where" option
    }); 
    if (existingLabel) return existingLabel;
    return this.labelRepository.create({ name });
  }
}
```

- And finally, add labels to the `createTaskDto`:

```typescript task.dto.ts
export class CreateTaskDto {
  @IsNotEmpty()
  @IsString()
  title: string;

  @IsString({ each: true })
  labels: string[]; // 👈 Add labels to the createTaskDto
}
```

Now when we create a new task, we can pass the labels that don't exist in the database. The labels will be created automatically.

```json request.json
{
  "title": "Task 1",
  "labels": ["label 1", "label 2"]
}
```

**NOTE:**

- Don't forget to register the `Label` entity in the `task.module.ts` file.

```typescript task.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([Task, Label])], // 👈 Add the Label entity here
  // ...
})
```

---
