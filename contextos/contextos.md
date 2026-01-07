# Gestão de Contextos em Desenvolvimento com IA

## Objetivo da Seção
Explorar como Large Language Models trabalham com contexto, estratégias para otimizar o fornecimento de informações, e técnicas para superar limitações de janela de contexto no desenvolvimento de software.

---

## 1. Fundamentos de Contexto em LLMs

### 1.1 O que é Contexto?
**Definição**: Todo o conjunto de informações que um LLM considera ao gerar uma resposta, incluindo:
- Prompt do usuário
- Histórico da conversa
- Documentos ou código fornecidos
- System prompts (instruções base)

### 1.2 Janela de Contexto (Context Window)
**Definição**: Quantidade máxima de tokens que um modelo pode processar de uma vez.

**Comparativo de modelos**:
| Modelo | Janela de Contexto | Tokens (aprox.) | Equivalente em Código |
|--------|-------------------|-----------------|----------------------|
| GPT-3.5 | 4K | ~4,000 | ~500 linhas |
| GPT-4 | 8K / 32K | ~8,000 / ~32,000 | ~1,000 / ~4,000 linhas |
| GPT-4 Turbo | 128K | ~128,000 | ~16,000 linhas |
| Claude 3 | 200K | ~200,000 | ~25,000 linhas |
| Gemini 1.5 Pro | 1M | ~1,000,000 | ~125,000 linhas |

**Implicações práticas**:
- Arquivos grandes precisam ser divididos
- Histórico de conversação ocupa espaço
- Trade-off entre contexto e velocidade/custo

### 1.3 Como LLMs Processam Contexto

#### Mecanismo de Atenção
- **Self-attention**: Modelo relaciona diferentes partes do contexto
- **Degradação posicional**: Informações distantes podem ter menos peso
- **Contexto relevante vs. ruído**: Importância de filtrar informação

#### Limitações Cognitivas
```
[Início do contexto]
↓
Mais atenção/relevância
↓
[Prompt do usuário]
↓
Mais atenção/relevância
↓
[Fim do contexto]
```

**Estratégia**: Colocar informações críticas no início ou próximo ao prompt.

---

## 2. Estratégias de Otimização de Contexto

### 2.1 Princípio da Relevância
**Regra de ouro**: Forneça apenas o que é diretamente relevante para a tarefa.

#### ✅ BOM:
```markdown
Analise a função `processPayment` no arquivo abaixo:

```typescript
// payment-service.ts (linhas 45-78)
export async function processPayment(order: Order): Promise<PaymentResult> {
  // [código específico]
}
```

Contexto: Sistema de e-commerce, integração com Stripe
Problema: Pagamentos falhando intermitentemente
```

#### ❌ RUIM:
```markdown
Analise esse arquivo:

[arquivo inteiro de 2000 linhas, incluindo imports, comentários, código não relacionado]
```

### 2.2 Hierarquia de Informação

**Ordem de prioridade**:
1. **Tarefa específica** (o que precisa ser feito)
2. **Código/dados diretamente relacionados**
3. **Contexto de negócio essencial**
4. **Arquitetura/padrões do projeto**
5. **Informações de background** (quando necessário)

**Exemplo de estruturação**:
```markdown
# TAREFA
Corrigir bug de autenticação

# CÓDIGO AFETADO
[trecho específico com bug]

# CONTEXTO TÉCNICO
- Framework: Express.js + Passport
- Fluxo: JWT com refresh tokens
- Database: PostgreSQL

# COMPORTAMENTO ESPERADO
[descrição]

# COMPORTAMENTO ATUAL (BUG)
[descrição + logs relevantes]

# ARQUITETURA (se relevante)
[diagrama ou descrição breve]
```

### 2.3 Chunking de Código

**Quando usar**: Ao trabalhar com arquivos grandes que excedem janela de contexto.

#### Técnica 1: Chunking por Função/Classe
```markdown
Analise cada função separadamente:

## Análise 1 - Função `authenticate`
[código da função + dependências diretas]

## Análise 2 - Função `authorize`
[código da função + dependências diretas]
```

#### Técnica 2: Chunking por Fluxo
```markdown
Trace o fluxo de execução:

## Passo 1: Request recebida
[código do handler]

## Passo 2: Validação
[código de validação]

## Passo 3: Processamento
[código de negócio]
```

