# Volume 03 — GitHub Actions: CI, Testes e Automação

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 03_GITHUB_ACTIONS.md  
**Versão:** 0.1.0  
**Status:** Primeira versão para expansão incremental  
**Pré-requisitos:** Volumes 01 e 02

---

## 1. Objetivo

Este volume explica como o GitHub Actions transforma eventos do repositório em automações executáveis.

Fluxo central:

```text
Evento GitHub
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
     |
     +-- checkout
     +-- instalar dependências
     +-- lint
     +-- testes
     +-- build
     +-- artifacts
```

No pipeline completo:

```text
Branch
  |
  v
Pull Request
  |
  v
GitHub Actions
  |
  v
CI
  |
  +-- lint
  +-- unit
  +-- integration
  +-- smoke E2E
  |
  v
Merge
  |
  v
Deploy DEV
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

# 2. O que é GitHub Actions

GitHub Actions é a plataforma de automação integrada ao GitHub.

Ela permite executar tarefas quando determinados eventos acontecem.

Exemplos:

```text
PR aberta
   |
   v
executar testes
```

```text
push em main
   |
   v
build
   |
   v
deploy DEV
```

```text
aprovação
   |
   v
deploy PROD
```

---

# 3. GitHub Actions não é o teste

GitHub Actions é o orquestrador.

O teste é executado por ferramentas específicas.

Exemplos:

```text
GitHub Actions
     |
     +-- Vitest
     +-- Jest
     +-- PHPUnit
     +-- Playwright
     +-- Cypress
```

Portanto, o GitHub não inventa automaticamente os testes do seu sistema.

É necessário definir:

- quais testes existem;
- como são executados;
- quando são executados;
- quais resultados bloqueiam o merge.

---

# 4. Workflow

Um workflow é um arquivo YAML localizado normalmente em:

```text
.github/workflows/
```

Exemplo:

```text
.github/
└── workflows/
    ├── ci.yml
    ├── e2e.yml
    └── deploy.yml
```

Cada arquivo descreve uma automação.

---

# 5. Exemplo mínimo

```yaml
name: CI

on:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Teste simples
        run: echo "CI funcionando"
```

Conceito:

```text
PR
 |
 v
workflow CI
 |
 v
job test
 |
 v
runner ubuntu
 |
 v
steps
```

---

# 6. Anatomia do workflow

Elementos principais:

```yaml
name:
on:
permissions:
env:
concurrency:
jobs:
```

Dentro de cada job:

```yaml
runs-on:
needs:
if:
environment:
strategy:
steps:
```

---

# 7. name

Define o nome apresentado na interface.

```yaml
name: CI Backend
```

Prefira nomes claros:

```text
CI Backend
CI Frontend
E2E Smoke
Deploy DEV
Deploy PROD
Security Scan
```

---

# 8. on

Define os eventos que disparam o workflow.

Exemplo:

```yaml
on:
  pull_request:
```

Outro:

```yaml
on:
  push:
    branches:
      - main
```

---

# 9. pull_request

Muito importante para CI.

```yaml
on:
  pull_request:
    branches:
      - main
```

Fluxo:

```text
PR para main
    |
    v
CI
```

Pode ser executado quando a PR é:

- aberta;
- atualizada com novos commits;
- reaberta;
- sincronizada conforme o evento.

---

# 10. push

Exemplo:

```yaml
on:
  push:
    branches:
      - main
```

Fluxo:

```text
merge da PR
    |
    v
push em main
    |
    v
workflow
```

É adequado para ações pós-merge, como deploy DEV.

---

# 11. workflow_dispatch

Permite execução manual.

```yaml
on:
  workflow_dispatch:
```

Na interface:

```text
Actions
   |
   v
Run workflow
```

É útil para:

- deploy manual;
- manutenção;
- testes especiais;
- operações administrativas.

---

# 12. Inputs manuais

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: Ambiente
        required: true
        type: choice
        options:
          - dev
          - production
```

Isso permite selecionar parâmetros antes da execução.

---

# 13. schedule

Permite execução programada.

```yaml
on:
  schedule:
    - cron: '0 3 * * *'
```

Uso típico:

```text
03:00
 |
 v
suíte E2E completa
```

Muito útil para testes longos.

---

# 14. Vários eventos

```yaml
on:
  pull_request:
  push:
    branches:
      - main
  workflow_dispatch:
```

Um mesmo workflow pode responder a múltiplos eventos.

Mas nem sempre é a melhor organização.

Às vezes é preferível separar:

```text
ci.yml
deploy-dev.yml
deploy-prod.yml
```

---

# 15. jobs

Um workflow possui um ou mais jobs.

```yaml
jobs:

  lint:
    ...

  unit:
    ...

  integration:
    ...
```

Por padrão, jobs independentes podem executar em paralelo, dependendo de disponibilidade de runners e limites aplicáveis.

---

# 16. Job

Exemplo:

```yaml
jobs:
  unit-tests:
    runs-on: ubuntu-latest

    steps:
      - run: npm test
```

Cada job é atribuído a um runner compatível.

---

# 17. Runner

O runner é a máquina/agente que executa o job.

GitHub-hosted:

```yaml
runs-on: ubuntu-latest
```

Self-hosted:

```yaml
runs-on: self-hosted
```

