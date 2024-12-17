# Integrando Banco de Dados no NestJS com TypeORM

## 1. Preparação do Ambiente

```bash
npm install @nestjs/typeorm typeorm pg
```

E configurar os scripts no `package.json`:

```json
{
  "scripts": {
    "typeorm": "ts-node ./node_modules/typeorm/cli",
    "migration:generate": "npm run typeorm -- migration:generate -n",
    "migration:run": "npm run typeorm migration:run",
    "migration:revert": "npm run typeorm migration:revert"
  }
}
```

## 2. Configurando o TypeORM

Crie o arquivo `ormconfig.js` na raiz do projeto:

```javascript
module.exports = {
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'seu_usuario',
  password: 'sua_senha',
  database: 'nestjs_db',
  entities: ['dist/**/*.entity.js'],
  migrations: ['dist/migrations/*.js'],
  cli: {
    migrationsDir: 'src/migrations'
  }
}
```

Configure o módulo TypeORM no `app.module.ts`:

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'seu_usuario',
      password: 'sua_senha',
      database: 'nestjs_db',
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: false, // NUNCA use true em produção
    }),
  ],
})
export class AppModule {}
```

## 3. Criando Nossa Primeira Entidade

Vamos começar com a entidade User:

```typescript
// src/users/entities/user.entity.ts
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({ unique: true })
  email: string;
}
```

## 4. Gerando Nossa Primeira Migration

Agora vamos gerar a migration para criar a tabela users:

```bash
npm run migration:generate src/migrations/CreateUserTable
```

Isso vai gerar um arquivo como `src/migrations/[TIMESTAMP]-CreateUserTable.ts` com o SQL necessário para criar a tabela.

## 5. Executando a Migration

```bash
npm run migration:run
```

## 6. Evoluindo a Entidade User

Vamos adicionar o campo password:

```typescript
@Entity('users')
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string; // Novo campo
}
```

Gerando migration para a nova coluna:

```bash
npm run migration:generate src/migrations/AddPasswordToUser
```

Execute a nova migration:

```bash
npm run migration:run
```

## 7. Criando a Segunda Entidade com Relacionamento

Agora vamos criar a entidade Order:

```typescript
// src/orders/entities/order.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from 'typeorm';
import { User } from '../../users/entities/user.entity';

@Entity('orders')
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  description: string;

  @Column('decimal', { precision: 10, scale: 2 })
  total: number;

  @ManyToOne(() => User, user => user.orders)
  user: User;
}
```

E atualizar a entidade User para incluir o relacionamento:

```typescript
// src/users/entities/user.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from 'typeorm';
import { Order } from '../../orders/entities/order.entity';

@Entity('users')
export class User {
  // ... campos anteriores ...

  @OneToMany(() => Order, order => order.user)
  orders: Order[];
}
```

Gere a migration para criar a tabela orders e o relacionamento:

```bash
npm run migration:generate src/migrations/CreateOrderTable
```

Execute a migration:

```bash
npm run migration:run
```

## 8. Implementando o Módulo Users

```typescript
// src/users/users.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './entities/user.entity';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

## 9. Criando DTOs

```typescript
// src/users/dto/create-user.dto.ts
export class CreateUserDto {
  name: string;
  email: string;
  password: string;
}

// src/users/dto/update-user.dto.ts
import { PartialType } from '@nestjs/mapped-types';
import { CreateUserDto } from './create-user.dto';

export class UpdateUserDto extends PartialType(CreateUserDto) {}
```

## 10. Implementando o Service

```typescript
// src/users/users.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './entities/user.entity';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}

  create(createUserDto: CreateUserDto) {
    const user = this.usersRepository.create(createUserDto);
    return this.usersRepository.save(user);
  }

  findAll() {
    return this.usersRepository.find();
  }

  findOne(id: number) {
    return this.usersRepository.findOne({ where: { id } });
  }

  async update(id: number, updateUserDto: UpdateUserDto) {
    await this.usersRepository.update(id, updateUserDto);
    return this.findOne(id);
  }

  async remove(id: number) {
    await this.usersRepository.delete(id);
  }
}
```

## 11. Implementando o Controller

```typescript
// src/users/users.controller.ts
import { Controller, Get, Post, Body, Patch, Param, Delete } from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  @Get()
  findAll() {
    return this.usersService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(+id);
  }

  @Patch(':id')
  update(@Param('id') id: string, @Body() updateUserDto: UpdateUserDto) {
    return this.usersService.update(+id, updateUserDto);
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.usersService.remove(+id);
  }
}
```

## Exercício Prático

Agora é sua vez! Implemente o CRUD completo para Orders seguindo os mesmos passos:

1. Crie os DTOs para Order (CreateOrderDto e UpdateOrderDto)
2. Implemente o OrdersModule
3. Crie o OrdersService com os métodos CRUD
4. Implemente o OrdersController
5. Adicione um endpoint extra para listar todas as orders de um usuário específico

### Dica para listar orders de um usuário:

```typescript
// No OrdersService
findByUser(userId: number) {
  return this.ordersRepository.find({
    where: { user: { id: userId } },
    relations: ['user'],
  });
}
```


Se precisar reverter alguma migration, use:
```bash
npm run migration:revert
```
