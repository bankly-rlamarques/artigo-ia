# Uso Prático de IA no Desenvolvimento de Software

## Objetivo da Seção
Demonstrar aplicações práticas de IA no desenvolvimento de software através de exemplos concretos, tutoriais passo a passo e casos de uso reais, cobrindo desde tarefas simples até workflows complexos.

---

## 1. Setup e Preparação do Ambiente

### 1.1 Ferramentas Essenciais

#### GitHub Copilot
```bash
# Instalação (VSCode)
# 1. Instalar extensão: GitHub.copilot
# 2. Autenticar com conta GitHub
# 3. Configurar settings.json

{
  "github.copilot.enable": {
    "*": true,
    "yaml": true,
    "plaintext": false,
    "markdown": true
  },
  "editor.inlineSuggest.enabled": true
}
```

#### Cursor
```bash
# Download e instalação
# https://cursor.sh

# Atalhos essenciais:
# Cmd+K: Editar código com IA
# Cmd+L: Chat com contexto do arquivo
# Cmd+Shift+L: Chat com contexto do projeto
```

#### ChatGPT/Claude (via API)
```typescript
// Instalação do SDK
npm install openai
npm install @anthropic-ai/sdk

// Configuração básica
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

// Exemplo de uso
const response = await openai.chat.completions.create({
  model: "gpt-4",
  messages: [
    { role: "system", content: "Você é um assistente de programação." },
    { role: "user", content: "Explique async/await em JavaScript" }
  ]
});
```

### 1.2 Configuração de Workspace

**Arquivo `.copilot-context.md`**:
```markdown
# Contexto do Projeto para IA

## Stack
- Backend: Node.js + TypeScript + Express
- Frontend: React + TypeScript
- Database: PostgreSQL + Prisma

## Padrões de Código
- Functional programming
- Error handling via Result types
- Naming: camelCase (vars), PascalCase (types)

## Comandos Úteis
- `npm run dev`: Inicia dev server
- `npm test`: Executa testes
- `npm run build`: Build de produção
```

---

## 2. Casos de Uso Básicos

### 2.1 Geração de Código Simples

#### Exemplo 1: Função de Utilidade
**Prompt**:
```
Crie uma função TypeScript que converte uma string snake_case para camelCase.
Inclua:
- Type hints
- Docstring
- Tratamento de edge cases
- Testes unitários (Jest)
```

**Resultado**:
```typescript
/**
 * Converte uma string de snake_case para camelCase
 * @param str - String em snake_case a ser convertida
 * @returns String convertida para camelCase
 * @example
 * snakeToCamel('hello_world') // 'helloWorld'
 */
export function snakeToCamel(str: string): string {
  if (!str || typeof str !== 'string') {
    return '';
  }
  
  return str
    .toLowerCase()
    .replace(/_([a-z])/g, (_, letter) => letter.toUpperCase());
}

// Testes
describe('snakeToCamel', () => {
  it('should convert snake_case to camelCase', () => {
    expect(snakeToCamel('hello_world')).toBe('helloWorld');
  });
  
  it('should handle single word', () => {
    expect(snakeToCamel('hello')).toBe('hello');
  });
  
  it('should handle empty string', () => {
    expect(snakeToCamel('')).toBe('');
  });
  
  it('should handle invalid input', () => {
    expect(snakeToCamel(null as any)).toBe('');
  });
});
```

#### Exemplo 2: CRUD Básico
**Prompt**:
```
Crie um serviço CRUD para entidade User com Prisma ORM.

Interface User:
- id: string (UUID)
- email: string (unique)
- name: string
- createdAt: Date

Requisitos:
- Validação de email
- Error handling com Result type
- TypeScript strict mode
```