Com labels:

```yaml
runs-on:
  - self-hosted
  - linux
  - x64
  - docker
```

---

# 18. Relação Actions e runner

```text
GitHub Actions
      |
      | agenda job
      v
Runner
      |
      | executa
      v
Steps
```

GitHub Actions coordena.

Runner executa.

---

# 19. GitHub-hosted runner

Modelo:

```text
job
 |
 v
máquina temporária
 |
 v
execução
 |
 v
máquina descartada
```

Vantagens:

- pouca administração;
- ambiente limpo;
- alta conveniência.

Desvantagens:

- consumo de minutos conforme plano/repositório;
- menor controle;
- custo pode crescer.

---

# 20. Self-hosted runner

Modelo:

```text
GitHub
 |
 v
seu servidor
 |
 v
runner
 |
 v
job
```

Vantagens:

- controle;
- hardware próprio;
- acesso à rede interna;
- caches persistentes;
- redução da dependência de minutos hospedados.

Desvantagens:

- administração;
- segurança;
- limpeza;
- disponibilidade.

O Volume 04 trata da instalação completa.

---

# 21. steps

Steps são as etapas do job.

```yaml
steps:

  - name: Checkout
    uses: actions/checkout@v4

  - name: Instalar
    run: npm ci

  - name: Testar
    run: npm test
```

Executam sequencialmente dentro do job, salvo condições específicas.

---

# 22. uses

`uses` executa uma Action reutilizável.

Exemplo:

```yaml
- uses: actions/checkout@v4
```

Outro:

```yaml
- uses: actions/setup-node@v4
```

---

# 23. run

Executa comandos de shell.

```yaml
- name: Testes
  run: npm test
```

Vários comandos:

```yaml
- name: Build
  run: |
    npm ci
    npm run build
```

---

# 24. Checkout

O runner não deve pressupor que o código da PR já está disponível.

Use:

```yaml
- name: Checkout
  uses: actions/checkout@v4
```

Essa etapa obtém o conteúdo necessário do repositório.

---

# 25. Setup Node

```yaml
- name: Configurar Node
  uses: actions/setup-node@v4
  with:
    node-version: 22
```

Isso documenta a versão esperada pelo pipeline.

---

# 26. npm ci

Em CI, normalmente:

```bash
npm ci
```

é preferível a:

```bash
npm install
```

quando existe lockfile adequado.

Objetivo:

- instalação reproduzível;
- respeito ao lockfile;
- comportamento apropriado para CI.

---

# 27. Exemplo CI Node.js

```yaml
name: CI Node

on:
  pull_request:
    branches:
      - main

jobs:

  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - run: npm ci

      - run: npm run lint

      - run: npm test

      - run: npm run build
```

---

# 28. Exemplo com self-hosted

Mudança principal:

```yaml
runs-on:
  - self-hosted
  - linux
  - x64
```

Workflow:

```yaml
name: CI Self Hosted

on:
  pull_request:

jobs:

  test:
    runs-on:
      - self-hosted
      - linux
      - x64

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - run: npm ci
      - run: npm test
```

---

# 29. Separar jobs

Em vez de:

```text
job gigante
 |
 +-- lint
 +-- unit
 +-- integration
 +-- e2e
 +-- build
```

podemos ter:

```text
lint
unit
integration
build
e2e
```

Vantagens:

- visualização clara;
- paralelização;
- diagnóstico rápido;
- políticas de checks específicas.

---

# 30. Paralelismo

Exemplo:

```yaml
jobs:

  lint:
    runs-on: self-hosted
    steps:
      - ...

  unit:
    runs-on: self-hosted
    steps:
      - ...
```

Se existirem runners suficientes, jobs podem executar simultaneamente.

Com apenas um runner self-hosted, os jobs podem ficar em fila.

---

# 31. needs

Cria dependência entre jobs.

```yaml
jobs:

  test:
    ...

  build:
    needs: test
    ...
```

Fluxo:

```text
test
 |
 PASS
 |
 v
build
```

Se `test` falhar, `build` normalmente não executa.

---

# 32. Pipeline em DAG

Exemplo:

```text
          lint
           |
           |
unit ------+------ integration
           |
           v
         build
           |
           v
          e2e
```

No YAML:

```yaml
build:
  needs:
    - lint
    - unit
    - integration
```

---

# 33. if

Permite execução condicional.

```yaml
if: github.ref == 'refs/heads/main'
```

Outro exemplo:

```yaml
if: always()
```

Muito útil em limpeza.

---

# 34. always()

Imagine:

```yaml
- name: Subir containers
  run: docker compose up -d

- name: Testar
  run: npm test

- name: Derrubar containers
  if: always()
  run: docker compose down -v
```

Mesmo se o teste falhar, o workflow tenta limpar o ambiente.

Isso é especialmente importante em runners persistentes.

---

# 35. success()

Executa se etapas anteriores estiverem bem-sucedidas.

```yaml
if: success()
```

---

# 36. failure()

Executa quando existe falha.

```yaml
if: failure()
```

Pode ser útil para:

- coletar logs;
- screenshots;
- diagnostics.

---

# 37. Environment variables

No nível do workflow:

```yaml
env:
  NODE_ENV: test
```

No job:

```yaml
jobs:
  test:
    env:
      DATABASE_NAME: app_test
```

