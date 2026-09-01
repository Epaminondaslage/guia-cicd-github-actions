# Volume 02: GitHub e Pull Requests

**Projeto:** Guia Pessoal de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 02_GITHUB_PULL_REQUESTS.md  
**Versão:** 0.2.0  
**Status:** Revisado. Pull Requests, branch protection, CODEOWNERS e merge atualizados (2026)

---

## 1. Objetivo

Este volume apresenta o GitHub como plataforma de colaboração sobre Git, com foco especial em **Pull Requests (PRs)**.

Ao final, o fluxo deverá estar claro:

```text
Necessidade
    |
    v
SPEC / Issue
    |
    v
Branch
    |
    v
Desenvolvimento
    |
    v
Commits
    |
    v
Push
    |
    v
Pull Request
    |
    +--> revisão
    +--> testes
    +--> correções
    |
    v
Merge
    |
    v
main
```

---

## 2. GitHub não é Git

Git é o sistema de controle de versão.

GitHub é uma plataforma que utiliza Git e acrescenta recursos como:

- hospedagem de repositórios;
- Pull Requests;
- Issues;
- revisão de código;
- permissões;
- branch protection;
- GitHub Actions;
- releases;
- packages;
- environments;
- projetos;
- segurança;
- auditoria.

---

## 3. Repositório local e GitHub

Exemplo:

```text
Computador do desenvolvedor
        |
        | git push
        v
GitHub
        |
        | git pull / fetch
        v
Outros ambientes
```

O GitHub funciona como ponto central de colaboração, embora o Git continue sendo distribuído.

---

## 4. O que é uma Pull Request

Uma Pull Request é uma **proposta de integração de alterações de uma branch em outra**.

Exemplo:

```text
feature/dashboard
        |
        | Pull Request
        v
       main
```

A PR não é simplesmente "o código novo que substituirá o antigo".

Ela é um objeto de colaboração que reúne:

- diferença entre branches;
- commits;
- descrição da mudança;
- discussão;
- revisão;
- resultados dos testes;
- conflitos;
- aprovações;
- histórico das alterações;
- decisão de merge.

---

## 5. Anatomia de uma PR

Uma PR normalmente possui:

```text
Título
Descrição
Branch origem
Branch destino
Commits
Files changed
Checks
Reviewers
Comentários
Aprovações
Estado do merge
```

Exemplo:

```text
PR #42
"Refatorar dashboard de atendimento"

Base:
main

Compare:
feature/dashboard-v2
```

---

## 6. Base e compare

Em uma PR:

```text
base <- compare
```

Exemplo:

```text
main <- feature/login
```

Significa:

> Quero integrar as alterações de `feature/login` em `main`.

É importante verificar a direção antes de abrir a PR.

---

## 7. Criando uma branch

```bash
git switch main
git pull
git switch -c feature/dashboard
```

Desenvolva.

Depois:

```bash
git add .
git commit -m "feat: implementa novo dashboard"
git push -u origin feature/dashboard
```

A branch estará disponível no GitHub.

---

## 8. Abrindo a Pull Request

No GitHub:

```text
Repository
   |
   v
Pull requests
   |
   v
New pull request
```

Selecione:

```text
base: main
compare: feature/dashboard
```

Adicione título e descrição.

---

## 9. Uma PR não precisa ser integrada imediatamente

É possível abrir:

```text
PR #10
PR #11
PR #12
```

e continuar trabalhando.

Exemplo:

```text
main
 |
 +-- feature/A --> PR #10
 |
 +-- feature/B --> PR #11
 |
 +-- feature/C --> PR #12
```

As PRs podem permanecer abertas enquanto:

- código é desenvolvido;
- testes são executados;
- revisão ocorre;
- requisitos são ajustados.

---

## 10. Várias PRs simultâneas

Sim, é possível desenvolver várias PRs antes de fazer merge.

Porém, existem dois cenários.

### Cenário A: PRs independentes

```text
main
 |
 +-- feature/clientes
 |
 +-- feature/relatorios
 |
 +-- fix/login
```

As três podem ser integradas independentemente.

### Cenário B: PRs dependentes

```text
main
 |
 +-- feature/A
       |
       +-- feature/B
              |
              +-- feature/C
```

Nesse caso, B depende de A e C depende de B.

A ordem importa.

---

## 11. PRs independentes

São preferíveis quando possível.

Exemplo:

```text
PR #20 — altera tela de clientes
PR #21 — corrige timeout MQTT
PR #22 — adiciona relatório PDF
```

Se não compartilham dependências relevantes, cada uma pode ser:

- testada;
- revisada;
- aprovada;
- integrada;

separadamente.

---

## 12. PRs dependentes

Imagine:

```text
PR A
cria API /tickets

PR B
cria frontend que depende de /tickets
```

A PR B depende da A.

Uma estratégia:

```text
main
 |
 +-- feature/api-tickets
        |
        +-- feature/ui-tickets
```

A PR B pode inicialmente usar `feature/api-tickets` como base.

