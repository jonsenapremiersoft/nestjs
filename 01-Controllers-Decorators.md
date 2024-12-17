# Controllers e Decorators no NestJS

## 1. Estrutura Básica de um Controller

Os controllers são responsáveis por receber as requisições HTTP e retornar respostas. No NestJS, eles são definidos usando o decorator `@Controller()`.

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Get()
  findAll() {
    return ['João', 'Maria', 'Pedro'];
  }
}
```

Neste exemplo:
- `@Controller('users')` define o prefixo da rota como '/users'
- O método `findAll()` responderá a requisições GET em '/users'

## 2. Principais Decorators

O NestJS fornece vários decorators para definir rotas e métodos HTTP:

```typescript
import { Controller, Get, Post, Put, Delete, Patch } from '@nestjs/common';

@Controller('products')
export class ProductsController {
  @Get()
  findAll() {
    return 'Lista todos os produtos';
  }

  @Post()
  create() {
    return 'Cria um novo produto';
  }

  @Put(':id')
  update() {
    return 'Atualiza um produto';
  }

  @Delete(':id')
  remove() {
    return 'Remove um produto';
  }

  @Patch(':id')
  patch() {
    return 'Atualiza parcialmente um produto';
  }
}
```

## 3. Parâmetros de Rota e Query

O NestJS oferece decorators para acessar diferentes tipos de parâmetros:

```typescript
import { Controller, Get, Param, Query } from '@nestjs/common';

@Controller('products')
export class ProductsController {
  @Get(':id')
  findOne(@Param('id') id: string) {
    return `Produto com ID: ${id}`;
  }

  @Get()
  search(@Query('name') name: string, @Query('price') price: number) {
    return `Busca produtos com nome: ${name} e preço: ${price}`;
  }
}
```

- `@Param()`: acessa parâmetros de rota (URL segments)
- `@Query()`: acessa parâmetros de query string (?name=valor)

## 4. Requisições e Respostas HTTP

Para trabalhar com requisições e respostas de forma mais detalhada:

```typescript
import { Controller, Get, Post, Req, Res, Body, HttpStatus } from '@nestjs/common';
import { Request, Response } from 'express';

@Controller('products')
export class ProductsController {
  @Post()
  create(@Body() createProductDto: any, @Res() res: Response) {
    // Manipula os dados recebidos no body
    const product = {
      id: 1,
      ...createProductDto
    };

    return res.status(HttpStatus.CREATED).json(product);
  }

  @Get()
  findAll(@Req() request: Request) {
    // Acessa headers, cookies, etc.
    console.log(request.headers);
    return ['Produto 1', 'Produto 2'];
  }
}
```

## 5. Exercício Prático

Vamos criar um controller para gerenciar uma lista de tarefas (TODO list).

```typescript
import { Controller, Get, Post, Put, Delete, Body, Param } from '@nestjs/common';

interface Todo {
  id: number;
  title: string;
  completed: boolean;
}

@Controller('todos')
export class TodosController {
  private todos: Todo[] = [];
  private currentId = 1;

  @Post()
  create(@Body() todoData: { title: string }) {
    const todo: Todo = {
      id: this.currentId++,
      title: todoData.title,
      completed: false
    };
    this.todos.push(todo);
    return todo;
  }

  @Get()
  findAll() {
    return this.todos;
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.todos.find(todo => todo.id === Number(id));
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() todoData: { completed: boolean }) {
    const todo = this.todos.find(todo => todo.id === Number(id));
    if (todo) {
      todo.completed = todoData.completed;
    }
    return todo;
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    const index = this.todos.findIndex(todo => todo.id === Number(id));
    if (index >= 0) {
      this.todos.splice(index, 1);
    }
    return { success: true };
  }
}
```

### Desafio do Exercício:

1. Implemente o controller acima em seu projeto NestJS
2. Teste os endpoints usando Postman ou similar
3. Adicione validação para garantir que o título da tarefa não esteja vazio
4. Implemente um endpoint para marcar todas as tarefas como concluídas
5. Adicione um endpoint para filtrar tarefas por status (completas/incompletas)

## Pontos Importantes a Lembrar:

- Controllers são classes decoradas com `@Controller()`
- Um controller pode ter múltiplos métodos (endpoints)
- Use decorators apropriados para cada método HTTP (@Get, @Post, etc.)
- Parâmetros de rota são definidos com `:parametro` e acessados via `@Param()`
- Query strings são acessadas via `@Query()`
- O body da requisição é acessado via `@Body()`
- Use `@Res()` quando precisar manipular a resposta manualmente