No step:

```yaml
- run: npm test
  env:
    API_URL: http://localhost:3000
```

Prefira o menor escopo necessário.

---

# 38. Variables

Valores não sensíveis podem ser armazenados como variables do GitHub.

Exemplos:

```text
DEV_HOST
APP_PORT
NODE_ENV
```

Não utilize variables comuns para senhas.

---

# 39. Secrets

Informações sensíveis devem ser tratadas como secrets.

Exemplos:

```text
SSH_PRIVATE_KEY
DATABASE_PASSWORD
DEPLOY_TOKEN
API_SECRET
```

Uso:

```yaml
env:
  DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
```

---

# 40. Regra de secrets

Nunca:

```yaml
env:
  DB_PASSWORD: minha_senha_real
```

Nunca commitar:

```text
.env real
private key
token
password
```

Secrets devem ser rotacionados se forem expostos.

---

# 41. permissions

Workflows podem declarar permissões do `GITHUB_TOKEN`.

Exemplo:

```yaml
permissions:
  contents: read
```

Princípio:

```text
mínimo privilégio
```

Não conceda escrita se o job só precisa ler o repositório.

---

# 42. GITHUB_TOKEN

O GitHub fornece um token temporário para determinados usos durante o workflow.

As permissões dependem da configuração e devem ser controladas explicitamente quando possível.

Evite tratar o token automático como uma credencial ilimitada.

---

# 43. Cache

Instalar dependências repetidamente pode ser caro.

Exemplo Node:

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 22
    cache: npm
```

O cache pode reduzir o tempo de instalação.

---

# 44. Cache não é artifact

Cache:

```text
otimiza execuções futuras
```

Artifact:

```text
preserva resultado de uma execução
```

Exemplo cache:

```text
dependências
```

Exemplo artifact:

```text
relatório de testes
build
screenshots E2E
```

---

# 45. Artifacts

Podemos armazenar:

```text
coverage/
playwright-report/
dist/
logs/
screenshots/
```

Isso permite investigar falhas e reutilizar outputs.

---

# 46. Artifact de E2E

Quando Playwright falha, podemos preservar relatório.

Exemplo conceitual:

```yaml
- name: Upload relatório
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: playwright-report
    path: playwright-report/
```

---

# 47. Coverage

Exemplo:

```text
unit tests
    |
    v
coverage
    |
    v
artifact
```

Coverage não deve ser confundida com qualidade absoluta.

100% de cobertura não significa necessariamente bons testes.

---

# 48. Matrix

Permite testar combinações.

Exemplo:

```yaml
strategy:
  matrix:
    node:
      - 20
      - 22
```

Depois:

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: ${{ matrix.node }}
```

Fluxo:

```text
Node 20
Node 22
```

---

# 49. Matrix e custo

Matrices multiplicam jobs.

Exemplo:

```text
2 versões Node
x
3 bancos
=
6 combinações
```

Use somente quando a compatibilidade realmente precisa ser garantida.

---

# 50. Services

GitHub Actions pode disponibilizar containers auxiliares em cenários compatíveis.

Exemplo conceitual:

```yaml
services:
  redis:
    image: redis:alpine
```

Ou banco:

```yaml
services:
  mysql:
    image: mysql:8
```

Em self-hosted, requisitos do runner e Docker precisam ser considerados.

---

# 51. Docker Compose

Para sistemas mais complexos, podemos preferir:

```bash
docker compose -f docker-compose.test.yml up -d
```

Depois:

```bash
npm test
```

E limpeza:

```bash
docker compose -f docker-compose.test.yml down -v
```

---

# 52. Banco isolado para testes

Nunca execute testes automatizados destrutivos contra banco de produção.

Fluxo:

```text
CI
 |
 v
Banco temporário
 |
 v
migrations
 |
 v
fixtures
 |
 v
tests
 |
 v
destruição
```

---

# 53. MQTT em CI

Para aplicações de automação:

```text
workflow
   |
   v
Mosquitto container
   |
   +-- publish
   +-- subscribe
   +-- assertions
```

Isso permite testar integrações MQTT sem depender do broker de produção.

---

# 54. Lint

Lint verifica regras estáticas de código.

Exemplos:

```text
ESLint
PHP_CodeSniffer
```

Deve ser rápido.

Por isso normalmente é um dos primeiros gates.

---

# 55. Unit tests

Testes unitários devem ser rápidos e isolados.

Fluxo ideal:

```text
PR
 |
 v
unit
 |
 poucos minutos ou segundos
```

Eles formam a base da estratégia de testes.

---

# 56. Integration tests

Validam interação entre componentes.

Exemplos:

```text
API + banco
service + Redis
backend + MQTT
```

São mais caros que testes unitários.

---

# 57. E2E

E2E valida o sistema de ponta a ponta.

Exemplo:

```text
browser
 |
 v
frontend
 |
 v
backend
 |
 v
database
```

É poderoso, mas geralmente mais lento.

---

# 58. Por que E2E cresce

À medida que o sistema cresce:

```text
mais telas
mais fluxos
mais combinações
mais dados
mais browsers
```

a suíte pode ficar demorada.

Não devemos responder apenas adicionando hardware.

Também precisamos melhorar a estratégia.

---

# 59. Pirâmide de testes

