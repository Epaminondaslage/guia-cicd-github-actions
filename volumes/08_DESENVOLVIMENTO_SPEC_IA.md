# Volume 08 — Desenvolvimento Orientado por Especificação e IA

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 08_DESENVOLVIMENTO_SPEC_IA.md  
**Versão:** 0.1.0  
**Pré-requisitos:** Volumes 01 a 07

---

## 1. Objetivo

Este volume descreve um processo de desenvolvimento assistido por IA no qual a implementação começa por uma especificação clara e rastreável.

Fluxo principal:

```text
Necessidade
   |
   v
Entrevista de requisitos
   |
   v
SPEC
   |
   v
Revisão humana
   |
   v
Plano de implementação
   |
   v
Branch
   |
   v
Implementação assistida
   |
   v
Testes locais
   |
   v
PR
   |
   v
CI + E2E
   |
   v
Merge
   |
   v
DEV
   |
   v
Aprovação
   |
   v
PROD
```

---

# 2. Por que especificar antes de codificar

Sem especificação:

```text
"melhore o dashboard"
```

pode resultar em interpretações muito diferentes.

Com SPEC:

```text
objetivo
escopo
comportamento
restrições
critérios de aceitação
fora de escopo
```

a implementação fica verificável.

---

# 3. IA como entrevistadora

Antes de escrever código, o agente pode fazer perguntas como:

- qual problema será resolvido?
- quem usa?
- qual fluxo atual?
- qual fluxo desejado?
- existe compatibilidade obrigatória?
- quais telas/endpoints mudam?
- quais comportamentos não podem mudar?
- qual critério define conclusão?

---

# 4. Estrutura de uma SPEC

```markdown
# SPEC-014 — Novo Dashboard

## Contexto

## Problema

## Objetivo

## Escopo

## Requisitos funcionais

## Requisitos não funcionais

## Restrições

## Critérios de aceitação

## Casos de teste

## Fora de escopo

## Riscos

## Referências
```

---

# 5. Contexto

Explique por que a mudança existe.

Exemplo:

```text
A interface atual foi introduzida pela PR #88.
Após uso em DEV, verificou-se que a hierarquia visual não atende
ao fluxo operacional.
```

---

# 6. Problema versus solução

Problema:

```text
usuário demora para localizar chamados críticos
```

Solução proposta:

```text
novo painel com prioridade visual
```

Não confunda os dois.

A SPEC deve preservar o problema mesmo se a solução mudar.

---

# 7. Objetivo

Bom:

```text
reduzir o número de passos necessários para identificar e abrir
um chamado prioritário
```

Ruim:

```text
deixar bonito
```

---

# 8. Escopo

Liste explicitamente o que muda.

Exemplo:

```text
- tabela principal
- filtros
- badge de prioridade
- ordenação inicial
```

---

# 9. Fora de escopo

Muito importante para agentes.

Exemplo:

```text
- não alterar autenticação
- não alterar schema do banco
- não refatorar módulo MQTT
- não modificar API pública
```

Isso reduz expansão indevida.

---

# 10. Requisitos funcionais

Exemplo:

```text
RF01 — usuário pode filtrar por status.
RF02 — chamados críticos aparecem destacados.
RF03 — filtro permanece durante navegação.
```

---

# 11. Requisitos não funcionais

Exemplos:

```text
RNF01 — tela deve responder em até X sob carga definida.
RNF02 — layout deve funcionar em viewport mobile.
RNF03 — não introduzir nova dependência sem justificativa.
```

---

# 12. Critérios de aceitação

Devem ser verificáveis.

Exemplo:

```text
Given existem chamados críticos
When usuário abre dashboard
Then críticos aparecem antes dos demais
```

---

# 13. Casos de teste na SPEC

Não precisam ser implementação técnica completa.

Exemplo:

```text
CA01 — filtro por aberto
CA02 — filtro sem resultados
CA03 — prioridade crítica
CA04 — mobile
```

Depois o plano decide camada:

```text
unit / API / E2E / visual
```

---

# 14. Revisão humana da SPEC

Antes do código:

```text
SPEC gerada
   |
   v
humano revisa
   |
   +-- corrigir
   |
   v
aprovada
```

