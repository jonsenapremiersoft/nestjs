# Sistema de Pizzaria 🍕

Neste tutorial, vamos criar uma API para uma pizzaria, com duas entidades simples: `Category` (Categoria de Pizza) e `Pizza`. Por enquanto, não vamos implementar relacionamentos entre elas, mas aprenderemos como estruturar uma API completa!

## O que vamos construir?
- Uma API onde podemos cadastrar categorias de pizza (exemplo: Salgada, Doce, Vegetariana)
- Uma API para cadastrar as pizzas com suas informações
- Documentação automática com Swagger
- Validações de dados

## 1. Criando o Projeto

```bash
# Cria um novo projeto chamado pizzaria-api
nest new pizzaria-api

# Entre na pasta do projeto
cd pizzaria-api
```

Quando o NestJS perguntar qual gerenciador de pacotes você quer usar, escolha `npm`.

## 2. Instalando as Dependências Necessárias

```bash
# Instala o Prisma e seu cliente
npm install @prisma/client prisma

# Instala pacotes para documentação Swagger
npm install @nestjs/swagger swagger-ui-express

# Instala pacotes para validação
npm install class-validator class-transformer
```

## 3. Configurando o Prisma

O Prisma é uma ferramenta que nos ajuda a trabalhar com o banco de dados de forma mais fácil.

```bash
# Inicializa o Prisma no projeto
npx prisma init
```

Este comando cria:
- Uma pasta `prisma` com o arquivo `schema.prisma`
- Um arquivo `.env` para configurações

### 3.1. Configurando o Banco de Dados

Abra o arquivo `.env` e configure a URL do seu banco de dados:

```
DATABASE_URL="postgresql://seu_usuario:sua_senha@localhost:5432/pizzaria_db?schema=public"
```

Substitua `seu_usuario` e `sua_senha` pelos seus dados do PostgreSQL.

### 3.2. Definindo o Schema do Prisma

Abra `prisma/schema.prisma` e substitua seu conteúdo por:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Modelo para categorias de pizza
model Category {
  id          Int      @id @default(autoincrement())
  name        String   @unique
  description String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}