```text
          E2E
        /-----\
       Integração
     /-----------\
       Unitários
```

A maior quantidade tende a estar na base.

Objetivo:

```text
muitos testes baratos
poucos testes caros e valiosos
```

---

# 60. Smoke E2E na PR

Na PR:

```text
login
fluxo principal
operação crítica
logout
```

Não necessariamente todas as variações.

---

# 61. Full E2E

Pode ser executado:

```text
nightly
release
manual
antes de deploy crítico
```

Isso reduz o tempo de feedback da PR.

---

# 62. Testes afetados

Em projetos maiores, uma evolução é identificar o que mudou.

Exemplo:

```text
frontend/chamados
       |
       v
testes de chamados
```

em vez de executar toda a aplicação.

Essa estratégia exige arquitetura e mapeamento adequados.

---

# 63. Concurrency

Novos commits podem tornar execuções antigas inúteis.

Exemplo:

```yaml
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
```

Fluxo:

```text
commit A -> CI A
commit B -> cancela CI A -> CI B
```

Isso reduz desperdício.

---

# 64. Concurrency em PR

Uma forma mais refinada pode utilizar contexto da PR/branch para agrupar execuções.

A expressão exata deve ser escolhida de acordo com os eventos usados no workflow.

Objetivo:

```text
somente a execução mais recente daquela linha de trabalho permanece ativa
```

---

# 65. timeout-minutes

Jobs podem travar.

Defina limite quando apropriado:

```yaml
timeout-minutes: 20
```

Isso evita consumo indefinido por:

- browser travado;
- serviço indisponível;
- teste esperando evento;
- deadlock.

---

# 66. continue-on-error

Pode permitir que uma etapa falhe sem marcar imediatamente todo o job como falho.

```yaml
continue-on-error: true
```

Use com cuidado.

Não esconda falhas importantes.

Adequado apenas para verificações explicitamente não bloqueantes.

---

# 67. Fail fast

Em matrices, pode ser útil controlar se outras combinações devem continuar quando uma falha ocorre.

A decisão depende do objetivo:

```text
feedback rápido
versus
coletar todos os resultados
```

---

# 68. Workflow para PR

Modelo:

```yaml
name: PR CI

on:
  pull_request:
    branches:
      - main

permissions:
  contents: read

concurrency:
  group: pr-ci-${{ github.ref }}
  cancel-in-progress: true

jobs:

  test:
    runs-on:
      - self-hosted
      - linux
      - x64

    timeout-minutes: 20

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build
```

---

# 69. Workflow E2E separado

```yaml
name: E2E Smoke

on:
  pull_request:
    branches:
      - main

jobs:

  e2e:
    runs-on:
      - self-hosted
      - linux
      - e2e

    timeout-minutes: 30

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - run: npm ci

      - run: npx playwright install --with-deps

      - run: npx playwright test --grep @smoke
```

A marcação `@smoke` depende da organização adotada nos testes.

---

# 70. Workflow noturno

Exemplo:

```yaml
name: Full E2E Nightly

on:
  schedule:
    - cron: '0 3 * * *'

  workflow_dispatch:
```

Job:

```yaml
- run: npx playwright test
```

Isso mantém a PR rápida e ainda executa regressão ampla regularmente.

---

# 71. Workflow de deploy DEV

Conceito:

```yaml
name: Deploy DEV

on:
  push:
    branches:
      - main
```

Fluxo:

```text
merge
 |
 v
main
 |
 v
Deploy DEV
```

O deploy deve depender de uma política clara de validação.

---

# 72. GitHub Environments

Podemos definir ambientes como:

```text
development
production
```

Eles ajudam a organizar:

- secrets;
- variables;
- proteção;
- aprovações;
- histórico de deployment.

---

# 73. Environment no job

```yaml
jobs:

  deploy:
    environment:
      name: production
```

Esse job passa a utilizar as regras associadas ao environment.

---

# 74. Gate de produção

Objetivo:

```text
DEV validado
     |
     v
aguarda aprovação
     |
   +----+
   |    |
 NÃO   SIM
        |
        v
       PROD
```

A política de aprovação deve ser configurada no ambiente/repositório conforme os recursos disponíveis na conta/plano.

---

# 75. Deploy não deve acontecer porque o teste "parece bom"

A decisão precisa ser explícita.

Pipeline:

```text
CI
 |
 PASS
 |
 v
DEV
 |
 PASS
 |
 v
aprovação
 |
 v
PROD
```

Isso separa:

```text
qualidade automatizada
```

de:

```text
decisão operacional
```

---

# 76. Deploy via SSH

Modelo conceitual:

```text
Runner
 |
 | SSH
 v
Servidor DEV
```

Credenciais devem estar em secrets.

Nunca:

```text
senha SSH no YAML
```

Preferir chaves dedicadas com privilégio mínimo.

---

# 77. Deploy com Docker

Exemplo conceitual no servidor:

```bash
docker compose pull
docker compose up -d
```

Ou:

```text
build image
push registry
deploy version
```

O modelo exato será aprofundado no Volume 09.

---

# 78. Health check pós-deploy

Após deploy:

```text
deploy
 |
 v
health check
 |
 +-- OK -> continuar
 |
 +-- FAIL -> falha/rollback
```

Exemplo simples:

```bash
curl --fail http://servidor/health
```

Em produção, o endpoint deve representar saúde real da aplicação.

---

# 79. Rollback

Todo deploy profissional precisa responder:

> Como voltar?

Exemplo:

```text
v1.4.0
 |
 deploy v1.5.0
 |
 falha
 |
 rollback
 |
 v1.4.0
```

O pipeline futuro deverá preservar referência à versão implantada.

---

# 80. Reusable workflows

Se vários repositórios usam a mesma lógica, podemos criar workflows reutilizáveis.

Objetivo:

```text
não copiar 200 linhas de YAML
para 10 projetos
```

Centralização deve ser feita com governança para não criar dependência frágil.

---

# 81. Composite Actions

Também é possível agrupar steps reutilizáveis em uma Action composta.

Útil para:

```text
setup comum
validação comum
deploy padronizado
```

A escolha entre composite action e reusable workflow depende do nível de reutilização desejado.

---

# 82. Expressions

GitHub Actions utiliza expressões:

```text
${{ ... }}
```

Exemplo:

```yaml
${{ github.ref }}
```

Outro:

```yaml
${{ secrets.DB_PASSWORD }}
```

---

# 83. Context github

Contém informações do evento.

Exemplos conceituais:

```text
github.ref
github.sha
github.actor
github.event_name
```

Esses dados permitem decisões condicionais.

---

# 84. SHA

Cada execução está relacionada a um commit.

```text
github.sha
```

Isso é importante para rastreabilidade:

```text
deploy
 |
 v
qual commit?
 |
 v
SHA
```

---

# 85. Versionar artifacts por SHA

Exemplo conceitual:

```text
app-a91c302.tar.gz
```

ou imagem:

```text
app:a91c302
```

Isso facilita identificar exatamente qual código foi implantado.

---

# 86. Imutabilidade

Uma prática forte:

```text
build uma vez
 |
 v
artifact
 |
 +-- DEV
 |
 +-- PROD
```

em vez de reconstruir versões diferentes em cada ambiente.

Isso reduz divergências.

Será aprofundado em deploy.

---

# 87. Segurança de Actions de terceiros

Exemplo:

```yaml
uses: empresa/action@v1
```

Uma Action externa executa código no pipeline.

Portanto, deve ser tratada como dependência de software.

Avalie:

- origem;
- manutenção;
- permissões;
- versão;
- risco de supply chain.

---

# 88. Pinning

Para ambientes de segurança elevada, pode-se fixar Actions por commit SHA.

Isso reduz o risco de uma tag mutável apontar para código inesperado.

A estratégia será aprofundada no Volume de Segurança.

---

# 89. Forks e self-hosted runners

Não permita automaticamente que código não confiável de terceiros execute em runner persistente com acesso à sua rede.

Risco:

```text
PR maliciosa
   |
   v
runner interno
   |
   +-- rede
   +-- Docker
   +-- arquivos
```

Self-hosted runner exige política de confiança.

---

# 90. pull_request_target

Eventos com contexto privilegiado exigem extremo cuidado.

Não execute código não confiável de uma PR usando secrets privilegiados apenas para facilitar o workflow.

Esse é um tema de segurança avançada.

---

# 91. Logs

Todo step gera logs.

Use nomes claros:

```yaml
- name: Executar migrations de teste
```

é melhor que:

```yaml
- name: Step 4
```

Logs são parte do diagnóstico operacional.

---

# 92. Debug

Ao investigar uma falha:

```text
qual workflow?
qual job?
qual step?
qual comando?
qual exit code?
qual commit?
qual runner?
```

Esse método reduz tentativas aleatórias.

---

# 93. Exit code

Em shell:

```text
0 = sucesso
diferente de 0 = erro
```

GitHub Actions utiliza isso para determinar sucesso/falha de comandos.

Um script que engole erros pode gerar falso positivo.

---

# 94. Bash seguro

Scripts administrativos podem usar:

```bash
set -euo pipefail
```

Mas é necessário compreender o efeito antes de aplicar indiscriminadamente.

O objetivo é evitar continuar após erros silenciosos.

---

# 95. Scripts versus YAML gigante

Evite colocar toda a lógica dentro do workflow.

Ruim:

```text
400 linhas de shell dentro do YAML
```

Melhor:

```text
.github/workflows/deploy.yml
scripts/deploy.sh
scripts/health-check.sh
```

Benefícios:

- testes locais;
- legibilidade;
- reutilização;
- revisão.

---

# 96. Makefile ou scripts npm

Uma boa estratégia é fazer o CI chamar os mesmos comandos usados localmente.

Exemplo:

```bash
npm run lint
npm run test:unit
npm run test:integration
npm run test:e2e:smoke
```

Assim:

```text
local
   |
   +-- mesmos comandos
   |
CI
```

Reduz divergências.

---

# 97. "Funciona na minha máquina"

Docker, lockfiles e scripts padronizados ajudam a reduzir esse problema.

Objetivo:

```text
local
DEV
CI
PROD
```

utilizarem componentes e processos tão reproduzíveis quanto possível.

---

# 98. Estratégia recomendada para o projeto

Primeira versão:

```text
PR
 |
 +-- lint
 +-- unit
 +-- integration
 +-- build
 +-- smoke E2E
```

Depois do merge:

```text
main
 |
 v
Deploy DEV
```

Após validação:

```text
aprovação humana
 |
 v
PROD
```

Nightly:

```text
full E2E
```

---

# 99. Estrutura sugerida

```text
.github/
└── workflows/
    ├── ci.yml
    ├── e2e-smoke.yml
    ├── e2e-full.yml
    ├── deploy-dev.yml
    └── deploy-prod.yml

scripts/
├── ci/
├── deploy/
└── health/
```

---

# 100. Workflow CI recomendado

```yaml
name: CI

on:
  pull_request:
    branches:
      - main

permissions:
  contents: read

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:

  quality:
    runs-on:
      - self-hosted
      - linux
      - ci

    timeout-minutes: 20

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - name: Install
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Unit
        run: npm run test:unit

      - name: Build
        run: npm run build
```

Adapte os scripts ao `package.json` real do projeto.

---

# 101. Job de integração

Exemplo conceitual:

```yaml
integration:
  runs-on:
    - self-hosted
    - linux
    - docker

  steps:
    - uses: actions/checkout@v4

    - name: Subir dependências
      run: docker compose -f docker-compose.test.yml up -d

    - name: Testar
      run: npm run test:integration

    - name: Limpar
      if: always()
      run: docker compose -f docker-compose.test.yml down -v
```

---

# 102. Job E2E

```yaml
e2e:
  runs-on:
    - self-hosted
    - linux
    - e2e

  steps:
    - uses: actions/checkout@v4

    - uses: actions/setup-node@v4
      with:
        node-version: 22
        cache: npm

    - run: npm ci

    - run: npx playwright install --with-deps

    - run: npm run test:e2e:smoke
```

Depois otimizaremos cache e imagens de runner.

---

# 103. Dependências entre jobs

Podemos exigir:

```text
quality ----+
            |
integration +----> e2e
```

YAML:

```yaml
e2e:
  needs:
    - quality
    - integration
```

Assim, E2E caro só começa se verificações anteriores passarem.

---

# 104. Fail fast econômico

Ordem conceitual:

```text
lint
 |
 v
unit
 |
 v
integration
 |
 v
E2E
```

Se lint falhar em 10 segundos, não faz sentido gastar 15 minutos de E2E.

Por outro lado, paralelizar lint e unit pode reduzir latência total.

O equilíbrio depende do custo e da capacidade do runner.

---

# 105. Runner único

Com um único runner:

```text
job A
 |
job B espera
 |
job C espera
```

Muitos jobs separados não geram paralelismo real.

Nesse cenário, um job com etapas sequenciais pode ser mais eficiente para alguns fluxos.

---

# 106. Vários runners

Com:

```text
runner-ci-01
runner-ci-02
runner-e2e-01
```

podemos ter:

```text
lint ------ runner-ci-01
unit ------ runner-ci-02
e2e ------- runner-e2e-01
```

A arquitetura cresce conforme necessidade.

---

# 107. Métricas importantes

Acompanhe:

```text
tempo total da PR
tempo de fila
tempo de instalação
tempo unit
tempo integration
tempo E2E
taxa de falha
falhas intermitentes
```

Sem medir, otimização vira palpite.

---

# 108. Flaky tests

Um teste que passa e falha sem mudança relevante é `flaky`.

Impactos:

- perda de confiança;
- reruns;
- tempo desperdiçado;
- merges atrasados.

Não normalize flaky tests.

Eles devem ser investigados.

---

# 109. Rerun não é correção

Se um teste falha e a prática é apenas:

```text
Run failed jobs novamente
```

até passar, existe um problema.

O pipeline precisa ser confiável.

---

# 110. E2E e dados

E2E deve controlar os dados necessários.

Evite depender de:

```text
usuário que talvez exista
registro manual
estado deixado pelo teste anterior
```

Prefira:

```text
setup
 |
 v
teste
 |
 v
cleanup
```

---

# 111. Ambientes reproduzíveis

Uma estratégia:

```text
docker compose
 |
 +-- app
 +-- db
 +-- redis
 +-- mqtt
```

Cada execução sobe o ambiente necessário.

Isso melhora isolamento.

---

# 112. Custos e minutos

Quando se usa runner hospedado, o consumo depende do plano, tipo de repositório, sistema operacional e regras vigentes do GitHub.

Esses valores mudam com o tempo.

Por isso, consulte sempre a documentação e a página de billing atual antes de tomar decisões financeiras.

---

# 113. Migração gradual para self-hosted

Etapa 1:

```text
lint + unit
```

Etapa 2:

```text
integration
```

Etapa 3:

```text
E2E
```

Etapa 4:

```text
deploy DEV
```

Etapa 5:

```text
deploy PROD protegido
```

Não é necessário migrar todos os jobs de uma vez.

---

# 114. Workflow misto

É possível usar ambos:

```text
GitHub-hosted
      +
self-hosted
```

Exemplo:

```yaml
lint:
  runs-on: ubuntu-latest

e2e:
  runs-on:
    - self-hosted
    - e2e
```

A escolha pode considerar custo, segurança e capacidade.

---

# 115. Segurança do deploy

Runner que executa PR não confiável não deveria possuir automaticamente:

```text
SSH PROD
DB PROD
tokens administrativos
```

Separação:

```text
runner-ci
sem PROD

runner-deploy
acesso mínimo ao PROD
```

---

# 116. Secrets por environment

Uma arquitetura:

```text
development
 |
 +-- DEV_SSH_KEY

production
 |
 +-- PROD_SSH_KEY
```

Assim, secrets de produção ficam associados ao contexto de produção.

---

# 117. Aprovação humana

A aprovação deve ocorrer antes do acesso aos recursos de produção quando a configuração da plataforma permitir.

Objetivo:

```text
workflow quer PROD
       |
       v
gate
       |
       v
humano autoriza
       |
       v
job recebe condições necessárias
```

---

# 118. Auditoria

GitHub Actions registra:

- execução;
- commit;
- autor/evento;
- jobs;
- steps;
- horários;
- resultado.

Com Environments, PRs e releases, conseguimos construir rastreabilidade de deploy.

---

# 119. CI e CD separados

Sugestão:

```text
ci.yml
```

Responsável por qualidade.

```text
deploy-dev.yml
```

Responsável por DEV.

```text
deploy-prod.yml
```

Responsável por produção.

Isso reduz acoplamento e facilita permissões diferentes.

---

# 120. CI não deve alterar produção

Regra:

```text
PR CI
```

deve ser principalmente validação.

Não deve fazer deploy de produção.

---

# 121. Produção não deve recompilar sem necessidade

Arquitetura alvo:

```text
commit
 |
 v
CI
 |
 v
build
 |
 v
artifact versionado
 |
 +-- DEV
 |
 +-- PROD
```

Isso garante que PROD receba o mesmo artifact validado em DEV.

---

# 122. Artifacts versus imagens Docker

Para aplicações containerizadas:

```text
Docker image
```

pode ser o artifact implantável.

Exemplo:

```text
app:sha-a91c302
```

DEV:

```text
app:sha-a91c302
```

PROD:

```text
app:sha-a91c302
```

Sem rebuild entre ambientes.

---

# 123. Registry

Uma evolução será utilizar registry de containers.

Fluxo:

```text
CI
 |
 v
docker build
 |
 v
registry
 |
 +-- DEV pull
 |
 +-- PROD pull
```

Será detalhado em volumes posteriores.

---

# 124. Dependabot e Actions

Dependências de Actions também devem ser mantidas atualizadas.

O Dependabot pode ajudar a abrir PRs de atualização conforme configuração do repositório.

Cada PR deve passar pelo CI.

---

# 125. Workflow lint para YAML

O próprio pipeline também é código.

Erros em YAML podem impedir execução.

Podemos adicionar ferramentas de lint e validação para workflows e scripts.

---

# 126. Documentação do pipeline

Cada repositório deve explicar:

```text
quais workflows existem
quando executam
quais checks são obrigatórios
como executar testes localmente
como fazer deploy
como fazer rollback
```

O pipeline não deve ser uma caixa-preta.

---

# 127. Badge de CI

README pode mostrar status do workflow.

Isso fornece visibilidade rápida do estado da branch principal.

A URL específica depende do repositório e workflow.

---

# 128. Nomear jobs claramente

Ruim:

```yaml
jobs:
  job1:
```

Melhor:

```yaml
jobs:
  unit-tests:
```

Ou:

```yaml
name: Unit Tests
```

Checks legíveis melhoram a PR.

---

# 129. Step summary

Workflows podem produzir resumos úteis para a interface.

Exemplos:

```text
Testes: 182 passed
Coverage: 84%
E2E: 12 passed
Build: OK
```

Isso melhora a experiência de revisão.

---

# 130. Notificações

Falhas importantes podem posteriormente gerar:

- e-mail;
- Slack;
- webhook;
- sistema interno.

Mas não crie alertas excessivos.

Alerta que sempre dispara e ninguém lê perde valor.

---

# 131. GitHub Actions e IA

No fluxo assistido:

```text
IA implementa
 |
 v
push
 |
 v
Actions
 |
 v
testes objetivos
```

Isso cria uma camada independente de validação.

A IA pode dizer:

```text
"terminei"
```

mas o pipeline responde com evidência executável:

```text
PASS / FAIL
```

---

# 132. IA não deve alterar o teste para esconder falha

Um risco:

```text
implementação falha
       |
       v
agente modifica teste
       |
       v
teste passa
```

Por isso, revisão deve verificar se os testes continuam representando os requisitos.

---

# 133. Critério de aceitação → teste

Fluxo desejado:

```text
SPEC
 |
 v
Critério
 |
 v
Teste
 |
 v
Implementação
```

Exemplo:

```text
Critério:
usuário bloqueado não pode autenticar

Teste:
login de usuário bloqueado retorna 403

Implementação:
regra no serviço de autenticação
```

---

# 134. Pipeline como contrato

O CI estabelece regras objetivas.

Exemplo:

```text
Só integra se:

lint = PASS
unit = PASS
integration = PASS
build = PASS
smoke E2E = PASS
```

Isso reduz decisões improvisadas.

---

# 135. Quality Gate

Um Quality Gate é um conjunto de condições necessárias.

Exemplo:

```text
0 erros lint
unit pass
integration pass
build pass
sem vulnerabilidade crítica conhecida conforme política
```

O gate deve ser proporcional ao risco.

---

# 136. Evitar pipeline impossível