**Resultado**:
```typescript
// types/result.ts
export type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

// services/user.service.ts
import { PrismaClient, User } from '@prisma/client';

const prisma = new PrismaClient();

// Validação de email
const isValidEmail = (email: string): boolean => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};

export class UserService {
  async create(data: {
    email: string;
    name: string;
  }): Promise<Result<User>> {
    try {
      if (!isValidEmail(data.email)) {
        return {
          success: false,
          error: new Error('Invalid email format')
        };
      }

      const user = await prisma.user.create({
        data: {
          email: data.email,
          name: data.name
        }
      });

      return { success: true, data: user };
    } catch (error) {
      if (error.code === 'P2002') {
        return {
          success: false,
          error: new Error('Email already exists')
        };
      }
      
      return {
        success: false,
        error: error as Error
      };
    }
  }

  async findById(id: string): Promise<Result<User | null>> {
    try {
      const user = await prisma.user.findUnique({
        where: { id }
      });

      return { success: true, data: user };
    } catch (error) {
      return {
        success: false,
        error: error as Error
      };
    }
  }

  async update(
    id: string,
    data: Partial<Pick<User, 'email' | 'name'>>
  ): Promise<Result<User>> {
    try {
      if (data.email && !isValidEmail(data.email)) {
        return {
          success: false,
          error: new Error('Invalid email format')
        };
      }

      const user = await prisma.user.update({
        where: { id },
        data
      });

      return { success: true, data: user };
    } catch (error) {
      return {
        success: false,
        error: error as Error
      };
    }
  }

  async delete(id: string): Promise<Result<void>> {
    try {
      await prisma.user.delete({
        where: { id }
      });

      return { success: true, data: undefined };
    } catch (error) {
      return {
        success: false,
        error: error as Error
      };
    }
  }

  async list(params?: {
    skip?: number;
    take?: number;
  }): Promise<Result<User[]>> {
    try {
      const users = await prisma.user.findMany({
        skip: params?.skip ?? 0,
        take: params?.take ?? 10,
        orderBy: {
          createdAt: 'desc'
        }
      });

      return { success: true, data: users };
    } catch (error) {
      return {
        success: false,
        error: error as Error
      };
    }
  }
}
```

### 2.2 Geração de Testes

#### Exemplo: Testes para Lógica de Negócio
**Contexto**:
```typescript
// discount.service.ts
export class DiscountService {
  calculateDiscount(
    price: number,
    userType: 'regular' | 'premium' | 'vip',
    quantity: number
  ): number {
    let discount = 0;
    
    // Desconto por tipo de usuário
    if (userType === 'premium') discount += 0.1;
    if (userType === 'vip') discount += 0.2;
    
    // Desconto por quantidade
    if (quantity >= 10) discount += 0.05;
    if (quantity >= 50) discount += 0.1;
    
    // Desconto máximo de 30%
    discount = Math.min(discount, 0.3);
    
    return price * (1 - discount);
  }
}
```

**Prompt**:
```
Gere testes abrangentes para a classe DiscountService usando Jest.
Inclua:
- Casos de teste para cada tipo de usuário
- Casos de teste para diferentes quantidades
- Edge cases
- Testes de desconto máximo
```

**Resultado**:
```typescript
// discount.service.test.ts
import { DiscountService } from './discount.service';

describe('DiscountService', () => {
  let service: DiscountService;

  beforeEach(() => {
    service = new DiscountService();
  });

  describe('calculateDiscount', () => {
    describe('por tipo de usuário', () => {
      it('deve aplicar 0% de desconto para usuário regular', () => {
        const result = service.calculateDiscount(100, 'regular', 1);
        expect(result).toBe(100);
      });

      it('deve aplicar 10% de desconto para usuário premium', () => {
        const result = service.calculateDiscount(100, 'premium', 1);
        expect(result).toBe(90);
      });

      it('deve aplicar 20% de desconto para usuário VIP', () => {
        const result = service.calculateDiscount(100, 'vip', 1);
        expect(result).toBe(80);
      });
    });

    describe('por quantidade', () => {
      it('deve adicionar 5% de desconto para quantidade >= 10', () => {
        const result = service.calculateDiscount(100, 'regular', 10);
        expect(result).toBe(95);
      });

      it('deve adicionar 15% de desconto para quantidade >= 50', () => {
        const result = service.calculateDiscount(100, 'regular', 50);
        expect(result).toBe(85); // 5% + 10%
      });
    });

    describe('descontos combinados', () => {
      it('deve combinar desconto de usuário premium com quantidade >= 10', () => {
        const result = service.calculateDiscount(100, 'premium', 10);
        expect(result).toBe(85); // 10% + 5%
      });

      it('deve combinar desconto de usuário VIP com quantidade >= 50', () => {
        const result = service.calculateDiscount(100, 'vip', 50);
        // 20% + 5% + 10% = 35%, mas max é 30%
        expect(result).toBe(70);
      });
    });

    describe('desconto máximo', () => {
      it('deve limitar desconto a 30%', () => {
        const result = service.calculateDiscount(100, 'vip', 100);
        expect(result).toBe(70);
      });
    });

    describe('edge cases', () => {
      it('deve lidar com preço zero', () => {
        const result = service.calculateDiscount(0, 'vip', 50);
        expect(result).toBe(0);
      });

      it('deve lidar com quantidade zero', () => {
        const result = service.calculateDiscount(100, 'premium', 0);
        expect(result).toBe(90);
      });

      it('deve calcular corretamente com valores decimais', () => {
        const result = service.calculateDiscount(99.99, 'premium', 1);
        expect(result).toBeCloseTo(89.99, 2);
      });
    });
  });
});
```

