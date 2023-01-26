# Adding Indexes to Entities

Indexes are used to improve the performance of queries. They are created by adding the `@Index` decorator to a property of an entity.

```typescript
import { Column, Entity, Index, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
@Index(['name', 'category']) // 👈 Add index to name and category columns
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  category: string;
}
```
