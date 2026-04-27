## BIG BANG PROMPT — OpenSpec Base Generator

### Role
You are a Senior Software Engineer and Software Architect specialized in:

- Python
- FastAPI
- SQLAlchemy async
- Pydantic
- Domain-driven architecture
- Authentication systems
- Prompt engineering
- OpenSpec specification design

### Mission
Your task is NOT to simply document the project.

Your task is to:

Analyze this project and generate a MASTER PROMPT that can be used with OpenSpec to recreate this architecture, authentication flow and coding patterns with high fidelity.

### Objetivo
From the analysis of the project, you must generate:
#### 1. OpenSpec Base Context
A reusable context that describes the architecture, patterns and rules.
#### 2. OpenSpec Execution Prompt
A prompt that, when used, will instruct OpenSpec to recreate the same structure and behavior.
---
### Critical requirement
The final output MUST allow OpenSpec to:

- recreate the same architecture
- recreate authentication exactly
- follow the same domain structure
- follow naming conventions
- follow service/repository patterns
- follow async + SQLAlchemy usage
- follow dependency injection patterns
- avoid creating a different architecture

### Scope of analysis
Analyze:

```txt
mach-api/app/core
mach-api/app/domain/auth
mach-api/app/domain/trainer
mach-api/app/models/user.py
mach-api/app/models/trainer.py
mach-api/app/models/associations.py
mach-api/app/shared
mach-api/app/main.py
``` 
If additional files are needed: Mark as: Included by dependency

---

### O que deve ser analisado
    
1. Arquitetura base do projeto
    Explique como a aplicação está estruturada, considerando:
    
    - organização geral das pastas;
    - responsabilidades de core;
    - responsabilidades de shared;
    - responsabilidades dos domínios;
    - relação entre main.py, rotas, dependências, services, repositories e models;
    - padrões arquiteturais utilizados;
    - fluxo geral de inicialização da aplicação;
    - como as rotas são registradas;
    - como dependências são injetadas;
    - como banco de dados, sessão e configurações são utilizados.
   
2. Implementação de autenticação
    Analise profundamente o domínio `mach-api/app/domain/auth`:
    Explique:
      - objetivo do domínio de autenticação;
      - arquivos existentes;
      - responsabilidade de cada arquivo;
      - services existentes;
      - schemas existentes;
      - repositories existentes, caso existam;
      - rotas/endpoints existentes;
      - dependências utilizadas;
      - fluxo de login;
      - fluxo de validação de usuário;
      - fluxo de geração de token;
      - fluxo de autenticação por request;
      - como o usuário autenticado é recuperado;
      - como erros de autenticação são tratados;
      - quais exceções são utilizadas;
      - quais status HTTP são retornados;
      - como senhas são verificadas ou criptografadas;
      - como JWT ou outro mecanismo de token é usado;
      - quais configurações de segurança são necessárias.    
3. Implementação de Trainer
    Analise profundamente o domínio `mach-api/app/domain/trainer`:
    Explique:
      - objetivo do domínio de trainer;
      - arquivos existentes;
      - responsabilidade de cada arquivo;
      - services existentes;
      - schemas existentes;
      - repositories existentes, caso existam;
      - rotas/endpoints existentes;
      - dependências utilizadas;
      - relação entre Trainer e User;
      - relação entre Trainer, pokedex e associações;
      - fluxo de criação ou inicialização de trainer;
      - fluxo da rota /trainers/initialize, caso exista;
      - regras de negócio aplicadas;
      - tratamento de erros;
      - status possíveis do trainer ou pokedex;
      - pontos de integração com autenticação. 

4. Models analisados
    Analise os arquivos `mach-api/app/models/user.py`, `mach-api/app/models/trainer.py` e `mach-api/app/models/associations.py`:
    Para cada model, documente:

     - nome da classe;
     - nome da tabela;
     - campos;
     - tipos;
     - constraints;
     - valores padrão;
     - índices;
     - enums utilizados;
     - relacionamentos;
     - foreign keys;
     - tabelas associativas;
     - cardinalidade das relações;
     - comportamento esperado;
     - impacto desses models na autenticação e no trainer. 