#### Técnica 3: Chunking com Resumos
```markdown
## Contexto Geral (resumo)
- Módulo A: Faz X, Y, Z
- Módulo B: Faz W, usa Módulo A
- Módulo C: Faz V, coordena A e B

## Análise Detalhada (Módulo B)
[código completo do Módulo B]
[apenas interfaces/tipos dos Módulos A e C]
```

### 2.4 Uso de Símbolos e Abstrações

**Estratégia**: Substituir código repetitivo por representações abstratas.

#### Exemplo:
```markdown
Em vez de mostrar todo o código boilerplate:

// Resumo de endpoints existentes
- POST /api/users -> createUser (validação + DB insert)
- GET /api/users/:id -> getUser (fetch + serialização)
- PUT /api/users/:id -> updateUser (validação + DB update)

Agora implemente o endpoint:
- DELETE /api/users/:id -> deleteUser

Seguindo os mesmos padrões de validação e error handling.
```

### 2.5 Contexto Incremental

**Técnica**: Construir contexto progressivamente através de múltiplas interações.

```
Interação 1:
"Descreva a arquitetura geral do módulo de autenticação"
→ [resposta com visão geral]

Interação 2:
"Agora detalhe a implementação do JWT refresh token"
→ [IA já tem contexto da Interação 1 + novo foco]

Interação 3:
"Como integrar isso com o middleware de autorização?"
→ [contexto acumulado + nova tarefa]
```

**Benefícios**:
- Não excede limites de contexto
- IA mantém histórico relevante
- Permite refinamento progressivo

---

## 3. Técnicas Avançadas de Contextualização

### 3.1 Contexto via RAG (Retrieval-Augmented Generation)

**Arquitetura**:
```
[Base de Conhecimento: Código + Docs]
           ↓
    [Vector Database]
           ↓
[Usuário faz pergunta] → [Sistema recupera trechos relevantes]
           ↓
[Trechos + Pergunta] → [LLM] → [Resposta contextualizada]
```

**Implementação prática**:
```typescript
// Exemplo conceitual
async function queryCodebase(question: string) {
  // 1. Converte pergunta em embedding
  const queryEmbedding = await embeddings.create(question);
  
  // 2. Busca código similar
  const relevantCode = await vectorDB.similaritySearch(
    queryEmbedding,
    topK: 5
  );
  
  // 3. Constrói prompt com contexto recuperado
  const prompt = `
    Contexto relevante do codebase:
    ${relevantCode.map(c => c.content).join('\n---\n')}
    
    Pergunta: ${question}
  `;
  
  // 4. Consulta LLM
  return await llm.generate(prompt);
}
```

**Casos de uso**:
- Responder perguntas sobre codebase grande
- Encontrar padrões de implementação
- Localizar código relacionado

### 3.2 Contexto via Project Knowledge

**Estratégia**: Criar "resumos executivos" do projeto.

**Exemplo de arquivo `PROJECT_CONTEXT.md`**:
```markdown
# Contexto do Projeto

## Visão Geral
E-commerce platform para B2B com foco em indústria farmacêutica.

## Stack Tecnológica
- Backend: Node.js + TypeScript + Express
- Frontend: React + TypeScript + Vite
- Database: PostgreSQL + Prisma ORM
- Cache: Redis
- Message Queue: RabbitMQ

## Arquitetura
- Clean Architecture (Domain-Driven Design)
- Microsserviços (Services: Auth, Orders, Inventory, Payments)
- Event-driven communication

## Padrões de Código
- Functional programming preferido
- Imutabilidade enforçada
- Error handling via Result types (no throw)
- Testes: Jest (unit) + Playwright (E2E)

## Convenções
- Nomenclatura: camelCase (variáveis), PascalCase (tipos)
- Arquivos: kebab-case
- Commits: Conventional Commits

## Integrações Externas
- Payment: Stripe
- Email: SendGrid
- Auth: Próprio (JWT)
- File storage: AWS S3

## Glossário de Negócio
- SKU: Stock Keeping Unit (produto único)
- Consignment: Pedido com retirada programada
- Regulatory Compliance: Validações específicas da Anvisa
```

**Uso**:
```markdown
[Cole conteúdo de PROJECT_CONTEXT.md]

Agora, seguindo os padrões do projeto, implemente [tarefa].
```

### 3.3 Contexto via Tipo e Interface