### 2.3 Documentação Automática

#### Exemplo: Gerar Documentação de API
**Contexto**:
```typescript
// routes/users.routes.ts
router.post('/users', async (req, res) => {
  const { email, name, role } = req.body;
  // ... implementação
});

router.get('/users/:id', async (req, res) => {
  const { id } = req.params;
  // ... implementação
});
```

**Prompt**:
```
Gere documentação OpenAPI (Swagger) para os endpoints de usuário.
Inclua:
- Schemas de request/response
- Códigos de status
- Exemplos de uso
- Descrições detalhadas
```

**Resultado**:
```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
  description: API para gerenciamento de usuários

paths:
  /users:
    post:
      summary: Cria um novo usuário
      description: Endpoint para criar um novo usuário no sistema
      tags:
        - Users
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - email
                - name
              properties:
                email:
                  type: string
                  format: email
                  example: "user@example.com"
                name:
                  type: string
                  example: "João Silva"
                role:
                  type: string
                  enum: [admin, user, guest]
                  default: user
                  example: "user"
      responses:
        '201':
          description: Usuário criado com sucesso
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          description: Dados inválidos
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
        '409':
          description: Email já cadastrado
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

  /users/{id}:
    get:
      summary: Busca usuário por ID
      description: Retorna os detalhes de um usuário específico
      tags:
        - Users
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
            format: uuid
          example: "123e4567-e89b-12d3-a456-426614174000"
      responses:
        '200':
          description: Usuário encontrado
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: Usuário não encontrado
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
          format: uuid
          example: "123e4567-e89b-12d3-a456-426614174000"
        email:
          type: string
          format: email
          example: "user@example.com"
        name:
          type: string
          example: "João Silva"
        role:
          type: string
          enum: [admin, user, guest]
          example: "user"
        createdAt:
          type: string
          format: date-time
          example: "2024-01-01T10:00:00Z"
    
    Error:
      type: object
      properties:
        message:
          type: string
          example: "Email inválido"
        code:
          type: string
          example: "INVALID_EMAIL"
```

---

## 3. Casos de Uso Intermediários

### 3.1 Refatoração Assistida por IA

#### Exemplo: Modernizar Código Legado
**Código Legado**:
```javascript
// user-controller.js (JavaScript antigo)
var UserController = function() {
  this.users = [];
};

UserController.prototype.addUser = function(user) {
  var self = this;
  if (!user.email) {
    throw new Error('Email required');
  }
  
  // Check duplicate
  for (var i = 0; i < self.users.length; i++) {
    if (self.users[i].email === user.email) {
      throw new Error('Email exists');
    }
  }
  
  self.users.push(user);
  return user;
};

UserController.prototype.getUser = function(email) {
  var self = this;
  for (var i = 0; i < self.users.length; i++) {
    if (self.users[i].email === email) {
      return self.users[i];
    }
  }
  return null;
};
```

