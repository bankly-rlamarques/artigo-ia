# Engenharia de Prompts para Desenvolvimento de Software

## Objetivo da Seção
Apresentar melhores práticas, técnicas avançadas e armadilhas comuns na formulação de prompts para assistentes de IA no contexto de desenvolvimento de software.

---

## 1. Fundamentos da Engenharia de Prompts

### 1.1 O que é um Bom Prompt?
Um prompt eficaz deve ser:
- **Específico**: Detalha exatamente o que é necessário
- **Contextualizado**: Fornece informações relevantes ao problema
- **Estruturado**: Organiza informações de forma lógica
- **Verificável**: Permite validar se a resposta atende ao requisito

### 1.2 Anatomia de um Prompt
```
[PAPEL/CONTEXTO]
Você é um desenvolvedor sênior especializado em Python...

[TAREFA]
Crie uma função que...

[RESTRIÇÕES]
- Use type hints
- Inclua docstrings
- Trate erros apropriadamente

[FORMATO DE SAÍDA]
Retorne apenas o código, sem explicações adicionais.

[EXEMPLOS] (opcional)
Exemplo de entrada: ...
Exemplo de saída: ...
```

---

## 2. Técnicas Avançadas

### 2.1 Chain-of-Thought (CoT)
**Definição**: Solicitar que o modelo explique seu raciocínio passo a passo.

**Exemplo**:
```markdown
Analise o seguinte código e identifique bugs:
[código]

Siga estes passos:
1. Leia o código linha por linha
2. Identifique potenciais problemas
3. Explique por que cada item é um problema
4. Sugira correções
```

**Quando usar**:
- Debugging complexo
- Análise de arquitetura
- Refatoração de código legado

### 2.2 Few-Shot Learning
**Definição**: Fornecer exemplos concretos do comportamento esperado.

**Exemplo**:
```markdown
Converta funções para TypeScript seguindo este padrão:

Exemplo 1:
JavaScript:
function sum(a, b) { return a + b; }

TypeScript:
function sum(a: number, b: number): number { return a + b; }

Exemplo 2:
JavaScript:
function getName(user) { return user.name; }

TypeScript:
function getName(user: User): string { return user.name; }

Agora converta:
[sua função aqui]
```

**Quando usar**:
- Padrões específicos do projeto
- Convenções de código personalizadas
- Transformações sistemáticas

### 2.3 Role Prompting
**Definição**: Atribuir um papel específico ao modelo.

**Exemplos**:
- "Você é um especialista em segurança de aplicações web..."
- "Como um revisor de código sênior..."
- "Atuando como arquiteto de sistemas..."

**Benefícios**:
- Respostas mais alinhadas ao contexto
- Terminologia apropriada
- Foco em aspectos específicos

### 2.4 Template Prompting
**Definição**: Criar templates reutilizáveis para tarefas recorrentes.

**Exemplo - Template para Code Review**:
```markdown
# Code Review Request

## Código a Revisar:
```[linguagem]
[código]
```

## Aspectos a Avaliar:
- [ ] Legibilidade e clareza
- [ ] Performance
- [ ] Segurança
- [ ] Testes
- [ ] Documentação

## Contexto do Projeto:
- Stack: [tecnologias]
- Padrões: [convenções]
- Requisitos: [específicos]

## Formato de Resposta:
1. Resumo geral
2. Pontos positivos
3. Pontos de melhoria (com exemplos de código)
4. Questões críticas (se houver)
```

### 2.5 Iterative Refinement
**Definição**: Refinar prompts progressivamente baseado nos resultados.

**Processo**:
1. Prompt inicial (genérico)
2. Avaliar resultado
3. Identificar lacunas
4. Adicionar restrições/contexto
5. Repetir até satisfatório

---

## 3. Melhores Práticas por Tipo de Tarefa

### 3.1 Geração de Código

#### ✅ BOM:
```markdown
Crie uma função Python que:
- Recebe uma lista de dicionários representando usuários
- Filtra usuários ativos (campo 'active' = True)
- Retorna lista ordenada por 'created_at' (mais recente primeiro)
- Use type hints e inclua docstring
- Trate caso de lista vazia

Exemplo de entrada:
[
  {"id": 1, "name": "Alice", "active": True, "created_at": "2024-01-01"},
  {"id": 2, "name": "Bob", "active": False, "created_at": "2024-01-02"}
]
```

