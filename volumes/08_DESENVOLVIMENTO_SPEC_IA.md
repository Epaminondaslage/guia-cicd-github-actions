# Volume 08 — Desenvolvimento Orientado por Especificação e IA

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 08_DESENVOLVIMENTO_SPEC_IA.md  
**Versão:** 0.2.0  
**Pré-requisitos:** Volumes 01 a 07

---

## 1. Objetivo

Este volume descreve um processo de desenvolvimento assistido por IA no qual a implementação começa por uma especificação clara e rastreável.

Fluxo principal:

```text
Necessidade
   |
   v
Brainstorm (explorar intenção, alternativas, restrições)
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
Revisão humana do plano
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
Revisão humana do código gerado (obrigatória)
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

Duas revisões humanas nunca devem ser puladas: a da SPEC/plano (antes de codar) e a do código gerado (antes do merge). São coisas diferentes: a primeira valida o "o quê" e o "por quê"; a segunda valida o "como" foi implementado.

---

## 2. Por que especificar antes de codificar

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

## 3. Brainstorm antes da entrevista formal

Antes de qualquer SPEC, vale uma etapa curta e deliberadamente aberta de brainstorm, que não se confunde com a entrevista de requisitos (seção seguinte).

Objetivo do brainstorm:

```text
explorar intenção real do pedido
levantar alternativas de solução
identificar restrições ainda não ditas
descartar abordagens obviamente ruins
```

Diferença prática:

```text
Brainstorm  -> "que problema é esse, de fato? quais caminhos existem?"
Entrevista  -> "dado o caminho escolhido, quais são os detalhes concretos?"
```

Pular o brainstorm e ir direto para a SPEC costuma produzir specs tecnicamente corretas, mas que resolvem o problema errado, porque ninguém questionou a premissa inicial.

Sinal de que vale a pena brainstormar: o pedido veio como solução pronta ("adicionar um botão X") em vez de problema ("usuário não consegue fazer Y"). Nesse caso, volte um passo e pergunte pelo problema antes de aceitar a solução.

---

## 4. IA como entrevistadora

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

## 5. Estrutura de uma SPEC

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

## 6. Contexto

Explique por que a mudança existe.

Exemplo:

```text
A interface atual foi introduzida pela PR #88.
Após uso em DEV, verificou-se que a hierarquia visual não atende
ao fluxo operacional.
```

---

## 7. Problema versus solução

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

## 8. Objetivo

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

## 9. Escopo

Liste explicitamente o que muda.

Exemplo:

```text
- tabela principal
- filtros
- badge de prioridade
- ordenação inicial
```

---

## 10. Fora de escopo

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

## 11. Requisitos funcionais

Exemplo:

```text
RF01 — usuário pode filtrar por status.
RF02 — chamados críticos aparecem destacados.
RF03 — filtro permanece durante navegação.
```

---

## 12. Requisitos não funcionais

Exemplos:

```text
RNF01 — tela deve responder em até X sob carga definida.
RNF02 — layout deve funcionar em viewport mobile.
RNF03 — não introduzir nova dependência sem justificativa.
```

---

## 13. Critérios de aceitação

Devem ser verificáveis.

Exemplo:

```text
Given existem chamados críticos
When usuário abre dashboard
Then críticos aparecem antes dos demais
```

---

## 14. Casos de teste na SPEC

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

## 15. Revisão humana da SPEC

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

## 16. Plano de implementação

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

## 17. Plano não é SPEC

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

## 18. Plano por etapas pequenas

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

## 19. Branch dedicada

Exemplo:

```text
feature/014-dashboard-prioridades
```

A branch deve corresponder à unidade de mudança.

---

## 20. Commit checkpoints

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

## 21. Agente com escopo limitado

Instrução recomendada:

```text
Implemente somente as etapas 1 e 2.
Não altere arquivos fora dos diretórios listados
sem justificar antes no plano.
```

---

## 22. Mudanças inesperadas

Após implementação:

```bash
git diff --stat
git diff
```

Revise se arquivos fora do escopo foram alterados.

---

## 23. IA e testes

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

## 24. Teste antes da implementação

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

## 25. Revisão do diff por IA

Uma segunda passagem pode analisar:

- mudança fora de escopo;
- regressões;
- secrets;
- código duplicado;
- testes ausentes;
- inconsistência com SPEC.

Idealmente, o reviewer não é o mesmo contexto/agente que escreveu tudo.

---

## 26. Revisão humana obrigatória antes do merge

Código gerado por IA nunca deve ser mesclado sem revisão humana linha a linha. Isso vale mesmo quando:

```text
CI verde
testes passando
E2E passando
o agente afirma que está tudo certo
```

Nenhum desses sinais substitui a leitura do diff por uma pessoa. Um agente pode gerar código que passa nos testes existentes e ainda assim:

- introduzir uma vulnerabilidade que nenhum teste cobre;
- usar uma API/biblioteca que não existe, ou que existe mas com outra assinatura (alucinação plausível);
- resolver o sintoma testado sem resolver o requisito real;
- remover ou enfraquecer uma asserção para fazer o teste passar;
- copiar um padrão de outro trecho do projeto que não se aplica ao contexto atual;
- introduzir efeito colateral fora do escopo declarado.

### Checklist mínimo de revisão humana

- [ ] Todo o diff foi lido, não só o resumo gerado pelo agente.
- [ ] Cada trecho alterado corresponde a um item do plano/SPEC, nada "a mais".
- [ ] Nenhuma dependência, API ou função usada foi inventada; existe e faz o que o código presume.
- [ ] Testes novos realmente testam o comportamento do critério de aceitação, não apenas "não quebra nada".
- [ ] Nenhum teste existente foi enfraquecido, comentado ou apagado para fazer a suíte passar.
- [ ] Nenhum segredo, token, IP interno ou dado sensível foi incluído no código, teste ou log.
- [ ] Tratamento de erro e casos de borda fazem sentido, não só o caminho feliz.
- [ ] Mudanças em banco de dados são compatíveis e reversíveis.
- [ ] Estilo e padrões seguem o restante do projeto (revisão automática de lint não garante isso sozinha).
- [ ] O revisor entende o código a ponto de conseguir explicá-lo; se não entende, não aprova.

Regra prática: se a pessoa que aprova não consegue explicar o que o código faz e por quê, a revisão não está completa, independentemente de quem (humano ou IA) escreveu o código.

---

## 27. PR como unidade de auditoria

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

## 28. Nova SPEC após implementação insatisfatória

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

## 29. Referenciar PR original

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

## 30. Reativar projeto

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

## 31. Source of truth

Defina fontes:

```text
Código atual = verdade executável
SPEC aprovada = comportamento desejado
PRs = história
Docs = contexto
Tests = contrato automatizado parcial
```

---

## 32. Documentação viva

SPEC não precisa ser alterada eternamente.

Pode haver:

```text
SPEC-014 original
SPEC-021 refinamento
SPEC-034 novo comportamento
```

Isso preserva evolução.

---

## 33. Decision records

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

## 34. IA e ADR

Agente pode sugerir alternativas, mas decisão deve ser explícita.

Exemplo:

```text
Escolhemos Playwright porque...
Não escolhemos Cypress porque...
```

---

## 35. Prompt operacional

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

## 36. Contexto mínimo necessário

Mais contexto não é sempre melhor. Um prompt inchado com arquivos irrelevantes, histórico antigo ou documentação genérica dilui a atenção do agente e aumenta a chance de ele "inventar" conexões que não existem.

Prefira:

```text
o mínimo de contexto que torna a tarefa não-ambígua
```

em vez de:

```text
tudo que existe sobre o projeto
```

Sinais de contexto excessivo:

- o agente cita arquivos que não têm relação com a tarefa;
- a resposta menciona padrões de um módulo diferente do afetado;
- o prompt inclui documentação inteira quando só uma seção importa.

Sinais de contexto insuficiente:

- o agente pergunta algo que já estava disponível no repositório;
- a implementação ignora um padrão existente porque não foi mostrado;
- o agente reinventa uma função que já existe em outro arquivo.

### O que incluir

```text
SPEC e/ou plano da tarefa
arquivos que serão efetivamente alterados
arquivos que definem o padrão a seguir (exemplo de referência)
contrato/assinatura de funções ou APIs que serão chamadas
critérios de aceitação explícitos
comandos para rodar lint/testes localmente
o que está fora de escopo
```

### O que evitar incluir

```text
o repositório inteiro "por garantia"
documentação desatualizada sem aviso de que está desatualizada
histórico de conversas anteriores sem relação com a tarefa atual
arquivos de configuração irrelevantes ao módulo afetado
```

---

## 37. Exemplos e critérios de aceite no prompt

Um exemplo concreto vale mais que uma descrição abstrata. Sempre que possível, o prompt de implementação deve conter:

- um exemplo de entrada e saída esperada (ou de tela antes/depois);
- um trecho de código já existente no projeto que segue o padrão desejado;
- os critérios de aceitação da SPEC, copiados literalmente, não parafraseados;
- a definição explícita de "pronto" (ver Definition of Done, seção seguinte).

Exemplo de prompt bem formado:

```text
Contexto: SPEC-014, seção "Critérios de aceitação".