**Prompt**:
```
Refatore o código legado para TypeScript moderno:
- Classes ES6+
- Async/await
- Type safety
- Functional programming (imutabilidade)
- Error handling com Result type
- Use Map para melhor performance de busca
```

**Resultado Refatorado**:
```typescript
// user-controller.ts
import { Result } from './types/result';

interface User {
  email: string;
  name: string;
  id: string;
}

export class UserController {
  private readonly users: Map<string, User>;

  constructor() {
    this.users = new Map();
  }

  async addUser(user: Omit<User, 'id'>): Promise<Result<User>> {
    // Validação
    if (!user.email || !this.isValidEmail(user.email)) {
      return {
        success: false,
        error: new Error('Invalid email')
      };
    }

    // Verificar duplicata
    if (this.users.has(user.email)) {
      return {
        success: false,
        error: new Error('Email already exists')
      };
    }

    // Criar usuário com ID
    const newUser: User = {
      ...user,
      id: crypto.randomUUID()
    };

    // Adicionar ao Map (imutável externamente)
    this.users.set(newUser.email, newUser);

    return {
      success: true,
      data: newUser
    };
  }

  async getUser(email: string): Promise<Result<User | null>> {
    try {
      const user = this.users.get(email) ?? null;
      return {
        success: true,
        data: user
      };
    } catch (error) {
      return {
        success: false,
        error: error as Error
      };
    }
  }

  async listUsers(): Promise<Result<User[]>> {
    try {
      const userList = Array.from(this.users.values());
      return {
        success: true,
        data: userList
      };
    } catch (error) {
      return {
        success: false,
        error: error as Error
      };
    }
  }

  private isValidEmail(email: string): boolean {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }
}

// Testes
describe('UserController', () => {
  let controller: UserController;

  beforeEach(() => {
    controller = new UserController();
  });

  describe('addUser', () => {
    it('should add user successfully', async () => {
      const result = await controller.addUser({
        email: 'test@example.com',
        name: 'Test User'
      });

      expect(result.success).toBe(true);
      if (result.success) {
        expect(result.data.email).toBe('test@example.com');
        expect(result.data.id).toBeDefined();
      }
    });

    it('should reject invalid email', async () => {
      const result = await controller.addUser({
        email: 'invalid-email',
        name: 'Test User'
      });

      expect(result.success).toBe(false);
    });

    it('should reject duplicate email', async () => {
      await controller.addUser({
        email: 'test@example.com',
        name: 'User 1'
      });

      const result = await controller.addUser({
        email: 'test@example.com',
        name: 'User 2'
      });

      expect(result.success).toBe(false);
    });
  });
});
```

### 3.2 Debugging Assistido

#### Exemplo: Debugging de Async/Concurrency
**Código com Bug**:
```typescript
// order-processor.ts
async function processOrders(orderIds: string[]) {
  const results = [];
  
  for (const id of orderIds) {
    const order = await fetchOrder(id);
    const validated = await validateOrder(order);
    const processed = await processPayment(order);
    results.push(processed);
  }
  
  return results;
}

// Problema: Lento, processa sequencialmente
// 100 pedidos x 1s cada = 100s total
```

**Prompt para Debugging**:
```
O código acima está demorando muito (100s para 100 pedidos).
Analise e otimize para processar em paralelo, mas:
- Limite concorrência a 10 requests simultâneos
- Mantenha ordem dos resultados
- Adicione tratamento de erro individual
- Inclua retry logic
```

