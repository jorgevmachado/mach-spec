## Standardize domain architecture and fix trainer initialization flow

### Contexto:

O projeto possui uma arquitetura de domínio considerada referência em:

- `mach-api/app/domain/pokemon`

Use esse domínio como base para padronizar organização de arquivos, nomenclatura, responsabilidades, services, repositories, schemas, rotas e demais camadas.

Atualmente, os domínios abaixo não seguem o mesmo padrão arquitetural:

- `mach-api/app/domain/trainer`
- `mach-api/app/domain/auth`

Além disso, a rota abaixo apresenta comportamento incorreto:

```http
POST /trainers/initialize
```

Essa rota deveria inicializar corretamente os dados relacionados ao `trainer` e à `pokedex`, porém atualmente sempre retorna `pokedex_status` como `failed`.

O fluxo atual utiliza `BackgroundTasks`, mas esse mecanismo parece inadequado ou mal implementado para esse caso. A proposta deve investigar a causa do problema e corrigir o fluxo de forma simples, confiável e alinhada à arquitetura existente.

Também deve ser feita uma revisão da nomenclatura dos schemas em todo o projeto para garantir consistência entre os domínios.

---

### Regras:

1. Usar `mach-api/app/domain/pokemon` como referência arquitetural principal.

2. Padronizar os domínios:
   - `mach-api/app/domain/trainer`
   - `mach-api/app/domain/auth`

3. Revisar e ajustar:
   - organização de arquivos;
   - nomes de classes;
   - nomes de schemas;
   - services;
   - repositories;
   - use cases, caso existam;
   - controllers/routes;
   - imports;
   - responsabilidades de cada camada.

4. Revisar a nomenclatura dos schemas em todo o projeto:
   - manter padrão consistente;
   - evitar nomes genéricos ou ambíguos;
   - seguir o padrão usado no domínio `pokemon`, quando aplicável;
   - atualizar todos os pontos de uso caso algum schema seja renomeado.

5. Corrigir o fluxo da rota:

   ```http
   POST /trainers/initialize
   ```
   O fluxo não deve retornar `pokedex_status` como failed indevidamente.

6. Revisar o uso atual de `BackgroundTasks`:
   - identificar por que o status está ficando como `failed`;
   - avaliar se `BackgroundTasks` é adequado para esse fluxo;
   - caso não seja adequado, substituir por uma abordagem mais previsível;
   - evitar soluções complexas desnecessárias.
7. Garantir que a inicialização:
   - execute corretamente;
   - persista os dados esperados;
   - retorne status coerente;
   - trate erros de forma clara;
   - não esconda exceções silenciosamente.
8. Manter compatibilidade com os padrões existentes do projeto:
   - FastAPI;
   - SQLAlchemy async;
   - Pydantic;
   - arquitetura por domínio;
   - tipagem forte;
   - separação de responsabilidades.
9. Não criar uma nova arquitetura paralela.
10. Evitar duplicação de lógica.
11. Caso encontre inconsistências entre os domínios, corrigir seguindo o padrão mais maduro já presente em `pokemon`.   

### Critérios de Aceite:
 - [ ] `mach-api/app/domain/trainer` segue o mesmo padrão arquitetural de `mach-api/app/domain/pokemon`.
 - [ ] `mach-api/app/domain/auth` segue o mesmo padrão arquitetural de `mach-api/app/domain/pokemon`.
 - [ ] Os schemas do projeto possuem nomenclatura consistente.
 - [ ] Todos os imports quebrados após renomeações foram corrigidos.
 - [ ] A rota `POST /trainers/initialize` funciona corretamente.
 - [ ] O campo `pokedex_status` não retorna mais `failed` indevidamente.
 - [ ] O fluxo de inicialização possui tratamento de erro claro.
 - [ ] O uso de `BackgroundTasks` foi corrigido ou substituído por uma solução mais adequada.
 - [ ] O código continua seguindo os padrões de FastAPI async.
 - [ ] A implementação não quebra o domínio `pokemon`.
 - [ ] A solução proposta é simples, sustentável e alinhada com a arquitetura existente.
 - [ ] Foram adicionados ou ajustados testes quando necessário.
 
### Formato de resposta:
Responda no formato de uma proposta OpenSpec.
A resposta deve conter:

1. Resumo da mudança 
    - Explique brevemente o objetivo da proposta.
2. Problemas identificados
   - Liste os problemas arquiteturais encontrados em `trainer`.
   - Liste os problemas arquiteturais encontrados em `auth`.
   - Explique o problema da rota `/trainers/initialize`.
   - Explique o problema relacionado ao uso atual de `BackgroundTasks`.
3. Mudanças propostas
   - Descreva as alterações em `trainer`.
   - Descreva as alterações em `auth`.
   - Descreva as alterações na nomenclatura dos schemas.
   - Descreva a correção da inicialização do trainer/pokedex.
   - Informe se `BackgroundTasks` será mantido, corrigido ou removido.
4. Arquivos impactados
   - Liste os arquivos que devem ser criados
   - Liste os arquivos que devem ser alterados.
   - Liste os arquivos que devem ser removidos, se aplicável.
5. Plano de implementação
   - Organize a implementação em etapas claras e sequenciais.
6. Validações necessárias
   - Informe como validar manualmente.
   - Informe quais testes devem ser criados ou ajustados.
   - Informe cenários de sucesso e erro que precisam ser cobertos.
7. Riscos e cuidados
   - Liste possíveis impactos em rotas existentes.
   - Liste cuidados com renomeação de schemas e imports.
   - Liste riscos relacionados ao fluxo de inicialização.
8. Resultado esperado
   - Descreva o comportamento final esperado após a implementação.