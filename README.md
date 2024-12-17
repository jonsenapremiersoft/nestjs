# Introdução ao NestJS

## 1. O que é NestJS?
NestJS é um framework Node.js que foi criado para resolver problemas comuns no desenvolvimento de aplicações backend. Enquanto o Express nos dá muita liberdade, às vezes essa liberdade pode levar a códigos desorganizados em projetos grandes. O NestJS resolve isso oferecendo uma estrutura organizada e bem definida.

Imagine o NestJS como um conjunto de regras e ferramentas que ajudam você a construir sua aplicação de forma mais organizada, como se fosse uma planta de uma casa - cada coisa tem seu lugar específico.

## 2. Arquitetura do NestJS

### Estrutura em Camadas
No NestJS, organizamos nossa aplicação em diferentes camadas, cada uma com uma responsabilidade específica:

1. **Controllers (Controladores)**:
   - São como recepcionistas da sua aplicação
   - Recebem as requisições HTTP (quando alguém acessa sua API)
   - Não devem conter lógica de negócio, apenas direcionam as requisições
   - Exemplo prático:
   ```typescript
   @Controller('usuarios')
   export class UsuariosController {
     @Get()
     listarUsuarios() {
       // Aqui apenas chamamos o serviço responsável
       return this.usuariosService.listarTodos();
     }
   }
   ```

2. **Services (Serviços)**:
   - Contêm a lógica de negócio da sua aplicação
   - Fazem o trabalho pesado (cálculos, regras de negócio, acesso ao banco)
   - Exemplo prático:
   ```typescript
   @Injectable()
   export class UsuariosService {
     private usuarios = [];

     listarTodos() {
       // Aqui processamos a lógica de negócio
       return this.usuarios.filter(usuario => usuario.ativo === true);
     }
   }
   ```

3. **Modules (Módulos)**:
   - São como containers que agrupam código relacionado
   - Ajudam a organizar a aplicação em blocos funcionais
   - Exemplo: Um módulo de usuários agrupa tudo relacionado a usuários
   ```typescript
   @Module({
     controllers: [UsuariosController], // Controladores deste módulo
     providers: [UsuariosService],      // Serviços deste módulo
   })
   export class UsuariosModule {}
   ```

## 3. NestJS vs Express

### Express (Forma Tradicional)
No Express, você normalmente escreve código assim:
```javascript
const express = require('express');
const app = express();

// Rota para listar usuários
app.get('/usuarios', (req, res) => {
  const usuarios = []; // Busca no banco de dados
  res.json(usuarios);
});

// Rota para criar usuário
app.post('/usuarios', (req, res) => {
  const novoUsuario = req.body;
  // Salvar no banco de dados
  res.json(novoUsuario);
});
```

### NestJS
No NestJS, o mesmo código fica assim:
```typescript
@Controller('usuarios')
export class UsuariosController {
  constructor(private usuariosService: UsuariosService) {}

  @Get()
  listarUsuarios() {
    return this.usuariosService.listarTodos();
  }

  @Post()
  criarUsuario(@Body() usuario: CriarUsuarioDto) {
    return this.usuariosService.criar(usuario);
  }
}
```

### Por que o NestJS é diferente?
1. **Decorators (@)**: São como etiquetas que dizem ao NestJS como tratar cada parte do código
   - `@Controller`: Diz que esta classe vai lidar com requisições HTTP
   - `@Get()`, `@Post()`: Indicam qual método HTTP usar
   - `@Body()`: Diz ao NestJS para pegar os dados enviados na requisição

2. **Injeção de Dependência**: 
   - No construtor do controller, o NestJS automaticamente cria e fornece uma instância do service
   - Isso torna o código mais fácil de testar e manter

## 4. Configurando seu Primeiro Projeto

1. **Instalação**:
   ```bash
   npm i -g @nestjs/cli
   nest new meu-primeiro-projeto
   ```

2. **Estrutura criada**:
   ```
   meu-primeiro-projeto/
   ├── src/
   │   ├── app.controller.ts    # Controlador básico
   │   ├── app.service.ts       # Serviço básico
   │   ├── app.module.ts        # Módulo principal
   │   └── main.ts              # Ponto de entrada
   ```

3. **Primeiro endpoint**:
   ```typescript
   // app.controller.ts
   @Controller()
   export class AppController {
     constructor(private appService: AppService) {}

     @Get('hello')
     getHello(): string {
       return this.appService.getHello();
     }
   }
   ```

## 5. Exercício Prático
Vamos criar um endpoint que:
1. Recebe um nome via GET
2. Retorna uma mensagem personalizada

```typescript
// app.controller.ts
@Controller()
export class AppController {
  @Get('saudacao/:nome')
  dizOla(@Param('nome') nome: string) {
    return `Olá, ${nome}! Bem-vindo ao NestJS!`;
  }
}
```
