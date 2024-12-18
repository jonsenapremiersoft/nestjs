## 1. O que é um DTO e Por Que Usar?

DTOs, ou Data Transfer Objects, são objetos que transportam dados entre diferentes partes de uma aplicação. Eles servem como uma camada de abstração que permite a transferência de dados de forma estruturada e segura. Os DTOs são particularmente úteis em aplicativos Nest.js, onde a modularidade e a organização do código são essenciais.

### Benefícios do uso de DTOs:

1. **Validação de Dados**
   - Impede que dados incorretos entrem na sua aplicação
   - Garante que todos os campos obrigatórios estejam presentes
   - Valida formatos (email, CPF, telefone, etc.)

2. **Segurança**
   - Evita que dados sensíveis sejam expostos
   - Previne ataques de injeção de dados maliciosos
   - Controla exatamente quais dados podem ser recebidos

3. **Documentação**
   - Torna a API mais clara para outros desenvolvedores
   - Facilita a integração com front-end
   - Melhora a manutenção do código

Vamos ver um exemplo prático de um cenário sem DTO vs com DTO:

```typescript
// SEM DTO - Perigoso!
@Post()
createUser(@Body() randomData: any) {
  // Como saber se os dados estão corretos?
  return this.userService.create(randomData);
}

// COM DTO - Seguro e validado!
@Post()
createUser(@Body() userData: CreateUserDto) {
  // Temos garantia que os dados seguem o padrão definido
  return this.userService.create(userData);
}
```

## 2. Criando seu Primeiro DTO

Vamos criar um DTO passo a passo para cadastro de usuário:

```typescript
// 1. Primeiro, crie um arquivo create-user.dto.ts
export class CreateUserDto {
  name: string;
  email: string;
  password: string;
  age: number;
}
```

### 2.1 Adicionando Validações

Para adicionar validações, primeiro instale as dependências:

```bash
npm install class-validator class-transformer
```

Agora vamos melhorar nosso DTO com validações:

```typescript
import { IsString, IsEmail, MinLength, IsInt, Min, Max } from 'class-validator';

export class CreateUserDto {
  @IsString({ message: 'O nome precisa ser uma string' })
  name: string;

  @IsEmail({}, { message: 'Insira um email válido' })
  email: string;

  @IsString()
  @MinLength(6, { message: 'A senha precisa ter no mínimo 6 caracteres' })
  password: string;

  @IsInt({ message: 'A idade precisa ser um número inteiro' })
  @Min(0, { message: 'A idade não pode ser negativa' })
  @Max(120, { message: 'A idade máxima é 120 anos' })
  age: number;
}
```

### 2.2 Habilitando a Validação

No seu `main.ts`, adicione:

```typescript
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Isso ativa a validação global
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true, // Remove dados que não estão no DTO
    forbidNonWhitelisted: true, // Retorna erro se enviarem dados extras
    transform: true // Transforma os dados recebidos para o tipo correto
  }));
  
  await app.listen(3000);
}
```

## 3. Casos de Uso Práticos

### 3.1 DTO para Criação vs Atualização

Geralmente, na criação precisamos de todos os dados, mas na atualização alguns podem ser opcionais:

```typescript
// create-user.dto.ts
export class CreateUserDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(6)
  password: string;
}

// update-user.dto.ts
export class UpdateUserDto {
  @IsString()
  @IsOptional()
  name?: string;

  @IsEmail()
  @IsOptional()
  email?: string;

  @IsString()
  @MinLength(6)
  @IsOptional()
  password?: string;
}
```

### 3.2 DTO com Relacionamentos

Quando precisamos lidar com dados relacionados:

```typescript
// create-order.dto.ts
export class CreateOrderDto {
  @IsNumber()
  userId: number;

  @ValidateNested({ each: true })
  @Type(() => OrderItemDto)
  items: OrderItemDto[];
}

// order-item.dto.ts
export class OrderItemDto {
  @IsNumber()
  productId: number;

  @IsInt()
  @Min(1)
  quantity: number;
}
```

### 3.3 DTO com Transformação de Dados

Às vezes precisamos transformar os dados antes de validá-los:

```typescript
export class CreateUserDto {
  @Transform(({ value }) => value.trim())
  @IsString()
  name: string;

  @Transform(({ value }) => value.toLowerCase())
  @IsEmail()
  email: string;

  @Transform(({ value }) => value.replace(/\s/g, ''))
  @IsString()
  @MinLength(6)
  password: string;
}
```

## 4. Exemplo Completo em um CRUD

Vamos ver como usar DTOs em um CRUD completo:

```typescript
// user.controller.ts
@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}

  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.userService.create(createUserDto);
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.userService.findOne(id);
  }

  @Patch(':id')
  update(
    @Param('id', ParseIntPipe) id: number,
    @Body() updateUserDto: UpdateUserDto
  ) {
    return this.userService.update(id, updateUserDto);
  }

  @Delete(':id')
  remove(@Param('id', ParseIntPipe) id: number) {
    return this.userService.remove(id);
  }
}
```

## 5. Dicas e Boas Práticas

1. **Organize seus DTOs**
   ```
   src/
   └── users/
       └── dto/
           ├── create-user.dto.ts
           ├── update-user.dto.ts
           └── user-response.dto.ts
   ```

2. **Use Herança quando fizer sentido**
   ```typescript
   // base-user.dto.ts
   export class BaseUserDto {
     @IsString()
     name: string;

     @IsEmail()
     email: string;
   }

   // create-user.dto.ts
   export class CreateUserDto extends BaseUserDto {
     @IsString()
     @MinLength(6)
     password: string;
   }
   ```

3. **Adicione Documentação Swagger**
   ```typescript
   export class CreateUserDto {
     @ApiProperty({
       description: 'Nome completo do usuário',
       example: 'João Silva'
     })
     @IsString()
     name: string;

     @ApiProperty({
       description: 'Email do usuário',
       example: 'joao@email.com'
     })
     @IsEmail()
     email: string;
   }
   ```

4. **Use Enums para Valores Fixos**
   ```typescript
   export enum UserRole {
     ADMIN = 'admin',
     USER = 'user',
     GUEST = 'guest'
   }

   export class CreateUserDto {
     @IsEnum(UserRole)
     role: UserRole;
   }
   ```

## 6. Problemas Comuns e Soluções

1. **Dados não estão sendo validados**
   - Verifique se o ValidationPipe está configurado no main.ts
   - Confirme se as dependências estão instaladas
   - Verifique se os decorators estão importados corretamente

2. **Erro de tipos no TypeScript**
   ```typescript
   // Solução: Use o Type do class-transformer
   import { Type } from 'class-transformer';

   export class CreateUserDto {
     @Type(() => Number)
     @IsNumber()
     age: number;
   }
   ```

3. **Validação de Arrays**
   ```typescript
   export class CreateUserDto {
     @IsArray()
     @IsString({ each: true })
     tags: string[];
   }
   ```