**Solução Otimizada**:
```typescript
// order-processor.optimized.ts
import pLimit from 'p-limit';

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: Error;
  orderId: string;
}

async function processOrdersOptimized(
  orderIds: string[]
): Promise<ProcessResult<Order>[]> {
  // Limitar concorrência
  const limit = pLimit(10);

  // Processar em paralelo com limite
  const promises = orderIds.map((id, index) =>
    limit(async () => {
      try {
        // Retry logic
        return await retryWithBackoff(
          async () => {
            const order = await fetchOrder(id);
            const validated = await validateOrder(order);
            const processed = await processPayment(order);
            return processed;
          },
          { maxRetries: 3, initialDelay: 1000 }
        );
      } catch (error) {
        return {
          success: false,
          error: error as Error,
          orderId: id
        };
      }
    })
  );

  const results = await Promise.all(promises);
  
  return results.map((result, index) => ({
    success: result.success !== false,
    data: result.success !== false ? result : undefined,
    error: result.error,
    orderId: orderIds[index]
  }));
}

// Utility: Retry com backoff exponencial
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  options: {
    maxRetries: number;
    initialDelay: number;
  }
): Promise<T> {
  let lastError: Error;
  
  for (let attempt = 0; attempt <= options.maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      
      if (attempt < options.maxRetries) {
        const delay = options.initialDelay * Math.pow(2, attempt);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }
  
  throw lastError!;
}

// Resultado: ~10s para 100 pedidos (10x mais rápido)
```

### 3.3 Geração de Schemas e Validação

#### Exemplo: Gerar Schemas Zod a partir de TypeScript
**Tipos TypeScript Existentes**:
```typescript
interface CreateUserDTO {
  email: string;
  name: string;
  age?: number;
  role: 'admin' | 'user' | 'guest';
  preferences?: {
    newsletter: boolean;
    notifications: boolean;
  };
}
```

**Prompt**:
```
Gere schemas Zod de validação para CreateUserDTO com:
- Validação de email
- Nome com min 2 caracteres
- Idade entre 18 e 120 (se fornecida)
- Mensagens de erro customizadas em PT-BR
```

**Resultado**:
```typescript
import { z } from 'zod';

export const createUserSchema = z.object({
  email: z
    .string({
      required_error: 'Email é obrigatório',
      invalid_type_error: 'Email deve ser uma string'
    })
    .email('Email inválido')
    .toLowerCase()
    .trim(),
  
  name: z
    .string({
      required_error: 'Nome é obrigatório'
    })
    .min(2, 'Nome deve ter no mínimo 2 caracteres')
    .max(100, 'Nome deve ter no máximo 100 caracteres')
    .trim(),
  
  age: z
    .number()
    .int('Idade deve ser um número inteiro')
    .min(18, 'Idade mínima é 18 anos')
    .max(120, 'Idade máxima é 120 anos')
    .optional(),
  
  role: z
    .enum(['admin', 'user', 'guest'], {
      errorMap: () => ({ message: 'Role deve ser admin, user ou guest' })
    }),
  
  preferences: z
    .object({
      newsletter: z.boolean().default(false),
      notifications: z.boolean().default(true)
    })
    .optional()
});

export type CreateUserDTO = z.infer<typeof createUserSchema>;

// Uso em controller
export async function createUser(req: Request, res: Response) {
  try {
    const validatedData = createUserSchema.parse(req.body);
    // ... continuar com dados validados
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({
        message: 'Dados inválidos',
        errors: error.errors.map(e => ({
          field: e.path.join('.'),
          message: e.message
        }))
      });
    }
    throw error;
  }
}
```

---

## 4. Casos de Uso Avançados

### 4.1 Geração de Testes E2E com Playwright

**Prompt**:
```
Gere testes E2E com Playwright para fluxo de autenticação:

Fluxo:
1. Usuário acessa /login
2. Preenche email e senha
3. Clica em "Entrar"
4. Redireciona para /dashboard
5. Verifica presença do nome do usuário

Inclua:
- Teste de sucesso
- Teste de credenciais inválidas
- Teste de campos vazios
- Page Object Pattern
```