**Estratégia**: Fornecer definições de tipos em vez de implementações completas.

```typescript
// Em vez de mostrar implementações inteiras, forneça:

// types/user.ts
interface User {
  id: string;
  email: string;
  role: 'admin' | 'user' | 'guest';
  createdAt: Date;
}

// types/order.ts
interface Order {
  id: string;
  userId: string;
  items: OrderItem[];
  status: OrderStatus;
  total: number;
}

// Agora peça:
"Implemente a função que calcula o total de um pedido,
considerando descontos por role do usuário."
```

**Benefícios**:
- Menos tokens consumidos
- Foco no contrato, não na implementação
- IA infere comportamento esperado

### 3.4 Contexto via Testes

**Estratégia**: Testes como especificação de comportamento.

```markdown
Implemente a função que satisfaz estes testes:

```typescript
describe('calculateShipping', () => {
  it('should return 0 for orders over $100', () => {
    expect(calculateShipping(150, 'standard')).toBe(0);
  });
  
  it('should return $10 for standard shipping under $100', () => {
    expect(calculateShipping(50, 'standard')).toBe(10);
  });
  
  it('should return $25 for express shipping under $100', () => {
    expect(calculateShipping(50, 'express')).toBe(25);
  });
});
```

Implemente a função seguindo TDD.
```

**Vantagens**:
- Especificação precisa sem ambiguidade
- IA entende edge cases
- Resultado validável automaticamente

---

## 4. Gestão de Contexto em Diferentes Cenários

### 4.1 Debugging

**Contexto essencial**:
1. Mensagem de erro completa (stack trace)
2. Código onde erro ocorre
3. Dados de entrada que causam erro
4. Comportamento esperado

**Template**:
```markdown
## Erro
[stack trace completo]

## Código
[função/método específico, não o arquivo inteiro]

## Reprodução
Input: [dados]
Esperado: [comportamento]
Atual: [erro]

## Ambiente
- Node version: X
- Dependencies: [relevantes]
- Environment: [dev/prod]
```

### 4.2 Feature Implementation

**Contexto essencial**:
1. Requisitos funcionais
2. Padrões arquiteturais do projeto
3. Interfaces existentes a serem usadas
4. Restrições técnicas

**Template**:
```markdown
## Requisito
[user story ou descrição]

## Arquitetura Existente
[diagrama ou descrição dos módulos relacionados]

## Interfaces Disponíveis
[tipos/contratos a serem usados]

## Restrições
- Performance: [requisitos]
- Segurança: [considerações]
- Compatibilidade: [versões/browsers]

## Critérios de Aceitação
[lista de requisitos testáveis]
```

### 4.3 Refatoração

**Contexto essencial**:
1. Código atual a ser refatorado
2. Code smells identificados
3. Objetivo da refatoração
4. Restrições (manter API, performance, etc.)

**Template**:
```markdown
## Código Atual
[código a refatorar]

## Problemas Identificados
- [smell 1]: [explicação]
- [smell 2]: [explicação]

## Objetivos
1. [objetivo 1]
2. [objetivo 2]

## Restrições
- ✓ Manter comportamento externo idêntico
- ✓ Manter compatibilidade com API pública
- ✗ Não quebrar testes existentes

## Padrões Desejados
[princípios SOLID, patterns, etc.]
```

### 4.4 Code Review

**Contexto essencial**:
1. Código a ser revisado
2. Objetivo da mudança
3. Padrões do projeto
4. Checklist de review

**Template**:
```markdown
## Pull Request Context
- Feature: [descrição]
- Changed files: [lista]

## Código Principal
[diff ou código completo das mudanças principais]

## Checklist de Review
- [ ] Segue padrões do projeto
- [ ] Testes adequados
- [ ] Performance aceitável
- [ ] Sem vulnerabilidades
- [ ] Documentação atualizada

## Foco da Revisão
[aspectos específicos a serem avaliados]
```

---

## 5. Ferramentas e Técnicas de Gerenciamento

### 5.1 Context Window Management Tools

**PromptLayer**: Rastreia uso de tokens e otimiza prompts
```bash
npm install promptlayer
```

**Tiktoken**: Conta tokens antes de enviar
```python
import tiktoken

encoding = tiktoken.encoding_for_model("gpt-4")
tokens = encoding.encode("seu texto aqui")
print(f"Total de tokens: {len(tokens)}")
```