#### ❌ RUIM:
```markdown
Faz uma função que filtra usuários
```

### 3.2 Debugging

#### ✅ BOM:
```markdown
O seguinte código está gerando erro "TypeError: 'NoneType' object is not iterable":

```python
def process_data(items):
    result = []
    for item in items:
        result.append(item * 2)
    return result

data = fetch_from_db()  # Pode retornar None
output = process_data(data)
```

Comportamento esperado: Processar lista de números duplicando cada valor

Contexto adicional:
- fetch_from_db() retorna None quando não há dados
- Preciso lidar com esse caso gracefully

Pergunta: Como corrigir e adicionar tratamento de erro adequado?
```

#### ❌ RUIM:
```markdown
Meu código não funciona, me ajuda?
[código sem contexto]
```

### 3.3 Refatoração

#### ✅ BOM:
```markdown
Refatore o código abaixo seguindo estes princípios:
- SOLID principles
- Clean Code
- Padrões do projeto (DDD)

Código atual:
[código]

Requisitos:
- Manter funcionalidade existente
- Melhorar testabilidade
- Separar concerns
- Adicionar type safety

Stack: TypeScript + Node.js + Express
```

#### ❌ RUIM:
```markdown
Melhora esse código
[código]
```

### 3.4 Documentação

#### ✅ BOM:
```markdown
Gere documentação técnica para a seguinte API:

[código da API]

Formato desejado:
- Descrição geral do endpoint
- Parâmetros (tipo, required/optional, descrição)
- Corpo da requisição (com exemplo JSON)
- Respostas possíveis (códigos de status + exemplos)
- Casos de erro
- Exemplo de uso com curl

Público-alvo: Desenvolvedores externos consumindo a API
```

#### ❌ RUIM:
```markdown
Documenta essa API
[código]
```

### 3.5 Testes

#### ✅ BOM:
```markdown
Crie testes unitários para a função abaixo usando Jest:

```typescript
function calculateDiscount(price: number, coupon?: string): number {
  if (!coupon) return price;
  if (coupon === 'SAVE10') return price * 0.9;
  if (coupon === 'SAVE20') return price * 0.8;
  return price;
}
```

Casos de teste necessários:
- [ ] Sem cupom fornecido
- [ ] Cupom válido SAVE10
- [ ] Cupom válido SAVE20
- [ ] Cupom inválido
- [ ] Edge cases (price = 0, valores negativos)

Padrão de teste do projeto:
- Usar describe/it blocks
- Mensagens descritivas
- Arrange-Act-Assert pattern
```

#### ❌ RUIM:
```markdown
Cria testes para essa função
[código]
```

---

## 4. Armadilhas Comuns e Como Evitar

### 4.1 Prompts Muito Vagos
**Problema**: Resultados genéricos ou irrelevantes

**Solução**: Seja específico sobre:
- Linguagem/framework
- Restrições técnicas
- Contexto do projeto
- Formato de saída

### 4.2 Falta de Contexto
**Problema**: IA não entende o cenário completo

**Solução**: Inclua:
- Stack tecnológica
- Padrões do projeto
- Restrições de negócio
- Código relacionado

### 4.3 Excesso de Informação
**Problema**: Supera limite de tokens ou confunde o modelo

**Solução**:
- Divida tarefas complexas
- Forneça apenas contexto relevante
- Use técnicas de chunking

### 4.4 Prompts Ambíguos
**Problema**: Múltiplas interpretações possíveis

**Solução**:
- Use terminologia precisa
- Forneça exemplos concretos
- Especifique formato de saída

### 4.5 Não Validar Resultados
**Problema**: Aceitar código gerado sem verificação

**Solução**:
- Sempre revisar código gerado
- Executar testes
- Verificar segurança e performance
- Validar contra requisitos

---

## 5. Padrões de Prompts para Desenvolvimento

### 5.1 Padrão: Implementação com TDD
```markdown
Seguindo TDD, implemente [funcionalidade]:

1. Primeiro, crie os testes que definem o comportamento esperado
2. Depois, implemente o código mínimo para passar os testes
3. Refatore mantendo os testes verdes

Requisitos:
[lista de requisitos]

Framework de testes: [Jest/PyTest/etc]
```