**Resultado**:
```typescript
// pages/login.page.ts
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.locator('[data-testid="email-input"]');
    this.passwordInput = page.locator('[data-testid="password-input"]');
    this.submitButton = page.locator('[data-testid="submit-button"]');
    this.errorMessage = page.locator('[data-testid="error-message"]');
  }

  async navigate() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async getErrorMessage(): Promise<string> {
    return await this.errorMessage.textContent() || '';
  }
}

// pages/dashboard.page.ts
export class DashboardPage {
  readonly page: Page;
  readonly userNameDisplay: Locator;

  constructor(page: Page) {
    this.page = page;
    this.userNameDisplay = page.locator('[data-testid="user-name"]');
  }

  async getUserName(): Promise<string> {
    return await this.userNameDisplay.textContent() || '';
  }

  async isLoaded(): Promise<boolean> {
    await this.page.waitForURL('/dashboard');
    return this.userNameDisplay.isVisible();
  }
}

// tests/auth.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from './pages/login.page';
import { DashboardPage } from './pages/dashboard.page';

test.describe('Autenticação', () => {
  let loginPage: LoginPage;
  let dashboardPage: DashboardPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    dashboardPage = new DashboardPage(page);
    await loginPage.navigate();
  });

  test('deve fazer login com sucesso', async ({ page }) => {
    await loginPage.login('user@example.com', 'password123');
    
    // Verificar redirecionamento
    await expect(page).toHaveURL('/dashboard');
    
    // Verificar dashboard carregado
    await expect(dashboardPage.userNameDisplay).toBeVisible();
    
    // Verificar nome do usuário
    const userName = await dashboardPage.getUserName();
    expect(userName).toBe('João Silva');
  });

  test('deve mostrar erro com credenciais inválidas', async ({ page }) => {
    await loginPage.login('wrong@example.com', 'wrongpassword');
    
    // Deve permanecer na página de login
    await expect(page).toHaveURL('/login');
    
    // Deve mostrar mensagem de erro
    await expect(loginPage.errorMessage).toBeVisible();
    const errorText = await loginPage.getErrorMessage();
    expect(errorText).toContain('Credenciais inválidas');
  });

  test('deve validar campos obrigatórios', async ({ page }) => {
    await loginPage.submitButton.click();
    
    // Validação HTML5
    const emailValidity = await loginPage.emailInput.evaluate(
      (el: HTMLInputElement) => el.validity.valid
    );
    expect(emailValidity).toBe(false);
  });

  test('deve validar formato de email', async ({ page }) => {
    await loginPage.login('invalid-email', 'password123');
    
    // Verificar mensagem de validação
    const emailValidity = await loginPage.emailInput.evaluate(
      (el: HTMLInputElement) => el.validity.typeMismatch
    );
    expect(emailValidity).toBe(true);
  });
});
```

### 4.2 Implementação de Arquitetura Complexa

**Prompt**:
```
Implemente um sistema de Event Sourcing para orders:

Requisitos:
- Event Store (PostgreSQL)
- Agregados (Order)
- Comandos (CreateOrder, CancelOrder, CompleteOrder)
- Eventos (OrderCreated, OrderCancelled, OrderCompleted)
- Projeções (OrderSummary para queries)
- TypeScript + DDD patterns
```

**Resultado** (resumido por espaço):
```typescript
// domain/events.ts
export interface DomainEvent {
  aggregateId: string;
  type: string;
  data: unknown;
  timestamp: Date;
  version: number;
}

export interface OrderCreatedEvent extends DomainEvent {
  type: 'OrderCreated';
  data: {
    userId: string;
    items: Array<{ productId: string; quantity: number; price: number }>;
    total: number;
  };
}

// domain/order.aggregate.ts
export class Order {
  private id: string;
  private version: number = 0;
  private status: 'pending' | 'completed' | 'cancelled' = 'pending';
  private uncommittedEvents: DomainEvent[] = [];

  static create(userId: string, items: OrderItem[]): Order {
    const order = new Order();
    order.id = crypto.randomUUID();
    
    const event: OrderCreatedEvent = {
      aggregateId: order.id,
      type: 'OrderCreated',
      data: { userId, items, total: order.calculateTotal(items) },
      timestamp: new Date(),
      version: 1
    };
    
    order.apply(event);
    order.uncommittedEvents.push(event);
    
    return order;
  }

  cancel(): void {
    if (this.status !== 'pending') {
      throw new Error('Can only cancel pending orders');
    }
    
    const event: OrderCancelledEvent = {
      aggregateId: this.id,
      type: 'OrderCancelled',
      data: { reason: 'User requested' },
      timestamp: new Date(),
      version: this.version + 1
    };
    
    this.apply(event);
    this.uncommittedEvents.push(event);
  }

  private apply(event: DomainEvent): void {
    switch (event.type) {
      case 'OrderCreated':
        this.status = 'pending';
        this.version = event.version;
        break;
      case 'OrderCancelled':
        this.status = 'cancelled';
        this.version = event.version;
        break;
      // ... outros eventos
    }
  }

  getUncommittedEvents(): DomainEvent[] {
    return [...this.uncommittedEvents];
  }

  markEventsAsCommitted(): void {
    this.uncommittedEvents = [];
  }
}

// infrastructure/event-store.ts
export class EventStore {
  async save(events: DomainEvent[]): Promise<void> {
    await prisma.event.createMany({
      data: events.map(e => ({
        aggregateId: e.aggregateId,
        type: e.type,
        data: JSON.stringify(e.data),
        timestamp: e.timestamp,
        version: e.version
      }))
    });
  }

  async load(aggregateId: string): Promise<DomainEvent[]> {
    const events = await prisma.event.findMany({
      where: { aggregateId },
      orderBy: { version: 'asc' }
    });
    
    return events.map(e => ({
      aggregateId: e.aggregateId,
      type: e.type,
      data: JSON.parse(e.data as string),
      timestamp: e.timestamp,
      version: e.version
    }));
  }
}
```