Depois que A for integrada, B pode ser atualizada para `main`.

---

## 13. Stacked Pull Requests

O modelo de PRs dependentes é frequentemente chamado de **stacked PRs**.

Exemplo:

```text
main
  |
  A
  |
 PR1
  |
  B
  |
 PR2
  |
  C
  |
 PR3
```

Vantagem:

- mudanças menores;
- revisão progressiva.

Desvantagem:

- gerenciamento mais complexo;
- dependências entre PRs;
- atualização de bases;
- conflitos em cascata.

---

## 14. Não fazer uma PR gigantesca

Uma PR com milhares de linhas e múltiplas funcionalidades é difícil de revisar.

Prefira:

```text
SPEC
 |
 +-- PR 1: backend
 |
 +-- PR 2: frontend
 |
 +-- PR 3: integração
```

quando a arquitetura permitir.

O objetivo não é produzir PRs artificialmente pequenas, mas manter unidades de mudança compreensíveis.

---

## 15. Draft Pull Request

Uma PR pode ser aberta como **Draft**.

Isso comunica:

> A implementação ainda não está pronta para merge, mas quero tornar o trabalho visível.

Útil para:

- CI antecipado;
- discussão;
- acompanhamento;
- revisão preliminar;
- desenvolvimento assistido por IA.

Fluxo:

```text
Draft PR
   |
desenvolvimento
   |
testes
   |
Ready for review
   |
aprovação
   |
merge
```

Importante:

- uma PR Draft **não pode ser mergeada** até ser marcada como "Ready for review" (o botão de merge fica desabilitado);
- workflows do GitHub Actions disparados por `pull_request` rodam normalmente em uma Draft, a menos que o workflow filtre explicitamente por `github.event.pull_request.draft`;
- é possível reverter uma PR de "Ready for review" de volta para Draft a qualquer momento, útil quando o CI aponta problemas sérios e você quer sinalizar "não revisem ainda".

---

## 16. Descrição da PR

Uma boa descrição deve responder:

```text
Por quê?
O que mudou?
Como testar?
Quais riscos?
Existe migração?
Existe impacto no deploy?
```

Modelo:

```markdown
### Objetivo

Implementar ...

### Alterações

- ...
- ...

### Como testar

1. ...
2. ...

### Critérios de aceitação

- [ ] ...
- [ ] ...

### Referências

SPEC:
Issue:
PR anterior:
```

---

## 17. Referenciar a PR anterior

Quando uma nova implementação ajusta algo criado anteriormente, é útil referenciar a PR original.

Exemplo:

```text
PR #42
Implementação inicial do dashboard
        |
        v
Uso real mostra problema de UX
        |
        v
Nova SPEC
        |
        v
PR #57
Refina frontend criado na PR #42
```

Na nova PR:

```text
Refina a implementação introduzida em #42.
```

Isso melhora a rastreabilidade.

---

## 18. SPEC → Branch → PR

Fluxo recomendado neste guia:

```text
SPEC-023
   |
   v
feature/023-dashboard
   |
   v
Commits
   |
   v
PR #57
```

A descrição da PR deve citar a SPEC.

Isso conecta:

```text
necessidade
   |
decisão
   |
implementação
   |
testes
   |
merge
```

---

## 19. Alteração posterior da SPEC

Uma implementação já integrada não deve normalmente ser "reativada" alterando silenciosamente a PR antiga.

Crie uma nova unidade de mudança:

```text
PR antiga
   |
   v
nova necessidade
   |
   v
nova SPEC
   |
   v
nova branch
   |
   v
nova PR
```

A nova SPEC pode explicar:

```text
Contexto:
A interface atual foi criada pela PR #42.

Problema:
...

Novo comportamento desejado:
...
```

---

## 20. Files changed

A aba **Files changed** mostra o diff da PR.

Exemplo:

```diff
- const timeout = 5000;
+ const timeout = 10000;
```

O reviewer deve avaliar:

- comportamento;
- arquitetura;
- segurança;
- legibilidade;
- testes;
- escopo;
- regressões potenciais.

---

## 21. Code Review

Code Review não é apenas procurar erros de sintaxe.

Uma revisão profissional verifica:

```text
Requisitos
Arquitetura
Segurança
Performance
Legibilidade
Testabilidade
Manutenibilidade
Compatibilidade
Migração
Observabilidade
```

---

## 22. Comentários de revisão

Um reviewer pode comentar uma linha específica.

Exemplo conceitual:

```text
Este acesso ao banco deveria estar no service,
não diretamente no controller.
```

O autor pode:

- responder;
- alterar;
- justificar;
- discutir;
- marcar como resolvido.

---

## 23. Aprovação

Dependendo das regras do repositório, uma PR pode exigir aprovação.

Exemplo:

```text
PR
 |
 +-- CI passou
 |
 +-- Review aprovado
 |
 +-- sem conflitos
 |
 v
Merge permitido
```

---

## 24. Checks

Os **Checks** são resultados de automações associadas à PR.