A aprovação da SPEC reduz retrabalho de implementação.

---

# 15. Plano de implementação

Depois da SPEC, gere plano técnico.

Exemplo:

```text
1. alterar componente DashboardTable
2. criar PriorityBadge
3. atualizar endpoint de filtro
4. adicionar unit tests
5. adicionar E2E smoke
6. atualizar documentação
```

---

# 16. Plano não é SPEC

SPEC responde:

```text
o que e por quê
```

Plano responde:

```text
como
```

Se a solução técnica mudar, a SPEC pode continuar válida.

---

# 17. Plano por etapas pequenas

Prefira:

```text
Etapa 1
backend

Etapa 2
frontend

Etapa 3
testes

Etapa 4
docs
```

Isso facilita commits e revisão.

---

# 18. Branch dedicada

Exemplo:

```text
feature/014-dashboard-prioridades
```

A branch deve corresponder à unidade de mudança.

---

# 19. Commit checkpoints

Durante implementação assistida:

```text
plano
 |
 +-- etapa 1 -> teste -> commit
 |
 +-- etapa 2 -> teste -> commit
 |
 +-- etapa 3 -> teste -> commit
```

Isso cria pontos de restauração.

---

# 20. Agente com escopo limitado

Instrução recomendada:

```text
Implemente somente as etapas 1 e 2.
Não altere arquivos fora dos diretórios listados
sem justificar antes no plano.
```

---

# 21. Mudanças inesperadas

Após implementação:

```bash
git diff --stat
git diff
```

Revise se arquivos fora do escopo foram alterados.

---

# 22. IA e testes

Agente pode propor testes.

Mas a relação deve ser:

```text
critério de aceitação
      |
      v
teste
```

e não:

```text
código existente
      |
      v
teste que confirma qualquer comportamento atual
```

---

# 23. Teste antes da implementação

Para bugs:

```text
reproduzir
 |
 v
teste FAIL
 |
 v
corrigir
 |
 v
PASS
```

Excelente padrão para agentes.

---

# 24. Revisão do diff por IA

Uma segunda passagem pode analisar:

- mudança fora de escopo;
- regressões;
- secrets;
- código duplicado;
- testes ausentes;
- inconsistência com SPEC.

Idealmente, o reviewer não é o mesmo contexto/agente que escreveu tudo.

---

# 25. PR como unidade de auditoria

Descrição:

```markdown
## SPEC
docs/specs/SPEC-014.md

## Plano
docs/plans/PLAN-014.md

## Implementação
...

## Testes
...

## Referência anterior
PR #88
```

---

# 26. Nova SPEC após implementação insatisfatória

Se frontend não ficou adequado:

```text
PR #88 integrada
      |
      v
problema observado
      |
      v
SPEC-021
      |
      v
nova branch
      |
      v
PR #103
```

Não tente reescrever a história da PR antiga.

---

# 27. Referenciar PR original

Na SPEC:

```text
Contexto:
Esta mudança refina a interface introduzida pela PR #88.
```

Na PR:

```text
Related to #88.
Implements SPEC-021.
```

---

# 28. Reativar projeto

"Reativar" significa recuperar contexto.

Checklist:

```text
ler SPEC original
ler PR original
ver código atual
ver mudanças posteriores
executar testes
documentar problema atual
criar nova SPEC
```

Não presuma que o código ainda é idêntico ao da PR antiga.

---

# 29. Source of truth

Defina fontes:

```text
Código atual = verdade executável
SPEC aprovada = comportamento desejado
PRs = história
Docs = contexto
Tests = contrato automatizado parcial
```

---

# 30. Documentação viva

SPEC não precisa ser alterada eternamente.

Pode haver:

```text
SPEC-014 original
SPEC-021 refinamento
SPEC-034 novo comportamento
```

Isso preserva evolução.

---

# 31. Decision records

Para decisões arquiteturais importantes, use ADR:

```text
docs/adr/ADR-005-use-playwright.md
```

Estrutura:

```text
Contexto
Decisão
Alternativas
Consequências
```

---

# 32. IA e ADR

Agente pode sugerir alternativas, mas decisão deve ser explícita.

Exemplo:

```text
Escolhemos Playwright porque...
Não escolhemos Cypress porque...
```