### 5.2 Codebase Indexing

**Embeddings para busca semântica**:
```typescript
// Exemplo com LangChain
import { OpenAIEmbeddings } from "langchain/embeddings/openai";
import { Chroma } from "langchain/vectorstores/chroma";

// 1. Indexar codebase
const embeddings = new OpenAIEmbeddings();
const vectorStore = await Chroma.fromDocuments(
  codebaseDocuments,
  embeddings
);

// 2. Buscar contexto relevante
const relevantDocs = await vectorStore.similaritySearch(
  "como fazer autenticação?",
  k: 3
);
```

### 5.3 Automated Context Builder

**Exemplo de script helper**:
```typescript
// context-builder.ts
async function buildContextForTask(task: string, codebasePath: string) {
  // 1. Identificar arquivos relacionados
  const relatedFiles = await findRelatedFiles(task, codebasePath);
  
  // 2. Extrair símbolos relevantes
  const symbols = await extractSymbols(relatedFiles);
  
  // 3. Gerar resumo de dependências
  const depSummary = await summarizeDependencies(symbols);
  
  // 4. Construir contexto otimizado
  return `
    ## Tarefa
    ${task}
    
    ## Arquivos Relacionados
    ${relatedFiles.map(f => `- ${f.path}: ${f.summary}`).join('\n')}
    
    ## Símbolos Principais
    ${symbols.map(s => s.signature).join('\n')}
    
    ## Dependências
    ${depSummary}
  `;
}
```

---

## 6. Best Practices e Anti-patterns

### ✅ Best Practices

1. **Context Filtering**: Remova noise (comentários irrelevantes, código não relacionado)
2. **Progressive Disclosure**: Adicione contexto gradualmente
3. **Semantic Chunking**: Divida por conceitos lógicos, não arbitrariamente
4. **Context Reuse**: Crie contextos reutilizáveis para tarefas similares
5. **Token Budgeting**: Reserve espaço para resposta (20-30% da janela)

### ❌ Anti-patterns

1. **Context Dumping**: Jogar todo o arquivo sem filtrar
2. **Excessive History**: Manter histórico de conversação irrelevante
3. **Redundant Information**: Repetir mesmas informações em diferentes formatos
4. **Missing Critical Context**: Omitir informações essenciais para economia
5. **Over-abstraction**: Resumir tanto que perde informação crítica

---

## 7. Casos de Estudo

### Caso 1: Debugging em Codebase Grande
**Cenário**: Sistema com 50K linhas, erro intermitente

**Abordagem ineficiente**:
- Colar arquivos inteiros
- Histórico de tentativas anteriores
- Logs completos de execução
→ Excede contexto, IA perde foco

**Abordagem eficiente**:
1. Isolar trecho específico com erro (50 linhas)
2. Incluir apenas stack trace relevante
3. Definições de tipos usados
4. Casos de teste que falham
→ Contexto focado, solução rápida

### Caso 2: Implementação de Feature com RAG
[A ser desenvolvido]

### Caso 3: Refatoração Guiada por Contexto Incremental
[A ser desenvolvido]

---

## 8. Métricas e Monitoramento

### Métricas de Eficiência de Contexto
- **Token Efficiency Ratio**: Output quality / tokens used
- **Context Relevance Score**: % de contexto efetivamente usado
- **Resolution Rate**: Problemas resolvidos na primeira tentativa
- **Iteration Count**: Número de prompts para completar tarefa

### Ferramentas de Monitoramento
- **LangSmith**: Trace de execução e uso de contexto
- **Helicone**: Analytics de custos e performance
- **PromptLayer**: Versionamento e A/B testing de prompts

---

## Referências

- Anthropic. (2024). "Long Context Windows: Use Cases and Optimization"
- OpenAI. (2023). "Managing Context in GPT-4"
- Lewis et al. (2020). "Retrieval-Augmented Generation for Knowledge-Intensive NLP"

---

## Notas para Desenvolvimento
- [ ] Adicionar exemplos práticos com código executável
- [ ] Criar visualizações de uso de tokens
- [ ] Expandir casos de estudo com métricas reais
- [ ] Adicionar seção sobre context compression techniques
- [ ] Incluir comparativo de performance entre abordagens
- [ ] Desenvolver ferramentas de demo/playground