Exemplo:

```text
lint                 PASS
unit-tests           PASS
integration-tests    PASS
e2e-smoke            PASS
build                PASS
```

Esses checks normalmente serão executados pelo GitHub Actions.

O Volume 03 detalhará isso.

---

## 25. PR como ponto de controle

A PR funciona como uma fronteira entre:

```text
desenvolvimento
      |
      v
validação
      |
      v
integração
```

É um local adequado para exigir:

- testes;
- revisão;
- documentação;
- aprovação;
- políticas de segurança.

---

## 26. Merge

Quando a PR está pronta, suas alterações podem ser integradas.

As estratégias mais comuns no GitHub são:

```text
Merge commit
Squash and merge
Rebase and merge
```

Cada estratégia pode ser habilitada ou desabilitada individualmente nas configurações do repositório (**Settings → General → Pull Requests**). É recomendável permitir apenas a(s) estratégia(s) que o time realmente usa, para evitar inconsistência no histórico.

### Auto-merge

O GitHub permite habilitar **auto-merge** em uma PR: o merge é agendado automaticamente assim que todos os requisitos forem satisfeitos (checks obrigatórios verdes, aprovações necessárias, conversas resolvidas), sem exigir que alguém clique em "Merge" no momento exato em que o último check passa.

```text
PR
 |
 +-- Enable auto-merge
 |
 v
CI roda / review acontece
 |
 v
requisitos satisfeitos
 |
 v
merge automático (squash/merge commit/rebase, conforme escolhido)
```

Útil quando:

- o CI é lento e ninguém quer ficar monitorando manualmente;
- a PR já foi aprovada, mas ainda falta um check demorado (ex.: E2E);
- se agenda o merge e a pessoa segue para outra tarefa.

Auto-merge **não ignora** branch protection: se a branch `main` exige aprovação e checks obrigatórios, o merge automático só ocorre depois que todas as exigências forem cumpridas.

---

## 27. Merge commit

Preserva explicitamente a estrutura da branch.

Exemplo:

```text
A---B---------M
     \       /
      C---D---E
```

Vantagem:

- histórico completo da branch.

Desvantagem:

- histórico principal pode ficar mais complexo.

---

## 28. Squash and merge

Combina os commits da PR em um único commit.

Branch:

```text
fix typo
ajusta css
corrige teste
finaliza tela
```

Main após squash:

```text
feat: implementa nova tela de atendimento
```

É uma estratégia muito útil para manter `main` legível.

---

## 29. Rebase and merge

Reposiciona os commits da branch sobre a branch base sem criar merge commit.

Resultado linear:

```text
A---B---C---D---E
```

Pode ser adequado quando os commits já estão bem organizados.

---

## 30. Estratégia recomendada inicialmente

Para projetos pequenos/médios com desenvolvimento assistido por IA:

```text
Squash and merge
```

é uma escolha prática.

Motivos:

- PR representa uma unidade lógica;
- commits intermediários podem ser experimentais;
- histórico de `main` permanece limpo;
- rollback por feature fica mais simples.

Não é uma regra universal.

---

## 31. Delete branch

Depois do merge, a branch de feature normalmente pode ser removida.

Exemplo:

```text
feature/dashboard
```

já foi integrada.

A exclusão reduz branches obsoletas.

O histórico da PR continua existindo no GitHub.

---

## 32. Conflitos em PR

Exemplo:

```text
PR #10 altera dashboard.js
PR #11 altera a mesma região
```

Se #10 for integrada primeiro, #11 pode ficar em conflito.

A PR indicará que precisa ser atualizada.

---

## 33. Atualizar branch

Uma opção:

```bash
git switch feature/minha-feature
git fetch origin
git merge origin/main
```

Outra estratégia:

```bash
git rebase origin/main
```

A escolha deve seguir a política do projeto.

---

## 34. Branch Protection

A branch `main` pode ser protegida.

Objetivo:

```text
ninguém modifica main
sem passar pelo processo definido
```

> **Regra de ouro deste guia:** ninguém (nem humano, nem agente de IA) commita direto na `main`. Toda alteração nasce em uma branch e chega em `main` exclusivamente por Pull Request. Isso vale mesmo para "correções pequenas", "só um ajuste de config" ou trabalho solo: sem exceção.

Possíveis regras (configuráveis em **Settings → Branches** ou, no modelo mais novo, em **Settings → Rules → Rulesets**):

- exigir PR antes de integrar (`Require a pull request before merging`);
- exigir número mínimo de aprovações (`Require approvals`);
- exigir revisão do CODEOWNERS quando o caminho alterado tiver dono definido;
- descartar aprovações antigas quando novos commits chegarem (`Dismiss stale pull request approvals`);
- exigir checks obrigatórios (`Require status checks to pass`);
- exigir que a branch esteja atualizada com a base antes do merge (`Require branches to be up to date before merging`);
- exigir resolução de todas as conversas (`Require conversation resolution before merging`);
- exigir commits assinados (`Require signed commits`);
- exigir histórico linear (`Require linear history`, que bloqueia merge commit e força squash ou rebase);
- exigir merge queue (`Require merge queue`);
- bloquear force push (`Block force pushes`);
- impedir exclusão da branch (`Restrict deletions`);
- restringir quem pode fazer push direto, mesmo com bypass de PR (`Restrict who can push to matching branches`);
- aplicar as regras também a administradores; sem isso, admins conseguem contornar a proteção.

