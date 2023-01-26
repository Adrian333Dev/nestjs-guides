# Use Transaction

Implementing Transaction For Product recommendation as an example.

## Steps

- Generate Event Entity by running `nest g class events/entities/event.entity --no-spec` command.
- Update `events/entities/event.entity.ts` file as follows:

```typescript
import { Column, Entity, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class Event {
  @PrimaryGeneratedColumn({ name: 'event_id' })
  eventId: number;

  @Column()
  type: string; 

  @Column()
  name: string;  

  @Column('json')
  payload: Record<string, any>;
}
```

- **Note**: Don't forget to register `Event` entity in corresponding module.

---

- Add `recommendation` column to `Product` entity.

```typescript

@Entity()
export class Product {
  // ...
  @Column({ default: 0 })
  recommendation: string; // 👈 Add this column
}
```

- Update the `products.service.ts` file as follows:

```typescript
import { DataSource } from 'typeorm';

@Injectable()
export class ProductsService {
  constructor(
    private readonly dataSource: DataSource, // 👈 Inject the DataSource to use it in the transaction
  ) {}

  async createProduct(createProductDto: CreateProductDto): Promise<Product> {
    const product = this.productRepository.create(createProductDto);
    await this.productRepository.save(product);

    const event = new Event();
    event.type = 'product_created';
    event.name = 'Product Created';
    event.payload = { productId: product.id };

    const connection = await this.dataSource.getConnection();
    const queryRunner = connection.createQueryRunner();

    await queryRunner.connect();
    await queryRunner.startTransaction();

    try {
      await queryRunner.manager.save(event);
      await queryRunner.manager.increment(Product, { id: product.id }, 'recommendation', 1);
      await queryRunner.commitTransaction();
    } catch (err) {
      await queryRunner.rollbackTransaction();
    } finally {
      await queryRunner.release();
    }

    return product;
  }
}
```

**Read More**:

- [Handling Transactions in TypeORM and Nest.js](https://betterprogramming.pub/handling-transactions-in-typeorm-and-nest-js-with-ease-3a417e6ab5)
