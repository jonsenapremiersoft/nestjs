# Módulos e Estrutura do Projeto no NestJS

## 1. Organização de Módulos

### 1.1 O que são Módulos?
Módulos são classes decoradas com `@Module()` que organizam componentes relacionados em um mesmo contexto. São fundamentais para:
- Manter o código organizado
- Estabelecer limites claros entre diferentes funcionalidades
- Gerenciar dependências
- Facilitar a manutenção

Exemplo básico de um módulo:

```typescript
// users/users.module.ts
@Module({
  imports: [],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService]
})
export class UsersModule {}
```

### 1.2 Anatomia de um Módulo
O decorador `@Module()` aceita um objeto com as seguintes propriedades:
- `imports`: Outros módulos que este módulo necessita
- `controllers`: Controllers que pertencem a este módulo
- `providers`: Serviços e outros providers do módulo
- `exports`: Providers que outros módulos podem importar

## 2. Importação e Exportação de Módulos

### 2.1 Importando Módulos
Para usar funcionalidades de outro módulo, você precisa importá-lo:

```typescript
// auth/auth.module.ts
@Module({
  imports: [UsersModule],
  controllers: [AuthController],
  providers: [AuthService]
})
export class AuthModule {}
```

### 2.2 Exportando Funcionalidades
Para disponibilizar serviços para outros módulos:

```typescript
// shared/shared.module.ts
@Module({
  providers: [LoggerService, ConfigService],
  exports: [LoggerService] // Apenas LoggerService será acessível externamente
})
export class SharedModule {}
```

## 3. Feature Modules vs Root Module

### 3.1 Root Module (AppModule)
- É o módulo principal da aplicação
- Importa outros módulos principais
- Configura aspectos globais da aplicação

```typescript
// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot(),
    DatabaseModule,
    UsersModule,
    AuthModule
  ],
  controllers: [AppController],
  providers: [AppService]
})
export class AppModule {}
```

### 3.2 Feature Modules
- Encapsulam funcionalidades específicas
- Focam em um domínio ou recurso particular
- Podem ser reutilizados em diferentes partes da aplicação

```typescript
// products/products.module.ts
@Module({
  imports: [DatabaseModule],
  controllers: [ProductsController],
  providers: [
    ProductsService,
    ProductsRepository
  ],
  exports: [ProductsService]
})
export class ProductsModule {}
```

## 4. Boas Práticas de Estruturação

### 4.1 Estrutura de Diretórios Recomendada
```
src/
├── app.module.ts
├── main.ts
├── users/
│   ├── dto/
│   ├── entities/
│   ├── users.controller.ts
│   ├── users.service.ts
│   └── users.module.ts
├── auth/
│   ├── dto/
│   ├── guards/
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   └── auth.module.ts
└── shared/
    ├── config/
    ├── interfaces/
    └── shared.module.ts
```

### 4.2 Princípios de Organização
1. **Separação por Domínio**: Agrupar arquivos relacionados ao mesmo domínio
2. **Single Responsibility**: Cada módulo deve ter uma única responsabilidade
3. **Encapsulamento**: Expor apenas o necessário através de exports
4. **Coesão**: Manter junto o que muda junto

## 5. Exercício Prático

### Objetivo
Reorganizar uma aplicação monolítica em módulos bem estruturados.

### Cenário Inicial
Considere uma API de e-commerce com as seguintes entidades:
- Produtos
- Pedidos
- Usuários
- Pagamentos

### Tarefa
1. Criar a estrutura de módulos apropriada
2. Implementar as dependências entre módulos
3. Organizar os arquivos seguindo as boas práticas

### Exemplo de Solução

```typescript
// products/products.module.ts
@Module({
  imports: [DatabaseModule],
  controllers: [ProductsController],
  providers: [ProductsService],
  exports: [ProductsService]
})
export class ProductsModule {}

// orders/orders.module.ts
@Module({
  imports: [
    ProductsModule,
    PaymentsModule,
    UsersModule
  ],
  controllers: [OrdersController],
  providers: [OrdersService]
})
export class OrdersModule {}

// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot(),
    DatabaseModule,
    ProductsModule,
    OrdersModule,
    UsersModule,
    PaymentsModule
  ],
  controllers: [],
  providers: []
})
export class AppModule {}
```

### Estrutura Final Esperada
```
src/
├── app.module.ts
├── main.ts
├── products/
│   ├── dto/
│   │   ├── create-product.dto.ts
│   │   └── update-product.dto.ts
│   ├── entities/
│   │   └── product.entity.ts
│   ├── products.controller.ts
│   ├── products.service.ts
│   └── products.module.ts
├── orders/
│   ├── dto/
│   ├── entities/
│   ├── orders.controller.ts
│   ├── orders.service.ts
│   └── orders.module.ts
└── shared/
    ├── config/
    ├── database/
    └── shared.module.ts
```