**Rulesets** (recurso mais recente) fazem o mesmo papel das "classic branch protection rules", mas com vantagens: podem ser aplicados a múltiplos padrões de branch/tag de uma vez, têm histórico de auditoria próprio, suportam camadas (várias rulesets somam restrições) e podem ser reaproveitados via API/Terraform entre repositórios. Para projetos novos, prefira Rulesets; branch protection rules clássicas continuam funcionando, mas são o modelo antigo.

---

## 35. Main protegida

Modelo:

```text
Developer
   |
   X
push direto bloqueado
   |
   v
Branch
   |
   v
PR
   |
   v
Checks
   |
   v
Review
   |
   v
Merge
   |
   v
main
```

Isso torna o processo previsível.

---

## 36. Required Status Checks

Podemos definir que determinados testes precisam passar.

Exemplo:

```text
Required:

ci / lint
ci / unit
ci / integration
e2e / smoke
```

Se um falhar:

```text
merge bloqueado
```

Pontos importantes:

- o nome do check exigido precisa bater exatamente com o nome do job (ou do `name:` do workflow) que o GitHub Actions reporta; renomear um job sem atualizar a regra faz a PR ficar travada esperando um check que nunca mais aparece;
- é possível marcar `Require branches to be up to date before merging`, o que força atualizar a branch com a base antes de mergear; assim evita-se integrar uma combinação de código que nunca foi testada junta;
- checks obrigatórios que ainda não iniciaram (por exemplo, porque o workflow só roda em certas condições) também bloqueiam o merge; vale revisar os `paths`/`if` do workflow para não travar PRs que nunca vão disparar aquele check.

---

## 37. Require Pull Request Reviews

Uma regra pode exigir:

```text
1 approval
```

ou mais, e pode combinar com:

- `Dismiss stale pull request approvals when new commits are pushed`: qualquer novo commit invalida aprovações anteriores, forçando nova revisão;
- `Require review from Code Owners`: quando o caminho alterado tem dono definido em CODEOWNERS, a aprovação daquele dono é obrigatória além (ou no lugar) da aprovação genérica;
- `Require approval of the most recent reviewable push`: impede que o próprio autor aprove sua última alteração usando uma conta secundária.

Para um desenvolvedor individual, exigir aprovação de terceiros pode não ser necessário em todos os projetos. Mas isso **não dispensa a PR**. Mesmo sozinho, a proteção da `main` continua ativa: push direto bloqueado, PR obrigatória, checks obrigatórios. O que muda é apenas o número de aprovações humanas exigidas (podendo ser zero), nunca a obrigatoriedade do fluxo branch → PR → merge.

Mesmo trabalhando sozinho, a PR continua útil para:

- CI;
- histórico;
- documentação;
- revisão assistida por IA;
- rastreabilidade.

---

## 38. Desenvolvedor individual

Uma estrutura eficiente:

```text
main protegida
      |
feature branch
      |
PR
      |
automação
      |
revisão própria/IA
      |
merge
```

A PR não é exclusiva de equipes.

Ela também disciplina desenvolvimento individual.

---

## 39. Issues

Issue representa normalmente:

- bug;
- feature;
- tarefa;
- melhoria;
- investigação.

Exemplo:

```text
Issue #81
"Dashboard demora para carregar"
```

Uma branch pode ser criada para resolver:

```text
fix/81-dashboard-performance
```

PR:

```text
Fixes #81
```

Quando integrada, a Issue pode ser fechada automaticamente.

---

## 40. Issues e SPECs

Para mudanças simples:

```text
Issue
 |
 v
Branch
 |
 v
PR
```

Para mudanças complexas:

```text
Issue
 |
 v
SPEC
 |
 v
Plano
 |
 v
Branch
 |
 v
PR
```

---

## 41. Templates de Issue

Estrutura sugerida para bug:

```markdown
### Problema

### Comportamento atual

### Comportamento esperado

### Passos para reproduzir

### Ambiente

### Evidências

### Impacto
```

---

## 42. Template de feature

```markdown
### Objetivo

### Problema que será resolvido

### Comportamento esperado

### Critérios de aceitação

### Restrições

### Fora de escopo
```

---

## 43. Template de Pull Request

Arquivo:

```text
.github/pull_request_template.md
```

Exemplo:

```markdown
### Objetivo

### Alterações realizadas

### SPEC / Issue relacionada

### Como testar

### Testes executados

- [ ] Lint
- [ ] Unit
- [ ] Integration
- [ ] E2E pertinente

### Checklist

- [ ] Sem secrets
- [ ] Documentação atualizada
- [ ] Critérios de aceitação atendidos
- [ ] Sem mudanças fora do escopo

### Riscos / observações
```