---

# 33. Prompt operacional

Um bom prompt de implementação contém:

```text
SPEC
plano
branch
restrições
arquivos relevantes
comandos de teste
definition of done
```

---

# 34. Definition of Done

Exemplo:

```text
- critérios atendidos
- lint PASS
- unit PASS
- integration PASS
- smoke E2E PASS
- docs atualizadas
- sem secrets
- PR pronta
```

---

# 35. Superpower-like workflow

Fluxo genérico de skills/agentes:

```text
perguntas
 |
 v
SPEC
 |
 v
review
 |
 v
plan
 |
 v
implementation
 |
 v
E2E
 |
 v
DEV
 |
 v
human gate
 |
 v
PROD
```

A ferramenta específica pode mudar; o processo permanece útil.

---

# 36. Separar geração e validação

Não confie apenas em:

```text
agente que implementou
```

para declarar sucesso.

Use:

```text
pipeline externo
```

como verificação objetiva.

---

# 37. Falha no E2E

Fluxo:

```text
E2E FAIL
 |
 v
classificar
 |
 +-- bug de implementação
 +-- bug de teste
 +-- ambiente
 +-- requisito ambíguo
```

Não altere o teste automaticamente sem diagnóstico.

---

# 38. Requisito ambíguo

Se teste e implementação divergem porque a SPEC é ambígua:

```text
voltar à SPEC
```

e não escolher aleatoriamente um dos comportamentos.

---

# 39. Mudança de requisito durante PR

Se pequena e relacionada:

```text
atualize SPEC
continue mesma PR
```

Se muda substancialmente objetivo:

```text
considere nova SPEC/PR
```

Evite scope creep.

---

# 40. Scope creep

Sinal:

```text
"já que estamos aqui..."
```

seguido de várias refatorações não relacionadas.

Crie Issues/PRs separadas.

---

# 41. Refatoração oportunista

Pequenas melhorias diretamente necessárias podem ser aceitáveis.

Mas devem estar justificadas na PR.

---

# 42. IA e arquitetura existente

Antes de alterar:

```text
ler padrões atuais
```

Agente não deve introduzir nova arquitetura apenas por preferência.

---

# 43. Regra de consistência

Se projeto usa:

```text
service/repository/controller
```

uma feature pequena não deveria inventar arquitetura completamente diferente.

---

# 44. Mudança arquitetural

Se necessária:

```text
ADR
SPEC
plano
PR dedicada ou claramente delimitada
```

---

# 45. Código completo versus patch

Agentes podem produzir grandes alterações rapidamente.

Maior velocidade aumenta necessidade de:

- diff review;
- testes;
- limites;
- commits.

---

# 46. Context window

Projetos grandes excedem contexto de agentes.

Documentação estruturada reduz dependência de memória.

Arquivos úteis:

```text
README
ARCHITECTURE.md
CONTRIBUTING.md
docs/specs/
docs/adr/
```

---

# 47. AGENTS.md / instruções do projeto

Um arquivo de instruções pode registrar:

- stack;
- comandos;
- arquitetura;
- regras;
- testes;
- proibições.

O nome depende da ferramenta.

---

# 48. Comandos canônicos

Documente:

```text
npm ci
npm run lint
npm run test:unit
npm run test:integration
npm run test:e2e:smoke
```

Agentes devem usar os mesmos comandos do CI.

---

# 49. Ambientes

Agente deve saber:

```text
local
CI
DEV
PROD
```

e nunca confundi-los.

---

# 50. Produção requer autorização

A capacidade técnica de fazer deploy não significa permissão automática.

Fluxo:

```text
agente prepara
 |
 v
DEV validado
 |
 v
pergunta explícita
 |
 +-- não
 |
 +-- sim -> PROD
```

---

# 51. Secret safety

Nunca forneça secret em prompt se existe mecanismo de secret management.

Agente não precisa conhecer valor bruto para escrever integração.

---

# 52. Logs e secrets

Automação deve evitar:

```bash
echo "$SECRET"
```

e comandos que imprimam ambiente completo.

---

# 53. PR automation

Agente pode:

- criar branch;
- commit;
- abrir PR;
- acompanhar checks;
- corrigir falhas.