Arquivos relevantes:
- src/components/DashboardTable.tsx (será alterado)
- src/components/StatusBadge.tsx (padrão de referência para o novo PriorityBadge)

Não alterar:
- src/api/auth/** (fora de escopo)

Critério de aceite (copiado da SPEC):
Given existem chamados críticos
When usuário abre dashboard
Then críticos aparecem antes dos demais

Comandos de verificação:
npm run test:unit -- DashboardTable
npm run lint
```

Prompt vago, a evitar:

```text
melhora a tabela do dashboard pra mostrar os chamados urgentes primeiro,
segue o padrão que já tem no projeto
```

O segundo exemplo obriga o agente a adivinhar "qual padrão" e "o que é urgente", exatamente o tipo de ambiguidade que a SPEC deveria ter eliminado antes de chegar ao prompt.

---

## 38. Definition of Done

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

## 39. Superpower-like workflow

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

## 40. Separar geração e validação

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

## 41. Falha no E2E

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

## 42. Requisito ambíguo

Se teste e implementação divergem porque a SPEC é ambígua:

```text
voltar à SPEC
```

e não escolher aleatoriamente um dos comportamentos.

---

## 43. Mudança de requisito durante PR

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

## 44. Scope creep

Sinal:

```text
"já que estamos aqui..."
```

seguido de várias refatorações não relacionadas.

Crie Issues/PRs separadas.

---

## 45. Refatoração oportunista

Pequenas melhorias diretamente necessárias podem ser aceitáveis.

Mas devem estar justificadas na PR.

---

## 46. IA e arquitetura existente

Antes de alterar:

```text
ler padrões atuais
```

Agente não deve introduzir nova arquitetura apenas por preferência.

---

## 47. Regra de consistência

Se projeto usa:

```text
service/repository/controller
```

uma feature pequena não deveria inventar arquitetura completamente diferente.

---

## 48. Mudança arquitetural

Se necessária:

```text
ADR
SPEC
plano
PR dedicada ou claramente delimitada
```

---

## 49. Código completo versus patch

Agentes podem produzir grandes alterações rapidamente.

Maior velocidade aumenta necessidade de:

- diff review;
- testes;
- limites;
- commits.

---

## 50. Context window

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

## 51. AGENTS.md / instruções do projeto

Um arquivo de instruções pode registrar:

- stack;
- comandos;
- arquitetura;
- regras;
- testes;
- proibições.

O nome depende da ferramenta.

---

## 52. Comandos canônicos

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

## 53. Ambientes

Agente deve saber:

```text
local
CI
DEV
PROD
```

e nunca confundi-los.

---

## 54. Produção requer autorização

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

## 55. Secret safety

Nunca forneça secret em prompt se existe mecanismo de secret management.

Agente não precisa conhecer valor bruto para escrever integração.

---

## 56. Logs e secrets

Automação deve evitar:

```bash
echo "$SECRET"
```

e comandos que imprimam ambiente completo.

---

## 57. PR automation

Agente pode:

- criar branch;
- commit;
- abrir PR;
- acompanhar checks;
- corrigir falhas.

Mas merge/deploy devem seguir política.

---

## 58. Autonomia graduada

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

## 59. Human-in-the-loop

Humano deve permanecer principalmente em:

- requisitos;
- decisões arquiteturais;
- aprovação visual;
- risco;
- produção.

Automação assume tarefas repetíveis.

---

## 60. Revisão visual

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

## 61. Screenshots na PR

Agente pode anexar evidências de UI quando ferramenta permitir.

Ajuda revisão assíncrona.

---

## 62. Visual regression

Baseline aprovado:

```text
PR anterior
```

Nova alteração gera diff.

Mudança intencional requer atualização consciente do baseline.

---

## 63. Plano de rollback

Antes de deploy, agente pode verificar:

```text
qual versão anterior?
como voltar?
migration é compatível?
```

---

## 64. IA durante incidente

Durante incidente, limite autonomia.

Prioridade:

```text
restaurar serviço
```

Não iniciar refatoração ampla.

---

## 65. Post-mortem assistido

IA pode ajudar a organizar:

- timeline;
- logs;
- commits;
- PR;
- causa;
- ações.

A conclusão deve ser revisada por humano.

---

## 66. Audit trail

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

## 67. IDs consistentes

Exemplo:

```text
Issue #214
SPEC-214
feature/214-dashboard
PR #231
```

Não é obrigatório, mas simplifica navegação.

---

## 68. Plano versionado

Guardar plano em:

```text
docs/plans/
```

é útil para features complexas.

Para tarefas pequenas, corpo da PR pode ser suficiente.

---

## 69. Estado da implementação

Checklist no plano:

```text
- [x] backend
- [x] unit
- [ ] frontend
- [ ] E2E
```

Ajuda continuidade entre sessões/agentes.

---

## 70. Handoff entre agentes

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

## 71. Continuidade entre dias

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

## 72. Evitar código não commitado por longos períodos

Commits pequenos são checkpoints.

Especialmente importante quando diferentes agentes trabalham em sequência.

---

## 73. Worktrees

Git worktree pode permitir múltiplas branches em diretórios separados.

Útil para agentes paralelos.

Exemplo:

```text
worktree feature-A
worktree feature-B
```

Requer disciplina para não editar o mesmo domínio simultaneamente.

---

## 74. Paralelismo de agentes

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

## 75. Ownership temporário

Plano pode indicar:

```text
Agente A: src/auth/**
Agente B: frontend/dashboard/**
```

Reduz conflitos.

---

## 76. Integração de trabalhos paralelos

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

## 77. Stacked PRs com IA

Uma feature grande pode gerar:

```text
PR1 schema/API
PR2 service
PR3 frontend
PR4 E2E
```

Só use stack se dependências forem claras.

---

## 78. PR independente é preferível

Quando possível:

```text
cada PR parte de main
```

Isso reduz complexidade.

---

## 79. Revisão automática de segurança

Antes da PR pronta:

```text
search secrets
check dependency changes
check permissions
check Dockerfile
```

---

## 80. Dependency additions

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

## 81. Evitar package sprawl

IA pode adicionar biblioteca para tarefas triviais.

Prefira stack existente quando suficiente.

---

## 82. Database changes

SPEC deve indicar:

```text
migration necessária?
backward compatible?
dados existentes?
rollback?
```

---

## 83. API changes

Se contrato muda:

```text
versionamento?
frontend compatível?
clientes externos?
```

Contract tests ajudam.

---

## 84. Feature flags

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

## 85. Feature flag não é lixo permanente

Flags temporárias devem ter plano de remoção.

---

## 86. Definition of Ready

Antes de implementação:

```text
SPEC clara
critérios claros
dependências conhecidas
risco avaliado
```

---

## 87. Definition of Done

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

## 88. Checklist SPEC

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

## 89. Checklist plano

- [ ] Arquivos/módulos.
- [ ] Ordem.
- [ ] Testes.
- [ ] Migrations.
- [ ] Deploy.
- [ ] Rollback.
- [ ] Documentação.
- [ ] Mudanças arquiteturais justificadas.

---

## 90. Checklist implementação IA

- [ ] Branch correta.
- [ ] Sem mudanças fora de escopo.
- [ ] Diff lido linha a linha por um humano (não só o resumo do agente).
- [ ] Testes locais.
- [ ] Nenhum secret.
- [ ] Commits coerentes.
- [ ] SPEC atual continua atendida.
- [ ] Nenhuma API/dependência usada foi alucinada; todas existem e fazem o que o código presume.
- [ ] Revisor consegue explicar o que o código faz, sem depender da explicação do agente.

---

## 91. Checklist PR

- [ ] SPEC citada.
- [ ] Plano resumido.
- [ ] Critérios atendidos.
- [ ] Checks PASS.
- [ ] E2E adequado.
- [ ] Evidências visuais se necessário.
- [ ] Referência à PR anterior se refinamento.
- [ ] Riscos e rollout documentados.

---

## 92. Arquitetura documental sugerida

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

## 93. Template de SPEC

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

## 94. Template de plano

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

## 95. Fluxo final

```text
IDEIA
 |
 v
BRAINSTORM
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
MANDATORY HUMAN CODE REVIEW (line-by-line diff)
 |
 v
PR
 |
 v
CI / E2E
 |
 v
CODE REVIEW (segunda revisão, pares/CI de segurança)
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

## 96. Vibe coding vs. o fluxo deste volume

"Vibe coding" é o termo popularizado para descrever aceitar o que o agente de IA produz sem revisar linha a linha: pedir, rodar, ver que "funciona" na tela e seguir em frente. Funciona para protótipo descartável. Não deveria ser aceito como prática para código que vai para produção.

O fluxo deste volume (seção 95) existe justamente como antídoto estrutural ao vibe coding, mantendo a velocidade de gerar código com IA sem herdar seu principal risco: aceitar código que ninguém entende de fato.

Diferenças concretas:

| Vibe coding | Fluxo deste volume |
|---|---|
| Prompt vago, direto para o código | Brainstorm + entrevista antes de especificar (seções 2-4) |
| Sem spec; requisito vive só no prompt | SPEC escrita e revisada por humano (seções 5-15) |
| Implementação sem plano | Plano de implementação revisado antes de codar (seções 16-24) |
| Aceitar o diff porque "rodou" | Revisão humana obrigatória, linha a linha, antes do merge (seção 26); nunca dispensada, nem quando os testes passam |
| Um agente escreve e aprova a si mesmo | Separação escritor/revisor: revisão do diff por um agente diferente do que implementou, ou por subagente adversarial (seção 25) |
| Sem rastro do porquê | PR como unidade de auditoria; SPEC e plano documentam a decisão (seção 27) |
| Erro descoberto em produção | Gate humano antes de produção, validação em DEV, rollback documentado (Volume 09) |

Consequência prática: o tempo "economizado" pulando revisão humana costuma reaparecer depois, como incidente em produção, dependência inventada pelo agente que não existe, ou lógica que passou nos testes mas não faz o que a spec pedia. São os riscos que a checklist da seção 26 lista explicitamente. O fluxo deste guia é mais lento por PR do que vibe coding puro, mas o custo médio por linha de código que chega em produção é menor, porque o erro é pego na revisão humana e não no chamado de suporte.

---

## 97. Próximo volume

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

## Fontes

### Claude Code — fluxo geral

- [Overview](https://code.claude.com/docs/en/overview) — descreve o Claude Code como ferramenta agêntica que lê o código, edita arquivos e roda comandos; base para o fluxo de implementação assistida deste volume (seções 1, 16-22, 39, 95).
- [Common workflows](https://code.claude.com/docs/en/common-workflows) — cobre planejamento antes de editar ("Plan before editing"), execução paralela com worktrees e delegação de pesquisa a subagentes; sustenta as seções 15 (revisão humana da SPEC/plano), 73 (worktrees) e 74-78 (paralelismo de agentes).
- [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) — recomenda "Explore first, then plan, then code", dar a Claude uma forma de verificar o próprio trabalho, revisão adversarial por subagente antes de considerar a tarefa pronta, e padrões Writer/Reviewer; base direta das seções 3-4 (brainstorm/entrevista), 25-26 (revisão do diff e revisão humana obrigatória) e 40 (separar geração e validação).

### Prompt engineering e contexto mínimo

- [Prompt engineering overview](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview) — introduz a prática de engenharia de prompt e remete ao guia de boas práticas de prompting; embasa a seção 35 (prompt operacional).
- [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) — referência viva de técnicas de clareza, exemplos e estruturação de prompts; sustenta a seção 36 (contexto mínimo necessário) e a seção 37 (exemplos e critérios de aceite no prompt), que seguem o princípio de instruções específicas e exemplos concretos descrito nessa página e reforçado em "Best practices for Claude Code" (seção "Provide specific context in your prompts").

### Instruções de projeto (memory)

- [How Claude remembers your project](https://code.claude.com/docs/en/memory) — documenta os arquivos `CLAUDE.md`/`AGENTS.md` como mecanismo de instruções persistentes de projeto (stack, comandos, convenções, proibições); base da seção 51 (AGENTS.md / instruções do projeto) e da seção 50 (context window / documentação estruturada).

### Subagentes e paralelismo

- [Create custom subagents](https://code.claude.com/docs/en/sub-agents) — explica subagentes rodando em contexto isolado, delegação explícita e execução paralela de múltiplos agentes; sustenta as seções 25 (revisão do diff por um agente diferente do que escreveu o código), 73-78 (worktrees, paralelismo, ownership temporário e integração de trabalhos paralelos).

---

**Fim do Volume 08 — Desenvolvimento Orientado por Especificação e IA**