---

## 44. CODEOWNERS

Arquivo (qualquer um destes caminhos é reconhecido; a raiz e `.github/` são as mais comuns):

```text
.github/CODEOWNERS
CODEOWNERS
docs/CODEOWNERS
```

Permite associar partes do repositório a responsáveis.

Exemplo:

```text
## regra mais específica vence — a última linha que casar com o arquivo é a que vale
*           @owner-padrao
/backend/   @equipe-backend
/frontend/  @equipe-frontend
/infra/*.yml @equipe-devops @owner-padrao
```

Pontos importantes:

- a pessoa ou time listado precisa ter permissão de **write** (ou superior) no repositório, senão não pode ser exigido como reviewer;
- o efeito prático de CODEOWNERS só existe quando a branch protection/ruleset marca `Require review from Code Owners`; sem essa opção, o arquivo é só documentação;
- times (`@org/equipe-backend`) exigem que o time tenha acesso explícito ao repositório;
- em caso de padrões conflitantes, **a última regra que casar no arquivo prevalece** (não a mais específica por convenção lógica, e sim por ordem de declaração).

Em projetos individuais, pode não ser necessário inicialmente. Mas passa a fazer sentido assim que colaboradores externos ou agentes de IA começam a abrir PRs em áreas sensíveis (ex.: `infra/`, `.github/workflows/`).

---

## 45. Labels de Issues e PRs

Exemplos:

```text
bug
feature
documentation
security
frontend
backend
e2e
ci
deployment
priority-high
```

Labels facilitam busca e organização.

---

## 46. Milestones

Milestones agrupam Issues e PRs relacionadas a um objetivo.

Exemplo:

```text
Release 2.0
 |
 +-- Issue #80
 +-- Issue #81
 +-- PR #90
 +-- PR #92
```

---

## 47. Releases

Uma release representa uma versão distribuível do software.

Exemplo:

```text
v1.4.0
```

Ela pode estar associada a:

- tag Git;
- changelog;
- artifacts;
- notas de versão.

No pipeline futuro, uma release poderá disparar automações.

---

## 48. GitHub CLI

O GitHub possui CLI oficial:

```text
gh
```

Exemplos:

```bash
gh auth login
gh repo view
gh pr list
gh pr view 42
gh pr checks 42
```

Criar PR:

```bash
gh pr create
```

Isso é especialmente útil em automações e agentes de IA.

---

## 49. PR e GitHub Actions

Ao abrir uma PR:

```text
Pull Request
      |
      v
evento pull_request
      |
      v
GitHub Actions
      |
      v
workflow
      |
      v
runner
      |
      +-- lint
      +-- unit
      +-- integration
      +-- E2E
```

Os resultados voltam para a PR como checks.

---

## 50. PR e E2E

Não é obrigatório executar toda a suíte E2E em cada PR.

Estratégia:

```text
PR
 |
 +-- lint
 +-- unit
 +-- integration
 +-- smoke E2E
```

Depois:

```text
DEV
 |
 +-- E2E relacionado
```

E periodicamente:

```text
nightly
 |
 +-- full E2E
```

---

## 51. PR e deploy DEV

Uma possibilidade:

```text
PR
 |
 v
CI
 |
 v
Merge
 |
 v
main
 |
 v
Deploy DEV
```

Outra possibilidade para projetos específicos é criar ambientes temporários por PR.

Esse modelo será tratado em volumes avançados.

---

## 52. DEV → aprovação → PROD

Modelo adotado neste guia:

```text
Merge
 |
 v
Deploy DEV
 |
 v
Validação
 |
 v
Pergunta de aprovação
 |
 +------ NÃO
 |
 +------ SIM
           |
           v
          PROD
```

A autorização humana será implementada tecnicamente usando mecanismos apropriados do GitHub/deploy.

---

## 53. PR não é ambiente

É importante separar conceitos.

PR:

```text
objeto de colaboração e integração
```

DEV:

```text
ambiente onde a aplicação executa
```

Runner:

```text
máquina/agente que executa jobs
```

GitHub Actions:

```text
motor de automação
```

E2E:

```text
tipo de teste
```

---

## 54. PR não substitui branch

A PR aponta para alterações existentes em branches.

```text
branch
  |
  v
PR
```

Se novos commits forem enviados para a mesma branch, a PR normalmente é atualizada automaticamente.

---

## 55. Continuar desenvolvendo depois de abrir a PR

Fluxo:

```text
abrir PR
   |
   v
CI falha
   |
   v
corrigir localmente
   |
   v
commit
   |
   v
push
   |
   v
mesma PR atualizada
   |
   v
CI executa novamente
```

Não é necessário criar uma nova PR para cada correção durante o desenvolvimento daquela mesma mudança.

---

## 56. Quando criar uma nova PR