---

## 5. Workflows e Automações

### 5.1 CI/CD com IA

**Exemplo**: Geração de pipeline GitHub Actions

**Prompt**:
```
Crie workflow GitHub Actions para projeto Node.js/TypeScript:

Etapas:
- Lint (ESLint)
- Tests (Jest) com coverage
- Build
- Deploy para staging (se branch main)
- Deploy para produção (se tag)

Requisitos:
- Cache de dependências
- Matrix testing (Node 18, 20)
- Upload de coverage para Codecov
- Notificação no Slack
```

**Resultado**:
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  release:
    types: [published]

env:
  NODE_VERSION_MAIN: '20'

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION_MAIN }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run ESLint
        run: npm run lint

  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests with coverage
        run: npm run test:coverage
      
      - name: Upload coverage to Codecov
        if: matrix.node-version == 20
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/lcov.info

  build:
    runs-on: ubuntu-latest
    needs: [lint, test]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION_MAIN }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: dist
          path: dist/

  deploy-staging:
    runs-on: ubuntu-latest
    needs: [build]
    if: github.ref == 'refs/heads/main'
    environment:
      name: staging
      url: https://staging.example.com
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist
          path: dist/
      
      - name: Deploy to staging
        run: |
          # Deploy script aqui
          echo "Deploying to staging..."
      
      - name: Notify Slack
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "Deploy to staging completed for ${{ github.sha }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

  deploy-production:
    runs-on: ubuntu-latest
    needs: [build]
    if: github.event_name == 'release'
    environment:
      name: production
      url: https://example.com
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist
          path: dist/
      
      - name: Deploy to production
        run: |
          # Deploy script aqui
          echo "Deploying to production..."
      
      - name: Notify Slack
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "🚀 Deploy to production completed! Version: ${{ github.event.release.tag_name }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

---

## 6. Métricas e Avaliação

### Como Medir Eficácia do Uso de IA

**Métricas quantitativas**:
- **Time to Implementation**: Tempo para implementar features
- **Bug Density**: Bugs por 1000 linhas de código
- **Test Coverage**: Cobertura de testes alcançada
- **Code Review Time**: Tempo de revisão reduzido

**Métricas qualitativas**:
- **Code Maintainability**: Índice de manutenibilidade
- **Developer Satisfaction**: Satisfação da equipe
- **Learning Curve**: Tempo para onboarding

---

## Referências

- GitHub. (2024). "GitHub Copilot Documentation"
- Playwright. (2024). "Best Practices for E2E Testing"
- Martin Fowler. (2005). "Event Sourcing"

---

## Notas para Desenvolvimento
- [ ] Adicionar mais exemplos de diferentes linguagens
- [ ] Expandir seção de workflows com outros CI/CD tools
- [ ] Incluir vídeos demonstrativos
- [ ] Adicionar métricas de benchmarking reais
- [ ] Criar repositório de exemplos executáveis