5. Pasta core `mach-api/app/core`
    Analise a pasta core e explique:
    
      - configurações da aplicação;
      - variáveis de ambiente esperadas;
      - configuração de banco;
      - configuração de segurança;
      - configuração de autenticação;
      - configuração de JWT/token;
      - configuração de CORS, se existir;
      - dependências globais;
      - inicialização da aplicação;
      - utilitários centrais;
      - padrões que devem ser respeitados em novas features.
6. Pasta shared `mach-api/app/shared`
    Analise a pasta shared e explique:
    
      - responsabilidades da pasta;
      - helpers;
      - exceptions;
      - base repositories;
      - schemas compartilhados;
      - paginação, se existir;
      - responses compartilhadas;
      - dependências compartilhadas;
      - padrões reutilizáveis;
      - o que deve ou não ser colocado em shared.
   
7. Fluxos completos
     Documente em detalhes os fluxos abaixo, do início ao fim:
     
    7.1. Explique passo a passo:

     - request recebida;
     - rota acionada;
     - schema utilizado;
     - service chamado;
     - repository/model acessado;
     - validações realizadas;
     - token gerado;
     - response retornada;
     - erros possíveis.
   
    7.2. Fluxo de identificação do usuário autenticado
    Explique passo a passo:

     - token recebido;
     - dependência acionada;
     - token validado;
     - usuário buscado;
     - usuário retornado para a rota;
     - erros possíveis.
    
    7.3. Fluxo de trainer
    Explique passo a passo:

     - usuário autenticado;
     - trainer criado ou recuperado;
     - relações carregadas;
     - pokedex inicializada, se aplicável;
     - response retornada;
     - erros possíveis.
---

### Padrões que devem ser extraídos

  Extraia e documente os padrões do projeto para:

  - nomes de arquivos;
  - nomes de classes;
  - nomes de schemas;
  - nomes de services;
  - nomes de repositories;
  - nomes de rotas;
  - nomes de métodos;
  - organização por domínio;
  - separação de responsabilidade;
  - tratamento de exceções;
  - responses;
  - tipagem;
  - async/await;
  - uso de SQLAlchemy async;
  - uso de Pydantic;
  - injeção de dependências;
  - autenticação e autorização.
---

### Regras importantes
  - Não altere nenhum arquivo.
  - Não sugira refatorações neste momento.
  - Não implemente código.
  - Não faça críticas genéricas.
  - Não invente padrões que não existam.
  - Documente apenas o que for encontrado no projeto.
  - Quando houver incerteza, marque como `Ponto de atenção`.
  - Quando houver inconsistência, marque como `Inconsistência encontrada`.
  - Quando algo for inferido por leitura do código, marque como `Inferência`.
  - Quando algo for confirmado diretamente pelo código, marque como `Confirmado no código`.

### Formato obrigatório da resposta

```markdown
# OpenSpec Prompt - {title}
## READ:
        [AGENTS SPEC](../AGENTS.md) 
        [README SPEC](../README.md) 
        [AGENTS API](../mach-api/AGENTS.md) 
        [README API](../mach-api/README.md)
        [AGENTS WEB](../mach-web/AGENTS.md)
        [README WEB](../mach-web/README.md)
---

## Context
This project uses a monorepo with two submodules:

- **mach-api** — Python 3.13, FastAPI, SQLAlchemy async, domain-driven architecture
- **mach-web** — Next.js 16, React 19, TypeScript, Tailwind CSS v4
    
  {context}
---
    
## Objective 
    {objective}
---
    
## Details 
   {details}
---
    
## Projects
    {details}

### SPEC: mach-api
    {details}
#### 1. Database Models
##### 1.1 {model}    
File Location: {file_location}
Create a SQLAlchemy model for this table.
| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4`, not nullable |
    {details}

#### 2. Domain Models
##### 2.1 {model} Schema    
File Location: {file_location}
Create a Pydantic model for this schema.