Crie uma nova PR quando surgir uma **nova unidade lógica de mudança**.

Exemplo:

```text
PR #42
novo dashboard
```

Foi integrada.

Duas semanas depois:

```text
o frontend precisa ser redesenhado
```

Crie:

```text
nova SPEC
nova branch
nova PR
```

e referencie #42.

---

## 57. Quando não reaproveitar PR antiga

Não reutilize uma PR integrada como recipiente genérico de alterações futuras.

A PR deve continuar representando historicamente a mudança que foi feita naquele momento.

A rastreabilidade depende disso.

---

## 58. PR pequena versus PR grande

Uma boa PR deve ser:

```text
grande o suficiente para entregar uma unidade útil
pequena o suficiente para ser compreendida
```

Não existe número universal de linhas.

Avalie complexidade cognitiva.

---

## 59. Critérios de aceitação

A PR deve demonstrar atendimento aos critérios definidos na SPEC.

Exemplo:

```text
- usuário consegue autenticar;
- erro é exibido para senha inválida;
- sessão expira após período configurado;
- fluxo possui teste E2E.
```

Os testes automatizados devem cobrir o que for tecnicamente adequado.

---

## 60. Evidências

Para frontend, uma PR pode incluir:

- screenshots;
- vídeos curtos;
- resultados de testes;
- logs;
- comparação visual.

Isso reduz ambiguidade na revisão.

---

## 61. Frontend e revisão humana

E2E pode confirmar:

```text
botão existe
clique funciona
formulário envia
resposta aparece
```

Mas não garante necessariamente:

```text
design agradável
alinhamento desejado
identidade visual correta
experiência ideal
```

Por isso frontend frequentemente exige revisão visual humana ou testes visuais específicos.

---

## 62. E2E passou, mas frontend ficou ruim

Isso não significa necessariamente que o E2E falhou.

Pode significar que os critérios testados não incluíam aspectos visuais suficientes.

Fluxo correto:

```text
PR original integrada
       |
       v
avaliação visual
       |
       v
nova SPEC de frontend
       |
       v
nova PR
```

---

## 63. Testes visuais

Posteriormente podemos adicionar:

```text
visual regression testing
```

Fluxo:

```text
screenshot baseline
       |
       v
nova execução
       |
       v
comparação
       |
       v
diferença detectada
```

Isso complementa E2E funcional.

---

## 64. PR e agentes de IA

Fluxo assistido:

```text
Necessidade
   |
   v
Agente faz perguntas
   |
   v
SPEC
   |
   v
Revisão humana
   |
   v
Plano
   |
   v
Branch
   |
   v
Implementação assistida
   |
   v
PR
   |
   v
CI + E2E
   |
   v
Review
```

A IA não elimina a necessidade de rastreabilidade.

Na prática, torna essa rastreabilidade ainda mais importante.

---

## 65. Contexto para uma nova implementação

Ao pedir a um agente para alterar uma funcionalidade existente, forneça:

```text
SPEC original
PR original
problema atual
novo comportamento esperado
restrições
critérios de aceitação
```

Isso reduz o risco de reinterpretar incorretamente o sistema.

---

## 66. Branch Protection e IA

Se um agente possui capacidade de fazer commits, a `main` protegida impede que uma implementação seja integrada diretamente sem passar pelos gates.

Arquitetura:

```text
Agente
 |
 v
branch
 |
 v
PR
 |
 v
checks
 |
 v
aprovação
 |
 v
main
```

---

## 67. Permissões mínimas

Agentes, Actions e tokens devem receber apenas as permissões necessárias.

Exemplo:

```text
contents: read
```

é preferível a permissões amplas quando o job só precisa ler código.

Segurança de tokens será aprofundada posteriormente.

---

## 68. PRs de dependências

Ferramentas automáticas podem abrir PRs para atualizar dependências.

Exemplo:

```text
Dependabot
   |
   v
PR
   |
   v
CI
   |
   v
Review
```

Não faça merge automático indiscriminado sem política definida.

---

## 69. Concurrency

Imagine novos commits chegando rapidamente na mesma PR.

Execuções antigas podem se tornar inúteis.

No Volume 03 veremos:

```yaml
concurrency:
  group: ...
  cancel-in-progress: true
```

Isso pode cancelar pipelines antigos e economizar tempo de runner.

---

## 70. PR e custos de CI

Cada push pode disparar novos testes.

Em runners hospedados isso pode consumir minutos do plano.

Em self-hosted:

```text
GitHub coordena
       |
       v
sua máquina executa
```

Isso reduz dependência dos minutos hospedados para os jobs migrados, mas transfere custo de hardware, energia e administração.

---

## 71. Estratégia de PR para reduzir testes

Evite:

```text
commit
push
espera
commit
push
espera
```

quando mudanças ainda estão claramente incompletas.

Uma abordagem:

```text
desenvolvimento local
   |
testes locais rápidos
   |
commit(s)
   |
push
   |
CI
```

Mas não deixe de fazer pushes úteis por medo do CI.