Se o pipeline demora 90 minutos em toda pequena PR, desenvolvedores tenderão a contorná-lo.

O objetivo é:

```text
rápido o suficiente para uso diário
rigoroso o suficiente para gerar confiança
```

---

# 137. Pipeline em camadas

Camada 1 — segundos/minutos:

```text
lint
typecheck
unit
```

Camada 2:

```text
integration
build
```

Camada 3:

```text
smoke E2E
```

Camada 4:

```text
full regression
security aprofundado
```

---

# 138. Estratégia de execução

```text
PR
 |
 v
Camadas 1-3
```

```text
Nightly
 |
 v
Camada 4
```

```text
Release crítica
 |
 v
Camadas adicionais
```

---

# 139. Workflow completo conceitual

```text
PR
 |
 v
CI
 |
 +--> lint
 |
 +--> unit
 |
 +--> integration
 |
 +--> build
 |
 +--> smoke E2E
 |
 v
Merge permitido
```

Depois:

```text
main
 |
 v
Deploy DEV
 |
 v
Health Check
 |
 v
Validação
 |
 v
Aprovação
 |
 v
Deploy PROD
 |
 v
Health Check
 |
 v
Monitoramento
```

---

# 140. Checklist do workflow

- [ ] Nome claro.
- [ ] Evento correto.
- [ ] Permissões mínimas.
- [ ] Runner correto.
- [ ] Timeout configurado quando pertinente.
- [ ] Checkout.
- [ ] Versões de runtimes explícitas.
- [ ] Dependências reproduzíveis.
- [ ] Testes rápidos antes dos caros.
- [ ] Cleanup com `always()` quando necessário.
- [ ] Secrets não aparecem no código.
- [ ] Artifacts de diagnóstico preservados.
- [ ] Concurrency considerada.
- [ ] Deploy separado de CI.
- [ ] Produção protegida.

---

# 141. Checklist para self-hosted

- [ ] Labels correspondem ao workflow.
- [ ] Runner está online.
- [ ] Docker funciona.
- [ ] Usuário não é root.
- [ ] Disco monitorado.
- [ ] Workspace não contém secrets persistentes.
- [ ] Containers são limpos.
- [ ] PRs não confiáveis são bloqueadas.
- [ ] Runner CI não possui acesso desnecessário a PROD.
- [ ] Processo de reconstrução está documentado.

---

# 142. Diagnóstico quando workflow não inicia

Verifique:

```text
evento ocorreu?
arquivo está na branch correta?
YAML é válido?
runner existe?
labels coincidem?
runner está online?
permissões são suficientes?
```

---

# 143. Diagnóstico quando job fica "Queued"

Em self-hosted:

```text
runner offline
runner ocupado
labels incompatíveis
grupo sem acesso
```

são causas comuns.

---

# 144. Diagnóstico quando funciona local e falha no CI

Compare:

```text
Node version
PHP version
env vars
database
filesystem
timezone
locale
network
working directory
permissions
dependency lock
```

O objetivo é eliminar diferenças implícitas.

---

# 145. Diagnóstico E2E

Colete:

```text
trace
screenshot
video
browser logs
application logs
network errors
```

Playwright oferece recursos valiosos para isso.

Não dependa apenas de uma mensagem genérica de timeout.

---

# 146. Timeout E2E

Se um teste demora demais:

não aumente imediatamente todos os timeouts.

Primeiro descubra:

```text
aplicação lenta?
seletor ruim?
race condition?
serviço indisponível?
dados incorretos?
browser sem recurso?
```

Timeout alto pode apenas esconder um problema.

---

# 147. Próximos passos práticos

Depois de instalar o runner do Volume 04:

1. criar `.github/workflows/ci.yml`;
2. apontar `runs-on` para labels do runner;
3. executar lint;
4. executar unit;
5. adicionar integração;
6. adicionar smoke E2E;
7. medir tempos;
8. configurar concurrency;
9. separar full E2E;
10. criar deploy DEV;
11. configurar gate para produção.

---

# 148. Arquitetura final dos quatro primeiros volumes

```text
VOLUME 01
Git
 |
 v
Branch + Commit
 |
 v

VOLUME 02
GitHub + PR
 |
 v

VOLUME 03
GitHub Actions
 |
 v

VOLUME 04
Self-Hosted Runner
 |
 v
Execução real
```

Em conjunto:

```text
git
 |
 v
branch
 |
 v
commit
 |
 v
push
 |
 v
PR
 |
 v
Actions
 |
 v
self-hosted runner
 |
 v
tests
 |
 v
merge
 |
 v
DEV
 |
 v
approval
 |
 v
PROD
```

---

# 149. Próximo volume

**Volume 04 — Self-Hosted Runners com Ubuntu e Docker**

Esse volume já foi preparado separadamente e detalha:

- Ubuntu Server;
- usuário dedicado;
- instalação do Docker;
- registro do runner;
- labels;
- systemd;
- Node.js;
- PHP;
- Playwright;
- Docker Compose;
- segurança;
- limpeza;
- monitoramento;
- troubleshooting.

Depois dele, o próximo aprofundamento recomendado é:

**Volume 05 — Docker no Pipeline.**

---

**Fim do Volume 03 — GitHub Actions: CI, Testes e Automação**