// Modelo para pizzas
model Pizza {
  id          Int      @id @default(autoincrement())
  name        String   @unique
  description String
  price       Decimal  @db.Decimal(10, 2)
  isAvailable Boolean  @default(true)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

### 3.3. Criando o Banco de Dados

```bash
# Cria o banco de dados e as tabelas
npx prisma migrate dev --name init
```

## 4. Criando a Estrutura do Projeto

Vamos usar o CLI do NestJS para criar nossos arquivos:

```bash
# Cria o módulo do Prisma
nest g module prisma

# Cria os módulos para categorias e pizzas
nest g module categories
nest g module pizzas

# Cria controllers e services
nest g controller categories
nest g service categories
nest g controller pizzas
nest g service pizzas
```

## 5. Configurando o Serviço do Prisma

Crie o arquivo `src/prisma/prisma.service.ts`:

```typescript
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
  // Conecta ao banco quando o módulo iniciar
  async onModuleInit() {
    await this.$connect();
  }

  // Desconecta do banco quando o módulo for destruído
  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

Atualize `src/prisma/prisma.module.ts`:

```typescript
import { Module, Global } from '@nestjs/common';
import { PrismaService } from './prisma.service';

// @Global() permite que o PrismaService seja usado em toda a aplicação
@Global()
@Module({
  providers: [PrismaService],
  exports: [PrismaService], // Permite que outros módulos usem o PrismaService
})
export class PrismaModule {}
```

## 6. Criando os DTOs

DTOs (Data Transfer Objects) são classes que definem como os dados devem ser enviados pela rede.

### 6.1. DTOs para Categorias

Crie a pasta `src/categories/dto` e dentro dela:

`create-category.dto.ts`:
```typescript
import { ApiProperty } from '@nestjs/swagger';
import { IsString, IsOptional, MinLength } from 'class-validator';

export class CreateCategoryDto {
  @ApiProperty({ example: 'Salgada', description: 'Nome da categoria' })
  @IsString()
  @MinLength(3)
  name: string;

  @ApiProperty({ 
    example: 'Pizzas salgadas tradicionais', 
    description: 'Descrição da categoria',
    required: false 
  })
  @IsString()
  @IsOptional()
  description?: string;
}
```

`update-category.dto.ts`:
```typescript
import { PartialType } from '@nestjs/swagger';
import { CreateCategoryDto } from './create-category.dto';

// PartialType faz com que todos os campos se tornem opcionais
export class UpdateCategoryDto extends PartialType(CreateCategoryDto) {}
```

### 6.2. DTOs para Pizzas

Crie a pasta `src/pizzas/dto` e dentro dela:

`create-pizza.dto.ts`:
```typescript
import { ApiProperty } from '@nestjs/swagger';
import { IsString, IsNumber, IsBoolean, IsOptional, MinLength } from 'class-validator';

export class CreatePizzaDto {
  @ApiProperty({ example: 'Margherita' })
  @IsString()
  @MinLength(3)
  name: string;

  @ApiProperty({ example: 'Molho de tomate, mussarela, manjericão' })
  @IsString()
  @MinLength(10)
  description: string;

  @ApiProperty({ example: 45.90 })
  @IsNumber()
  price: number;

  @ApiProperty({ example: true, required: false })
  @IsBoolean()
  @IsOptional()
  isAvailable?: boolean;
}
```

`update-pizza.dto.ts`:
```typescript
import { PartialType } from '@nestjs/swagger';
import { CreatePizzaDto } from './create-pizza.dto';

export class UpdatePizzaDto extends PartialType(CreatePizzaDto) {}
```

## 7. Implementando os Serviços

### 7.1. Categories Service

Atualize `src/categories/categories.service.ts`:

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreateCategoryDto } from './dto/create-category.dto';
import { UpdateCategoryDto } from './dto/update-category.dto';

@Injectable()
export class CategoriesService {
  constructor(private prisma: PrismaService) {}

  // Cria uma nova categoria
  async create(createCategoryDto: CreateCategoryDto) {
    return this.prisma.category.create({
      data: createCategoryDto,
    });
  }

  // Lista todas as categorias
  findAll() {
    return this.prisma.category.findMany();
  }

  // Busca uma categoria específica
  async findOne(id: number) {
    const category = await this.prisma.category.findUnique({
      where: { id },
    });

    if (!category) {
      throw new NotFoundException(`Categoria com ID ${id} não encontrada`);
    }

    return category;
  }

  // Atualiza uma categoria
  async update(id: number, updateCategoryDto: UpdateCategoryDto) {
    try {
      return await this.prisma.category.update({
        where: { id },
        data: updateCategoryDto,
      });
    } catch (error) {
      throw new NotFoundException(`Categoria com ID ${id} não encontrada`);
    }
  }

  // Remove uma categoria
  async remove(id: number) {
    try {
      return await this.prisma.category.delete({
        where: { id },
      });
    } catch (error) {
      throw new NotFoundException(`Categoria com ID ${id} não encontrada`);
    }
  }
}
```

### 7.2. Pizzas Service

Implemente `src/pizzas/pizzas.service.ts` de forma similar, adaptando para o modelo Pizza.

## 8. Implementando os Controllers

### 8.1. Categories Controller

Atualize `src/categories/categories.controller.ts`:

```typescript
import { 
  Controller, 
  Get, 
  Post, 
  Body, 
  Patch, 
  Param, 
  Delete,
  ParseIntPipe 
} from '@nestjs/common';
import { ApiTags, ApiOperation, ApiResponse } from '@nestjs/swagger';
import { CategoriesService } from './categories.service';
import { CreateCategoryDto } from './dto/create-category.dto';
import { UpdateCategoryDto } from './dto/update-category.dto';

@ApiTags('categories') // Agrupa as rotas no Swagger
@Controller('categories') // Define o prefixo da rota como /categories
export class CategoriesController {
  constructor(private readonly categoriesService: CategoriesService) {}

  @Post()
  @ApiOperation({ summary: 'Cria uma nova categoria' })
  @ApiResponse({ status: 201, description: 'Categoria criada com sucesso.' })
  create(@Body() createCategoryDto: CreateCategoryDto) {
    return this.categoriesService.create(createCategoryDto);
  }

  @Get()
  @ApiOperation({ summary: 'Lista todas as categorias' })
  findAll() {
    return this.categoriesService.findAll();
  }

  @Get(':id')
  @ApiOperation({ summary: 'Busca uma categoria pelo ID' })
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.categoriesService.findOne(id);
  }

  @Patch(':id')
  @ApiOperation({ summary: 'Atualiza uma categoria' })
  update(
    @Param('id', ParseIntPipe) id: number,
    @Body() updateCategoryDto: UpdateCategoryDto,
  ) {
    return this.categoriesService.update(id, updateCategoryDto);
  }

  @Delete(':id')
  @ApiOperation({ summary: 'Remove uma categoria' })
  remove(@Param('id', ParseIntPipe) id: number) {
    return this.categoriesService.remove(id);
  }
}
```

### 8.2. Pizzas Controller

Implemente o `PizzasController` de forma similar, adaptando para as pizzas.

## 9. Configurando o Swagger

Atualize `src/main.ts`:

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Ativa a validação global
  app.useGlobalPipes(new ValidationPipe());

  // Configura o Swagger
  const config = new DocumentBuilder()
    .setTitle('Pizzaria API')
    .setDescription('API para gerenciamento de pizzaria')
    .setVersion('1.0')
    .build();
  
  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, document);

  await app.listen(3000);
}
bootstrap();
```

## 10. Como Testar a API

1. Inicie o servidor:
```bash
npm run start:dev
```

2. Acesse a documentação Swagger:
```
http://localhost:3000/api
```

3. Teste as rotas:
   - Crie algumas categorias (POST /categories)
   - Liste as categorias (GET /categories)
   - Crie algumas pizzas (POST /pizzas)
   - Liste as pizzas (GET /pizzas)
   - Teste os updates e deletes

## Sugestões de Melhorias Simples

1. **Adicionar Filtros Básicos**:
   - Buscar pizzas por faixa de preço
   - Buscar pizzas disponíveis/indisponíveis
   - Buscar categorias por nome

2. **Melhorar a Validação**:
   - Adicionar mais regras de validação (ex: preço mínimo)
   - Criar mensagens de erro personalizadas
   - Validar formatos específicos (ex: nome sem caracteres especiais)


- [NestJS](https://docs.nestjs.com/)
- [Prisma](https://www.prisma.io/docs/)