O equilíbrio depende do projeto.

---

## 72. Draft + CI seletivo

Uma evolução futura:

```text
Draft PR
 |
 +-- lint
 +-- unit
```

Quando:

```text
Ready for review
```

executar:

```text
integration
E2E
build completo
```

Isso pode economizar recursos.

---

## 73. Merge Queue

Em equipes ou repositórios com muitas PRs simultâneas, uma fila de merge (**merge queue**) ajuda a validar a combinação das mudanças antes da integração final.

Funcionamento resumido:

```text
PR aprovada + checks OK
        |
        v
entra na merge queue
        |
        v
GitHub cria um "merge temporário" com main + PR
        |
        v
roda os checks obrigatórios nessa combinação
        |
        v
se passar --> merge real em main
se falhar  --> PR removida da fila, autor é avisado
```

Isso evita o problema clássico de "duas PRs passaram isoladamente, mas juntas quebram main": cada PR na fila é testada já combinada com o estado mais recente de `main` e com as PRs que entraram antes dela na fila.

É habilitada em **Settings → Branches/Rules → Require merge queue**, exige repositório com Actions habilitado e é mais relevante a partir do momento em que várias PRs concorrem para a mesma branch. Em um projeto solo ou com poucas PRs simultâneas normalmente não compensa a complexidade extra.

---

## 74. Regras recomendadas para este guia

Inicialmente:

```text
main protegida (Ruleset ou branch protection classica)
push direto na main sempre bloqueado, sem excecao
PR obrigatória
CI obrigatório (status checks)
branch atualizada com a base antes do merge
conversas resolvidas
squash merge preferencial
branch excluída após merge
produção com gate humano
```

A exigência de reviewer humano (número de aprovações, CODEOWNERS) poderá variar conforme equipe/projeto. O que **não** varia é a proibição de commit direto na `main`: essa regra permanece ativa mesmo em projetos individuais ou com zero aprovações exigidas.

---

## 75. Estrutura sugerida do repositório

```text
.github/
├── workflows/
├── ISSUE_TEMPLATE/
├── pull_request_template.md
└── CODEOWNERS

docs/
├── specs/
└── architecture/

src/
tests/
```

---

## 76. Diretório de SPECs

Exemplo:

```text
docs/specs/
├── SPEC-001-login.md
├── SPEC-002-dashboard.md
└── SPEC-003-mqtt-events.md
```

Uma PR cita:

```text
SPEC-003
```

---

## 77. Nome de branch relacionado à SPEC

Exemplo:

```text
feature/003-mqtt-events
```

Isso torna a relação evidente.

Não é obrigatório, mas pode ajudar.

---

## 78. Título de PR

Prefira:

```text
feat: adiciona gerenciamento de eventos MQTT
```

em vez de:

```text
mudanças
```

O título frequentemente aparecerá no histórico, changelog ou squash commit. No **squash and merge**, o GitHub usa o título da PR (e a lista de commits, se houver mais de um) como sugestão de mensagem do commit final, editável antes de confirmar.

Vale adotar o padrão **Conventional Commits** (`feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`, `ci:`) tanto nos commits individuais quanto no título da PR. Vantagens práticas:

- gera changelogs automáticos de forma consistente;
- permite automações (ex.: bump de versão semântica) baseadas no tipo do commit;
- facilita ler `git log` depois do squash, já que o título da PR normalmente vira o commit único em `main`.

Isso não é exigido pelo GitHub. É uma convenção de repositório, geralmente reforçada por lint de commit/PR (ex.: uma Action que valida o título contra o padrão) rodando como check obrigatório.

---

## 79. Checklist operacional da PR

Antes de abrir:

- [ ] Branch correta.
- [ ] SPEC revisada.
- [ ] Código implementado.
- [ ] Testes locais pertinentes.
- [ ] Diff revisado.
- [ ] Nenhum secret.
- [ ] Documentação atualizada.

Depois de abrir:

- [ ] CI iniciou.
- [ ] Checks relevantes passaram.
- [ ] Falhas foram investigadas.
- [ ] Comentários foram tratados.
- [ ] Critérios de aceitação foram confirmados.
- [ ] PR está pronta para merge.

Depois do merge:

- [ ] Branch pode ser excluída.
- [ ] Deploy DEV ocorreu conforme política.
- [ ] Validação pós-deploy ocorreu.
- [ ] Produção só foi liberada se autorizada.
- [ ] Issue/SPEC ficou rastreável.

---

## 80. Exemplo completo

Necessidade:

```text
Melhorar a tela de chamados.
```

SPEC:

```text
SPEC-014
```

Branch:

```bash
git switch -c feature/014-chamados-ui
```

Desenvolvimento:

```text
frontend
testes
documentação
```

Commits:

```text
feat: reorganiza painel de chamados
test: adiciona E2E do filtro de chamados
```

Push:

```bash
git push -u origin feature/014-chamados-ui
```

PR:

```text
PR #88
feat: reorganiza painel de chamados
```