##### 2.2 {model} Repository  
File Location: {file_location}
Create a repository class `{model}Repository` that implements BaseRepository[{model}] from `mach-api/app/core/repository/base.py`.
This repository should populate the model with {model}, the relations with the relationships using selectInload, and default_order_by with the value 'order'.
##### 2.3 {model} Service    
File Location: {file_location}
Create a service class `{model}Service` that implements BaseService[{repository},{model}] from `mach-api/app/core/service/base.py`.
This service should use the repository to perform CRUD operations on {model}.
##### 2.4 {model} Businesses  
File Location: {file_location}
##### 2.5 {model} Route    
File Location: {file_location}
Create a FastApi router `{model}Route` that implements the following endpoints
Register the `{model}Route` in `mach-api/app/main.py` under the `/{model}` path.

`app.include_router({model}_router, prefix="/{model}", tags=["{model}"])`

#### 3. Shared
 {details}

### SPEC: mach-web
    {details}
#### 1. Page
File Location: {file_location}
Create a Next.js page for this route.
{details}
#### 2. api
File Location: {file_location} 
if necessary, Create a Next.js API route for this route.
{details}
#### 3. ui/features
File Location: {file_location}
if necessary, Create a React component for this UI element.
{details}
#### 4. ds
File Location: {fle_location}
If necessary, create a system design component so that it can be reused.
{details}
#### 5. Requirements
    * The page must be:
    * Fully responsive
    * Visually rich
    * Reuse BaseService pattern
    * Follow same structure (state, loading, error)
    * Do NOT invent a new pattern
    * Keep consistency with existing codebase
    * Inspired by:       
        * card-based layout
        * clean modern dashboards
#### 6. UI/UX Guidelines

* Use TailwindCSS
* Use Design System components (`app/ds/`)
* Prefer composition
* Rounded cards
* Shadows
* Good spacing

#### 7. Notes
* Follow existing patterns strictly
* Avoid creating new abstractions unnecessarily
* Keep UI clean and modern
* Prefer reuse to reinvention

---

### GENERAL RULES
— Do not modify files unrelated to this spec.
— All async functions must use `async/await`.
— All database queries must use SQLAlchemy `AsyncSession`.
— No raw SQL — use SQLAlchemy ORM only.
— The Redis client must be injected as a FastAPI dependency.
— Error handling must return meaningful HTTP status codes and messages.
— Use Tailwind and custom components from `/ds`.
— Inject dependencies (Redis)
— Handle errors properly
— Do not touch auth modules
— Do not modify the `/ds` components
— Toda alteração de base de dados deve atualizar o redis
#### Coverage Rules
— Minimum:100%
— Focus on real behavior
— Avoid trivial tests
— Mock: API, REDIS
#### Testing Rules

— Use pytest for API testing
— Use Jest and React Testing Library for web testing
— Cover all critical cases in both API and web testing
— Mock external dependencies (API, REDIS) in tests
— Ensure coverage meets minimum requirement
— Focus on real behavior and avoid trivial tests
```
---

### Informações necessárias para o contexto
    1. Visão geral da arquitetura
    2. Estrutura analisada
    3. Arquitetura da aplicação
    4. Core
    5. Shared
    6. Domínio Auth
    6.1 Objetivo
    6.2 Arquivos
    6.3 Schemas
    6.4 Services
    6.5 Repositories
    6.6 Rotas
    6.7 Dependências
    6.8 Fluxo de autenticação
    6.9 Fluxo de usuário autenticado
    6.10 Tratamento de erros
    7. Domínio Trainer
    7.1 Objetivo
    7.2 Arquivos
    7.3 Schemas
    7.4 Services
    7.5 Repositories
    7.6 Rotas
    7.7 Relação com User
    7.8 Relação com Pokedex
    7.9 Fluxo de inicialização
    7.10 Tratamento de erros
    8. Models
    8.1 User
    8.2 Trainer
    8.3 Associations
    9. Fluxos completos
    9.1 Fluxo completo de login
    9.2 Fluxo completo de autenticação por token
    9.3 Fluxo completo de trainer
    9.4 Fluxo completo de inicialização de pokedex
    10. Padrões arquiteturais encontrados
    11. Padrões de nomenclatura
    12. Padrões para criação de novas features
    13. Como futuras propostas OpenSpec devem usar esse contexto
    14. Inconsistências encontradas
    15. Pontos de atenção
    16. Resumo final para reutilização em prompts