### Schema Validation

Schema validation ensures that data stored in the database adheres to defined rules and constraints, such as data types, required fields, length restrictions, and unique values. Proper schema validation is essential for maintaining data integrity and preventing issues like invalid or inconsistent data entering your database.

In this section, we will explore how schema validation is implemented in **TypeORM**, **Prisma**, and **Mongoose**.

#### 1. TypeORM (SQL)

In **TypeORM**, schema validation is enforced through decorators on entity properties. These decorators specify the constraints on each field, such as whether it is required, unique, or has a specific length.

##### Example: TypeORM Schema Validation

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';
import { IsEmail, Length } from 'class-validator';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  @Length(3, 20) // Ensures username is between 3 and 20 characters
  username: string;

  @Column({ unique: true })
  @IsEmail() // Ensures the field contains a valid email
  email: string;

  @Column()
  @Length(8, 100) // Ensures password is at least 8 characters long
  password: string;
}
```

In this example:
- The `@Length` decorator ensures that the `username` and `password` fields meet certain length constraints.
- The `@IsEmail` decorator ensures that the `email` field contains a valid email address.
- The `{ unique: true }` option ensures the `email` is unique.

TypeORM integrates with libraries like **class-validator** to provide more advanced validation rules.

##### Validation on Save

When saving an entity in TypeORM, you can trigger validation using `validateOrReject()` from **class-validator**.

```typescript
import { validateOrReject } from 'class-validator';

const user = new User();
user.username = 'a'; // Invalid username
user.email = 'invalid-email'; // Invalid email
user.password = 'short'; // Invalid password

await validateOrReject(user).catch(errors => {
  console.log(errors); // Outputs validation errors
});
```

This ensures that invalid data does not get persisted to the database.

#### 2. Prisma (SQL)

Prisma enforces schema validation at the database level and in the generated TypeScript types. The Prisma schema file defines validation rules, such as field types, required fields, and unique constraints.

##### Example: Prisma Schema Validation

```prisma
model User {
  id       Int     @id @default(autoincrement())
  username String  @unique @db.VarChar(20) // Maximum length of 20 characters
  email    String  @unique @db.VarChar(100) @validate(email) // Must be a valid email
  password String  @db.VarChar(100) @validate(minLength: 8) // Minimum length of 8 characters
}
```

In this example:
- The `@unique` annotation ensures that the `username` and `email` fields are unique.
- The `@db.VarChar(20)` constraint limits the `username` length to 20 characters.
- `@validate(email)` and `@validate(minLength: 8)` enforce validation on `email` and `password`.

##### Runtime Validation in Prisma

Prisma ensures that data adheres to these rules at the database layer, so any invalid data will result in an error during a database operation.

```typescript
try {
  await prisma.user.create({
    data: {
      username: 'a', // Invalid username (too short)
      email: 'invalid-email', // Invalid email format
      password: 'short', // Password is too short
    },
  });
} catch (e) {
  console.error(e); // Prisma will throw an error if data is invalid
}
```

Prisma will catch any violations of the schema validation rules and throw an error, preventing invalid data from being persisted.

#### 3. Mongoose (NoSQL)

Mongoose enforces schema validation through its **schema definitions**. You can define required fields, unique constraints, type validation, and custom validation logic at the model level.

##### Example: Mongoose Schema Validation

```typescript
const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
    minlength: 3,
    maxlength: 20,
    unique: true,
  },
  email: {
    type: String,
    required: true,
    unique: true,
    match: [/^\S+@\S+\.\S+$/, 'Please enter a valid email address'],
  },
  password: {
    type: String,
    required: true,
    minlength: 8,
  },
});

const User = mongoose.model('User', userSchema);
```

In this example:
- The `required: true` constraint ensures that `username`, `email`, and `password` are required fields.
- `minlength` and `maxlength` define the character limits for `username` and `password`.
- The `unique: true` constraint ensures the uniqueness of `username` and `email`.
- The `match` option on `email` ensures it follows a valid email format.

##### Custom Validation in Mongoose

Mongoose also allows for custom validation logic using the `validate` property.

```typescript
const userSchema = new mongoose.Schema({
  age: {
    type: Number,
    validate: {
      validator: function (v) {
        return v >= 18; // Custom validation: age must be 18 or older
      },
      message: props => `${props.value} is below the minimum age!`,
    },
  },
});
```

In this example, the custom validation ensures that the `age` field must be 18 or older.

##### Validation on Save

Mongoose automatically validates data when saving a document to the database.

```typescript
const user = new User({
  username: 'a', // Invalid username
  email: 'invalid-email', // Invalid email
  password: 'short', // Invalid password
});

user.save((err) => {
  if (err) {
    console.error('Validation failed:', err);
  }
});
```

If the data is invalid, Mongoose throws a validation error, preventing invalid data from being saved.

---

Once you're ready, say "Next," and we will proceed to **Migrations**, where we’ll discuss database schema evolution and how to manage migrations using **TypeORM**, **Prisma**, and **Mongoose**.