Checks:

```text
lint         PASS
unit         PASS
e2e-smoke    PASS
```

Review:

```text
ajustar espaçamento mobile
```

Novo commit:

```text
fix: ajusta layout mobile do painel
```

Push:

```text
mesma PR atualizada
```

Checks passam.

Merge:

```text
Squash and merge
```

Resultado:

```text
main contém a nova interface
```

---

## 81. Exemplo de refinamento posterior

Depois de uso real:

```text
O frontend não ficou como desejado.
```

Não alteramos a história da PR #88.

Criamos:

```text
SPEC-021
```

Contexto:

```text
Refina a interface introduzida pela PR #88.
```

Nova branch:

```text
feature/021-refino-chamados-ui
```

Nova PR:

```text
PR #103
```

Assim:

```text
#88 implementação original
 |
 v
#103 refinamento
```

O histórico conta a evolução real do sistema.

---

## 82. Relação com o próximo volume

Até aqui:

```text
Branch
   |
   v
PR
```

Agora precisamos responder:

> Quem executa automaticamente os testes da PR?

Resposta:

```text
GitHub Actions
```

Fluxo:

```text
PR
 |
 v
GitHub Actions
 |
 v
Workflow
 |
 v
Job
 |
 v
Runner
 |
 v
Steps
```

Esse será o **Volume 03**.

---

## 83. Resumo

Pull Request é uma proposta rastreável de integração entre branches.

Ela permite centralizar:

- código;
- contexto;
- revisão;
- testes;
- discussão;
- aprovação;
- merge.

Para o fluxo deste projeto:

```text
SPEC
 |
 v
Branch
 |
 v
Implementação
 |
 v
PR
 |
 v
GitHub Actions
 |
 v
Testes
 |
 v
Merge
 |
 v
DEV
 |
 v
Validação
 |
 v
Aprovação
 |
 v
PROD
```

---

## 84. Próximo volume

**Volume 03: GitHub Actions**

Conteúdo previsto:

- workflows;
- YAML;
- eventos;
- `pull_request`;
- `push`;
- `workflow_dispatch`;
- jobs;
- steps;
- actions;
- runners;
- secrets;
- variables;
- cache;
- artifacts;
- matrices;
- conditions;
- dependencies;
- concurrency;
- environments;
- gates;
- CI;
- integração com self-hosted runners.

---

## Fontes

### Pull Requests: conceito, anatomia e ciclo de vida

- [About pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests) — definição oficial de PR como proposta de integração entre branches (base/compare), usada nas seções 4 a 8.
- [Changing the stage of a pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/changing-the-stage-of-a-pull-request) — comprova o comportamento de Draft PR, o botão de merge desabilitado e a possibilidade de reverter para Draft (seção 15).
- [Linking a pull request to an issue](https://docs.github.com/en/issues/tracking-your-work-with-issues/using-issues/linking-a-pull-request-to-an-issue) — sustenta o uso de palavras-chave como `Fixes #81` para fechar Issues automaticamente ao integrar a PR (seção 39).

### Code Review e aprovação

- [About pull request reviews](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/about-pull-request-reviews) — base para os conceitos de comentário, aprovação, "request changes" e exigência de aprovações antes do merge (seções 21 a 23, 37).

### Branch Protection e Rulesets

- [About protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches) — lista oficial das regras de proteção citadas (exigir PR, aprovações, dismiss de aprovações antigas, commits assinados, histórico linear, bloqueio de force push, restrição de push direto, aplicação a administradores) e a seção "Require status checks before merging" usada nas seções 34 a 36.
- [About rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) — sustenta a comparação entre Rulesets e branch protection clássica (múltiplos padrões, auditoria, camadas, reuso via API) na seção 34.

### CODEOWNERS

- [About code owners](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners) — confirma localização do arquivo (`.github/`, raiz, `docs/`), exigência de permissão de write, dependência da opção "Require review from Code Owners" e a regra de que a última linha que casar prevalece (seção 44).

### Merge, Auto-merge e Merge Queue

- [About pull request merges](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/about-pull-request-merges) — descreve as três estratégias (merge commit, squash and merge, rebase and merge) usadas nas seções 26 a 29.
- [Automatically merging a pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/automatically-merging-a-pull-request) — sustenta o comportamento de auto-merge (agendamento automático após checks e aprovações, sem ignorar branch protection) na seção 26.
- [Managing a merge queue](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue) — base para o funcionamento da merge queue (merge temporário testado contra `main` + PRs anteriores na fila) na seção 73.

### Templates de PR

- [Creating a pull request template for your repository](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository) — confirma o caminho `.github/pull_request_template.md` usado na seção 43.

### GitHub CLI

- [GitHub CLI manual](https://cli.github.com/manual/) — documentação oficial do `gh`, referência dos comandos `gh pr create`, `gh pr list`, `gh pr view`, `gh pr checks` usados na seção 48.

---

**Fim do Volume 02: GitHub e Pull Requests**