Mas merge/deploy devem seguir política.

---

# 54. Autonomia graduada

Níveis:

```text
1. sugerir
2. modificar branch
3. abrir PR
4. corrigir CI
5. merge autorizado
6. deploy DEV
7. PROD com gate
```

Defina explicitamente.

---

# 55. Human-in-the-loop

Humano deve permanecer principalmente em:

- requisitos;
- decisões arquiteturais;
- aprovação visual;
- risco;
- produção.

Automação assume tarefas repetíveis.

---

# 56. Revisão visual

Frontend merece etapa:

```text
DEV
 |
 v
humano abre interface
 |
 v
aprova ou cria refinamento
```

E2E funcional não substitui gosto/requisito visual.

---

# 57. Screenshots na PR

Agente pode anexar evidências de UI quando ferramenta permitir.

Ajuda revisão assíncrona.

---

# 58. Visual regression

Baseline aprovado:

```text
PR anterior
```

Nova alteração gera diff.

Mudança intencional requer atualização consciente do baseline.

---

# 59. Plano de rollback

Antes de deploy, agente pode verificar:

```text
qual versão anterior?
como voltar?
migration é compatível?
```

---

# 60. IA durante incidente

Durante incidente, limite autonomia.

Prioridade:

```text
restaurar serviço
```

Não iniciar refatoração ampla.

---

# 61. Post-mortem assistido

IA pode ajudar a organizar:

- timeline;
- logs;
- commits;
- PR;
- causa;
- ações.

A conclusão deve ser revisada por humano.

---

# 62. Audit trail

Rastreabilidade ideal:

```text
Issue
 |
SPEC
 |
PLAN
 |
Branch
 |
Commits
 |
PR
 |
Checks
 |
Deploy
 |
Version
```

---

# 63. IDs consistentes

Exemplo:

```text
Issue #214
SPEC-214
feature/214-dashboard
PR #231
```

Não é obrigatório, mas simplifica navegação.

---

# 64. Plano versionado

Guardar plano em:

```text
docs/plans/
```

é útil para features complexas.

Para tarefas pequenas, corpo da PR pode ser suficiente.

---

# 65. Estado da implementação

Checklist no plano:

```text
- [x] backend
- [x] unit
- [ ] frontend
- [ ] E2E
```

Ajuda continuidade entre sessões/agentes.

---

# 66. Handoff entre agentes

O handoff deve conter:

```text
o que foi feito
o que falta
branch
último commit
testes
falhas conhecidas
decisões
```

Não depender de conversa anterior.

---

# 67. Continuidade entre dias

Antes de retomar:

```bash
git status
git log --oneline -10
git diff
```

Leia:

```text
SPEC
PLAN
PR
```

---

# 68. Evitar código não commitado por longos períodos

Commits pequenos são checkpoints.

Especialmente importante quando diferentes agentes trabalham em sequência.

---

# 69. Worktrees

Git worktree pode permitir múltiplas branches em diretórios separados.

Útil para agentes paralelos.

Exemplo:

```text
worktree feature-A
worktree feature-B
```

Requer disciplina para não editar o mesmo domínio simultaneamente.

---

# 70. Paralelismo de agentes

Seguro:

```text
Agente A -> backend independente
Agente B -> documentação
```

Arriscado:

```text
A e B alteram mesmo arquivo central
```

---

# 71. Ownership temporário

Plano pode indicar:

```text
Agente A: src/auth/**
Agente B: frontend/dashboard/**
```

Reduz conflitos.

---

# 72. Integração de trabalhos paralelos

Cada agente:

```text
branch
 |
 v
PR
```

Depois integrar em ordem controlada.

Não juntar resultados manualmente sem histórico.

---

# 73. Stacked PRs com IA

Uma feature grande pode gerar:

```text
PR1 schema/API
PR2 service
PR3 frontend
PR4 E2E
```

Só use stack se dependências forem claras.

---

# 74. PR independente é preferível

Quando possível:

```text
cada PR parte de main
```

Isso reduz complexidade.

---

# 75. Revisão automática de segurança

Antes da PR pronta:

```text
search secrets
check dependency changes
check permissions
check Dockerfile
```

---

# 76. Dependency additions

Agente deve justificar nova dependência:

```text
por que?
alternativas?
manutenção?
licença?
tamanho?
segurança?
```

---

# 77. Evitar package sprawl

IA pode adicionar biblioteca para tarefas triviais.

Prefira stack existente quando suficiente.

---

# 78. Database changes

SPEC deve indicar:

```text
migration necessária?
backward compatible?
dados existentes?
rollback?
```

---

# 79. API changes

Se contrato muda:

```text
versionamento?
frontend compatível?
clientes externos?
```

Contract tests ajudam.

---

# 80. Feature flags

Mudanças grandes podem ser integradas desativadas.

Fluxo:

```text
merge
 |
 v
feature flag OFF
 |
 v
DEV
 |
 v
ativar
```

Requer gestão de flags.

---

# 81. Feature flag não é lixo permanente

Flags temporárias devem ter plano de remoção.

---

# 82. Definition of Ready

Antes de implementação:

```text
SPEC clara
critérios claros
dependências conhecidas
risco avaliado
```

---

# 83. Definition of Done

Depois:

```text
código
testes
docs
CI
E2E
review
DEV
```

e, se objetivo incluir release:

```text
PROD validado
```

---

# 84. Checklist SPEC

- [ ] Problema claro.
- [ ] Objetivo claro.
- [ ] Escopo.
- [ ] Fora de escopo.
- [ ] Critérios verificáveis.
- [ ] Riscos.
- [ ] Referências.
- [ ] Dependências.
- [ ] Impacto em dados/API.
- [ ] Necessidade visual.

---

# 85. Checklist plano

- [ ] Arquivos/módulos.
- [ ] Ordem.
- [ ] Testes.
- [ ] Migrations.
- [ ] Deploy.
- [ ] Rollback.
- [ ] Documentação.
- [ ] Mudanças arquiteturais justificadas.

---

# 86. Checklist implementação IA

- [ ] Branch correta.
- [ ] Sem mudanças fora de escopo.
- [ ] Diff revisado.
- [ ] Testes locais.
- [ ] Nenhum secret.
- [ ] Commits coerentes.
- [ ] SPEC atual continua atendida.

---

# 87. Checklist PR

- [ ] SPEC citada.
- [ ] Plano resumido.
- [ ] Critérios atendidos.
- [ ] Checks PASS.
- [ ] E2E adequado.
- [ ] Evidências visuais se necessário.
- [ ] Referência à PR anterior se refinamento.
- [ ] Riscos e rollout documentados.

---

# 88. Arquitetura documental sugerida

```text
docs/
├── specs/
│   ├── SPEC-001.md
│   └── SPEC-002.md
├── plans/
│   ├── PLAN-001.md
│   └── PLAN-002.md
├── adr/
└── architecture/
```

---

# 89. Template de SPEC

```markdown
# SPEC-NNN — Título

## Contexto

## Problema

## Objetivo

## Escopo

## Fora de escopo

## Requisitos

## Critérios de aceitação

## Casos de teste

## Riscos

## Referências
```

---

# 90. Template de plano

```markdown
# PLAN-NNN — Título

## Resumo técnico

## Etapas

## Arquivos afetados

## Banco

## API

## Frontend

## Testes

## Deploy

## Rollback

## Checklist
```

---

# 91. Fluxo final

```text
IDEIA
 |
 v
INTERVIEW
 |
 v
SPEC
 |
 v
HUMAN REVIEW
 |
 v
PLAN
 |
 v
BRANCH
 |
 v
AI IMPLEMENTATION
 |
 v
LOCAL TESTS
 |
 v
PR
 |
 v
CI / E2E
 |
 v
CODE REVIEW
 |
 v
MERGE
 |
 v
DEV
 |
 v
VISUAL/FUNCTIONAL VALIDATION
 |
 v
HUMAN GATE
 |
 v
PROD
```

---

# 92. Próximo volume

**Volume 09 — Continuous Deployment e Deploy DEV → PROD**

Abordará:

- artifacts;
- environments;
- DEV;
- gate humano;
- produção;
- SSH;
- Docker;
- versionamento;
- migrations;
- health checks;
- rollback;
- blue/green.

---

**Fim do Volume 08 — Desenvolvimento Orientado por Especificação e IA**