### 5.2 Padrão: Análise de Código Legado
```markdown
Analise o código legado abaixo e forneça:

1. Resumo do que o código faz
2. Identificação de code smells
3. Riscos e vulnerabilidades
4. Sugestões de refatoração (priorizadas)
5. Estimativa de complexidade para refatoração

Código:
[código legado]

Contexto: [informações sobre o sistema]
```

### 5.3 Padrão: Tradução de Requisitos
```markdown
Converta os requisitos de negócio abaixo em:
1. User stories técnicas
2. Critérios de aceitação
3. Proposta de arquitetura
4. Estimativa de esforço

Requisitos:
[descrição em linguagem natural]

Stack tecnológica: [tecnologias]
Restrições: [limitações]
```

### 5.4 Padrão: Otimização de Performance
```markdown
O código abaixo está com problemas de performance:

[código]

Perfil de uso:
- [dados sobre volume, frequência, etc]

Análise necessária:
1. Identificar gargalos
2. Propor otimizações específicas
3. Estimar impacto de cada otimização
4. Sugerir ferramentas de profiling

Restrições:
- [limitações técnicas ou de negócio]
```

---

## 6. Ferramentas e Recursos

### 6.1 Prompt Libraries
- **Awesome ChatGPT Prompts**: Repositório comunitário
- **PromptBase**: Marketplace de prompts
- **FlowGPT**: Compartilhamento de workflows

### 6.2 Testing de Prompts
- **PromptFoo**: Framework para testar prompts
- **LangSmith**: Debugging e monitoring
- **Helicone**: Analytics de prompts

### 6.3 Extensões e Integrações
- **GitHub Copilot**: Sugestões inline
- **Cursor**: IDE com prompts nativos
- **Continue**: Plugin open-source para VSCode

---

## 7. Métricas de Qualidade de Prompts

### Critérios de Avaliação:
1. **Clareza**: Prompt é inequívoco?
2. **Completude**: Todas as informações necessárias estão presentes?
3. **Eficiência**: É conciso sem perder contexto?
4. **Reprodutibilidade**: Gera resultados consistentes?
5. **Actionability**: Resulta em output utilizável?

### Checklist de Revisão:
- [ ] Objetivo claro e específico
- [ ] Contexto suficiente fornecido
- [ ] Restrições e requisitos explícitos
- [ ] Formato de saída especificado
- [ ] Exemplos incluídos (quando aplicável)
- [ ] Linguagem precisa e sem ambiguidade

---

## 8. Casos de Estudo

### Caso 1: Migração de Código Legado
**Desafio**: Migrar sistema PHP legado para Node.js

**Prompt inicial (ineficaz)**:
```
Converte esse código PHP para Node.js
[código]
```

**Prompt refinado (eficaz)**:
```
Migre a seguinte função PHP para Node.js/TypeScript:

[código PHP]

Requisitos:
- Use TypeScript com strict mode
- Mantenha a mesma lógica de negócio
- Adapte para padrões assíncronos (Promises)
- Use bibliotecas modernas equivalentes
- Adicione tratamento de erro adequado
- Inclua testes Jest

Contexto:
- Projeto usa Express.js
- Padrão: Clean Architecture
- Database: PostgreSQL com Prisma ORM
```

**Resultado**: Código TypeScript idiomático e bem estruturado

### Caso 2: Code Review Automatizado
[A ser desenvolvido]

### Caso 3: Geração de Testes E2E
[A ser desenvolvido]

---

## Referências

- OpenAI. (2023). "Best Practices for Prompt Engineering"
- Anthropic. (2024). "Prompt Engineering Guide"
- GitHub. (2024). "Copilot Prompt Engineering Documentation"

---

## Notas para Desenvolvimento
- [ ] Adicionar mais casos de estudo reais
- [ ] Incluir métricas quantitativas de melhoria
- [ ] Expandir seção de ferramentas com tutoriais
- [ ] Criar diagramas de fluxo para técnicas avançadas
- [ ] Adicionar comparativo de eficácia entre técnicas
- [ ] Incluir exemplos multilíngues (Python, Java, Go, etc.)
