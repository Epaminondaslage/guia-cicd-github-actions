# Volume 07 — Pipeline CI Pessoal

**Projeto:** Guia Pessoal de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 07_PIPELINE_CI_PESSOAL.md  
**Versão:** 0.1.0  
**Pré-requisitos:** Volumes 01 a 06

---

## 1. Objetivo

Este volume transforma os conceitos anteriores em um pipeline de integração contínua completo e operacional.

Arquitetura de referência:

```text
Pull Request
    |
    v
GitHub Actions
    |
    +--> lint
    +--> typecheck
    +--> unit
    +--> integration
    +--> build
    +--> smoke E2E
    |
    v
Quality Gate
    |
    +--> FAIL -> bloqueia merge
    |
    +--> PASS -> PR apta
```

---

## 2. O que é CI

Continuous Integration significa integrar alterações frequentemente e validar automaticamente se elas permanecem compatíveis com o sistema.

O CI deve responder rapidamente:

```text
este código:
compila?
segue padrões?
passa testes?
quebra integração?
gera artifact válido?
```

---

## 3. CI não é deploy

Separação:

```text
CI = validar
CD = entregar/implantar
```

Uma PR deve poder executar CI sem possuir qualquer acesso a produção.

---

## 4. Estrutura de workflows

Sugestão:

```text
.github/
└── workflows/
    ├── ci.yml
    ├── e2e-smoke.yml
    ├── e2e-nightly.yml
    ├── deploy-dev.yml
    └── deploy-prod.yml
```

Este volume concentra-se em `ci.yml`.

---

## 5. Quality gates

Checks iniciais:

```text
lint
unit
integration
build
smoke E2E
```

Em projetos TypeScript:

```text
typecheck
```

Em PHP:

```text
static analysis
phpunit
```

---

## 6. Ordem dos testes

Uma sequência econômica:

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
build
 |
 v
E2E
```

Se lint falhar, o pipeline evita gastar recursos em E2E.

---

## 7. Quando paralelizar

Com vários runners:

```text
lint --------+
unit --------+--> build --> E2E
integration -+
```

Com um único runner:

```text
lint -> unit -> integration -> build -> E2E
```

A arquitetura deve refletir a capacidade real.

### 7.1 Matrix strategy

Quando o mesmo job precisa rodar contra várias versões, sistemas operacionais ou shards, use `strategy.matrix` em vez de duplicar jobs manualmente:

```yaml
jobs:
  unit:
    runs-on: [self-hosted, linux, ci]
    strategy:
      fail-fast: false
      max-parallel: 4
      matrix:
        node-version: [20, 22]
        shard: [1, 2, 3, 4]

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: npm

      - run: npm ci
      - run: npm run test:unit -- --shard=${{ matrix.shard }}/4
```

Pontos importantes:

| Opção | Efeito |
|---|---|
| `fail-fast: true` (padrão) | Uma falha cancela todas as outras combinações |
| `fail-fast: false` | Cada combinação roda até o fim, útil para ver o panorama completo |
| `max-parallel` | Limita quantos jobs da matrix rodam ao mesmo tempo (proteção de runners self-hosted) |

Em runners self-hosted com capacidade limitada, `max-parallel` evita que uma matrix grande esgote todos os runners disponíveis e trave outros workflows.

---

## 8. Pipeline Node.js básico

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
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - run: npm ci
      - run: npm run lint
      - run: npm run test:unit
      - run: npm run build
```

---

## 9. Scripts do package.json

Evite lógica específica demais no YAML.

Prefira:

```json
{
  "scripts": {
    "lint": "eslint .",
    "test:unit": "vitest run",
    "test:integration": "vitest run tests/integration",
    "test:e2e:smoke": "playwright test --grep @smoke",
    "build": "npm run build:app"
  }
}
```

Assim os mesmos comandos funcionam localmente.

---

## 10. Pipeline PHP básico

```yaml
name: CI PHP

on:
  pull_request:

jobs:
  php-tests:
    runs-on:
      - self-hosted
      - linux
      - php

    steps:
      - uses: actions/checkout@v4

      - name: Composer install
        run: composer install --no-interaction --prefer-dist

      - name: PHPUnit
        run: vendor/bin/phpunit
```

---

## 11. Composer lock

Versione:

```text
composer.lock
```

quando o projeto é uma aplicação.

CI deve instalar versões reproduzíveis.

---

## 12. npm lockfile

Versione:

```text
package-lock.json
```

e use:

```bash
npm ci
```

no pipeline.

---

## 13. Integração com MariaDB

```text
CI
 |
 v
Compose
 |
 +-- app
 +-- MariaDB
```

Arquivo:

```text
compose.test.yml
```

---

## 14. Exemplo Compose

```yaml
services:
  db:
    image: mariadb:11
    environment:
      MARIADB_ROOT_PASSWORD: test
      MARIADB_DATABASE: app_test
      MARIADB_USER: app
      MARIADB_PASSWORD: test
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 3s
      timeout: 3s
      retries: 30
```

---

## 15. Subir ambiente

```yaml
- name: Start integration stack
  run: |
    docker compose \
      -p ci-${{ github.run_id }} \
      -f compose.test.yml \
      up -d
```

---

## 16. Executar migrations

Exemplo:

```yaml
- name: Run migrations
  run: npm run db:migrate:test
```

O banco deve nascer vazio.

---

## 17. Executar integração

```yaml
- name: Integration tests
  run: npm run test:integration
```

---

## 18. Coletar logs

```yaml
- name: Collect logs
  if: failure()
  run: |
    docker compose \
      -p ci-${{ github.run_id }} \
      -f compose.test.yml \
      logs --no-color
```

---

## 19. Cleanup

```yaml
- name: Cleanup
  if: always()
  run: |
    docker compose \
      -p ci-${{ github.run_id }} \
      -f compose.test.yml \
      down -v
```

---

## 20. MQTT no CI

Adicionar:

```yaml
mqtt:
  image: eclipse-mosquitto:2
```

Para testes, forneça configuração específica e sem credenciais reais.

Fluxo:

```text
teste publica
 |
 v
aplicação processa
 |
 v
aplicação publica retorno
 |
 v
assertion
```

---

## 21. Redis

```yaml
redis:
  image: redis:alpine
```

Útil para:

- cache;
- filas;
- sessões;
- locks.

---

## 22. Build Docker

Depois dos testes:

```yaml
- name: Build Docker image
  run: |
    docker build \
      -t app:${{ github.sha }} \
      .
```

---

## 23. Smoke da imagem

```yaml
- name: Start image
  run: |
    docker run -d \
      --name app-${{ github.run_id }} \
      -p 127.0.0.1::3000 \
      app:${{ github.sha }}
```

A publicação dinâmica de porta exige descobrir a porta atribuída ou usar networking adequado.

---

## 24. Health check

```bash
curl --fail http://ENDERECO/health
```

O objetivo é testar o artifact real.

---

## 25. E2E separado

`e2e-smoke.yml`:

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
```

Separar E2E facilita:

- runner dedicado;
- timeout próprio;
- artifacts próprios.

---

## 26. Dependencies

Podemos executar E2E somente se CI básico passou.

Uma alternativa é manter tudo no mesmo workflow com `needs`.

Exemplo:

```yaml
e2e:
  needs:
    - quality
    - integration
```

---

## 27. Required checks

No GitHub, configure checks obrigatórios da branch principal.

Exemplo:

```text
CI / quality
CI / integration
E2E Smoke / e2e
```

Sem PASS, merge bloqueado.

---

## 28. Concurrency

```yaml
concurrency:
  group: ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

Objetivo:

```text
commit novo
 |
 v
cancelar execução antiga da mesma branch
```

---

## 29. Timeout

```yaml
timeout-minutes: 20
```

Defina por job.

Isso evita pipelines presos.

---

## 30. Retry

Retry deve ser usado com disciplina.

Não transforme:

```text
flaky test
```

em:

```text
pipeline "verde" por insistência
```

---

## 31. Artifacts

Resultados úteis:

```text
coverage
playwright-report
screenshots
logs
build metadata
```

Uploads devem ocorrer especialmente em falha.

---

## 32. Coverage artifact

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: coverage
    path: coverage/
```

---

## 33. Playwright report

```yaml
- uses: actions/upload-artifact@v4
  if: always()
  with:
    name: playwright-report
    path: playwright-report/
```

**Atenção com `actions/upload-artifact@v4` e `actions/download-artifact@v4`:** a partir da v4, cada `name` de artifact precisa ser único dentro do run: não é mais possível fazer múltiplos uploads acumulando no mesmo nome (comportamento da v3). Em uma matrix, gere nomes únicos por combinação:

```yaml
- uses: actions/upload-artifact@v4
  if: always()
  with:
    name: playwright-report-${{ matrix.shard }}
    path: playwright-report/
    retention-days: 7
```

### 33.1 Passando artifacts entre jobs

Artifacts são o mecanismo padrão para levar arquivos de um job para outro (jobs não compartilham filesystem). Fluxo típico: `build` gera o artifact, `deploy` ou `e2e` baixa e usa:

```yaml
jobs:
  build:
    runs-on: [self-hosted, linux, ci]
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build

      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/
          retention-days: 7

  e2e:
    needs: build
    runs-on: [self-hosted, linux, e2e]
    steps:
      - uses: actions/checkout@v4

      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/

      - run: npm run test:e2e:smoke
```

Para baixar todos os artifacts de um run de uma vez (útil em jobs de consolidação), omita `name`:

```yaml
- uses: actions/download-artifact@v4
  with:
    path: all-artifacts/
```

### 33.2 Outputs entre jobs

Para valores pequenos (strings, flags, versões, SHAs), e não arquivos, use `needs.<job>.outputs` em vez de artifacts. O job produtor declara `outputs:` e escreve em `$GITHUB_OUTPUT`; nunca use o comando depreciado `set-output`, removido pelo GitHub em 2023 por questões de segurança:

```yaml
jobs:
  prepare:
    runs-on: [self-hosted, linux, ci]
    outputs:
      version: ${{ steps.vars.outputs.version }}
    steps:
      - id: vars
        run: echo "version=$(git rev-parse --short HEAD)" >> "$GITHUB_OUTPUT"

  build:
    needs: prepare
    runs-on: [self-hosted, linux, ci]
    steps:
      - run: echo "Construindo versão ${{ needs.prepare.outputs.version }}"
```

O mesmo vale para variáveis de ambiente dentro de um step: use `$GITHUB_ENV`, nunca `echo "::set-env::"` (também depreciado e removido):

```yaml
- run: echo "BUILD_ID=${{ github.run_id }}" >> "$GITHUB_ENV"
```

---

## 34. Cache de dependências

Cache reduz o tempo gasto reinstalando dependências que não mudaram. Existem duas abordagens:

```text
cache nativo das setup-actions  -> mais simples, cobre o caso comum
actions/cache                   -> controle total de chave, paths e política de restore
```

### 34.1 Cache nativo em setup-node

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 22
    cache: npm
```

O `cache: npm` usa o hash do `package-lock.json` como parte da chave automaticamente. Também funciona com `yarn` e `pnpm` (nesse caso, informe `cache-dependency-path` se o lockfile não estiver na raiz).

### 34.2 Cache nativo em setup-python

```yaml
- uses: actions/setup-python@v5
  with:
    python-version: "3.12"
    cache: pip
    cache-dependency-path: requirements.txt
```

### 34.3 Cache nativo em setup-java

```yaml
- uses: actions/setup-java@v4
  with:
    distribution: temurin
    java-version: "21"
    cache: maven
```

Suporta `maven`, `gradle` e `sbt`.

### 34.4 actions/cache para o caso genérico

Quando não existe cache nativo (ex.: cache de build do Composer, cache de ferramentas customizadas), use `actions/cache@v4` diretamente. A chave deve incluir algo que mude quando as dependências mudam (normalmente o hash do lockfile) e um `restore-keys` como fallback parcial:

```yaml
- uses: actions/cache@v4
  with:
    path: |
      ~/.composer/cache
    key: composer-${{ runner.os }}-${{ hashFiles('composer.lock') }}
    restore-keys: |
      composer-${{ runner.os }}-
```

Regras de ouro para a chave:

| Regra | Motivo |
|---|---|
| Incluir `runner.os` | Caches de SOs diferentes não são compatíveis |
| Incluir hash do lockfile | Cache muda quando as dependências mudam |
| `restore-keys` em cascata | Permite reaproveitar cache parcial em vez de começar do zero |
| Nunca usar chave fixa | "Cache sempre igual" nunca invalida e passa a mentir |

Em runners self-hosted, avalie se o cache do GitHub Actions (limitado por repositório) compensa frente a um cache local persistente no próprio runner: muitas vezes o segundo é mais rápido e não conta contra a cota do GitHub.

---

## 35. Cache Composer

Composer não tem uma setup-action oficial mantida pelo GitHub com cache nativo equivalente ao `setup-node`. A prática recomendada é combinar `shivammathur/setup-php` (ou instalar o Composer disponível no runner) com `actions/cache@v4`, como mostrado em 34.4, usando `composer.lock` na chave.

Evite caches globais sem chave apropriada: um cache sem hash do lockfile na chave serve dependências desatualizadas silenciosamente.

---

## 36. Docker layer cache

Em builds frequentes, BuildKit/buildx pode reutilizar layers.

Primeiro otimize Dockerfile (ordem de camadas, `.dockerignore`, dependências antes do código-fonte).

Depois adicione cache distribuído, por exemplo com `docker/build-push-action@v6` e `cache-from`/`cache-to` apontando para um registry (`type=registry`) ou para o cache do GitHub Actions (`type=gha`):

```yaml
- uses: docker/setup-buildx-action@v3

- uses: docker/build-push-action@v6
  with:
    context: .
    tags: app:${{ github.sha }}
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

Em runners self-hosted persistentes, o próprio cache local do Docker (`docker build` reutilizando layers já baixadas) já entrega boa parte do ganho sem configuração extra. Meça antes de adicionar cache remoto.

---

## 37. CI híbrido Node + PHP

Arquitetura:

```text
PR
 |
 +-- frontend-ci
 +-- node-api-ci
 +-- php-ci
 +-- integration
```

Cada stack possui job próprio.

---

## 38. Paths filters

Podemos evitar jobs quando nenhum arquivo relevante mudou.

Exemplo conceitual:

```yaml
paths:
  - 'frontend/**'
```

Mas dependências compartilhadas exigem cuidado.

---

## 39. Monorepo

Em monorepo:

```text
apps/frontend
apps/api
packages/common
```

Uma mudança em `packages/common` pode exigir ambos os pipelines.

Documente a matriz de dependências.

---

## 40. Path filtering conservador

Comece com filtros simples.

Só adote seleção avançada quando:

- pipeline já está correto;
- tempos são medidos;
- dependências são conhecidas.

---

## 41. Lint antes do E2E

Não faz sentido executar 20 minutos de browser se:

```text
syntax error
```

poderia ser detectado em segundos.

---

## 42. Typecheck

TypeScript:

```bash
tsc --noEmit
```

pode ser um gate rápido e valioso.

---

## 43. Static analysis PHP

Ferramentas como PHPStan/Psalm podem detectar problemas antes de runtime.

Podem ser adicionadas ao quality job.

---

## 44. Security baseline

CI pode incluir verificações rápidas:

```text
dependency audit
secret scan
static analysis
```

Checks pesados ficam em fluxo separado.

---

## 45. npm audit

Pode fornecer sinal de dependências vulneráveis.

Não configure bloqueio cego sem política de severidade e capacidade de tratamento.

---

## 46. Build metadata

Gere:

```json
{
  "commit": "a91c302",
  "build": "12345"
}
```

Isso pode ser incorporado à aplicação.

---

## 47. Endpoint version

Exemplo:

```text
/version
```

retorna:

```json
{
  "version": "a91c302"
}
```

Ajuda a confirmar qual artifact está rodando.

---

## 48. CI summary

Ao final:

```text
Lint: PASS
Unit: 184 PASS
Integration: 32 PASS
Build: PASS
E2E Smoke: 12 PASS
Commit: a91c302
```

Um summary bem organizado reduz tempo de diagnóstico.

---

## 49. Status vermelho é útil

Pipeline falho não é problema por si só.

Ele está cumprindo seu papel se detectou algo antes do merge.

O problema é:

```text
falha incompreensível
```

ou:

```text
falso positivo recorrente
```

---

## 50. CI local

Desenvolvedor deve poder executar:

```bash
npm run lint
npm run test:unit
npm run test:integration
```

antes do push.

---

## 51. Pre-commit

Hooks locais podem executar verificações rápidas.

Não substituem o CI porque podem ser ignorados ou variar por máquina.

---

## 52. Pre-push

Pode executar:

```text
lint
unit
```

para reduzir falhas óbvias no servidor.

Evite hooks locais que demoram demais.

---

## 53. IA antes do push

Agente pode executar:

```text
lint
unit
diff review
```

antes de publicar branch.

Ainda assim o GitHub Actions repete os gates essenciais.

---

## 54. Independência da validação

Fluxo:

```text
IA implementa
 |
 v
testa local
 |
 v
push
 |
 v
CI independente
```

Isso evita confiar apenas no relato do agente.

---

## 55. CI para PR em draft

Estratégia opcional:

```text
Draft -> checks rápidos
Ready -> E2E
```

Pode reduzir carga.

A lógica deve ser implementada explicitamente.

---

## 56. Cancelamento de runs antigas

Especialmente útil com agentes que fazem vários pushes durante refinamento.

```text
push 1 -> cancelado
push 2 -> cancelado
push 3 -> atual
```

---

## 57. Job queue

Métrica:

```text
quanto tempo o job esperou por runner?
```

Se fila cresce, gargalo é capacidade de runner, não teste.

---

## 58. Runner labels

Exemplo:

```text
ci
docker
e2e
php
node
```

Não use dezenas de labels desnecessárias.

---

## 59. Runner groups

Em organizações maiores, grupos controlam quais repositórios podem utilizar determinados runners.

Útil para separar:

```text
CI
deploy
produção
```

---

## 60. Segredos no CI

PR CI idealmente usa poucos secrets.

Quanto menos, menor superfície de risco.

Banco de teste:

```text
senha descartável
```

não precisa ser secret crítico.

---

## 61. Secrets de PROD

Nunca disponibilize secrets de produção em job de PR.

Separação absoluta:

```text
PR CI
       X
PROD secrets
```

---

## 62. Pull requests externas

Código não confiável não deve executar livremente em runner interno com privilégios.

Configure política conforme origem das contribuições.

---

## 63. CI e branch protection

A branch protection transforma workflow em regra operacional:

```text
não passou
 |
 v
não integra
```

Sem proteção, pipeline pode virar apenas informativo.

---

## 64. Merge após PASS

Fluxo:

```text
PR
 |
 checks
 |
 review
 |
 PASS
 |
 merge
```

Depois do merge, entra o CD.

---

## 65. Não fazer deploy dentro do CI de PR

Evite:

```text
PR -> production
```

CI deve ser seguro para experimentação.

---

## 66. Build oficial em main

Após merge:

```text
main SHA
 |
 v
build artifact oficial
```

Esse artifact será promovido.

---

## 67. Reuso de artifact

Ideal:

```text
build
 |
 v
artifact
 |
 +-- DEV
 |
 +-- PROD
```

Não reconstruir em PROD.

---

## 68. Pipeline modular

Evite um único YAML gigantesco.

Separe responsabilidades.

---

## 69. Reusable workflows

Projetos semelhantes podem reutilizar lógica através de um workflow chamável (`workflow_call`), em vez de copiar YAML entre repositórios.

Workflow reutilizável (`.github/workflows/node-ci.yml` em um repo central ou no próprio repo):

```yaml
name: Node CI reutilizável

on:
  workflow_call:
    inputs:
      node-version:
        type: string
        default: "22"
      working-directory:
        type: string
        default: "."
    secrets:
      NPM_TOKEN:
        required: false

jobs:
  quality:
    runs-on: [self-hosted, linux, ci]
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: npm
          cache-dependency-path: ${{ inputs.working-directory }}/package-lock.json

      - run: npm ci
      - run: npm run lint
      - run: npm run test:unit
      - run: npm run build
```

Workflow chamador, em outro repositório ou em outro arquivo do mesmo repositório:

```yaml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: org/repo-central/.github/workflows/node-ci.yml@v1
    with:
      node-version: "22"
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

Pontos importantes:

| Item | Descrição |
|---|---|
| `uses: org/repo@ref` | Sempre referencie por tag/branch/SHA, nunca sem versão |
| Inputs/secrets explícitos | Só o que é declarado em `workflow_call` fica visível ao chamador |
| `secrets: inherit` | Repassa todos os secrets do chamador (use com cautela) |

Versione o workflow central (`@v1`, `@v1.2.0` ou SHA fixo) para não quebrar todos os consumidores ao alterar a lógica interna. Trate o workflow reutilizável como uma dependência com contrato: mudanças breaking exigem uma nova major tag.

---

## 70. Composite actions vs reusable workflows

Ambos evitam duplicação, mas resolvem problemas diferentes:

| Mecanismo | O que reutiliza |
|---|---|
| Composite action | Uma sequência de STEPS dentro de um job existente |
| Reusable workflow | Um ou mais JOBS inteiros, com seu próprio `runs-on`, matrix e `needs` |

Composite action, útil para sequências repetidas como:

```text
setup interno
collect logs
health check
```

Exemplo (`.github/actions/collect-logs/action.yml`):

```yaml
name: Collect logs
description: Coleta logs do compose de teste em caso de falha

inputs:
  project-name:
    required: true

runs:
  using: composite
  steps:
    - shell: bash
      run: |
        docker compose \
          -p ${{ inputs.project-name }} \
          -f compose.test.yml \
          logs --no-color
```

Uso no workflow:

```yaml
- uses: ./.github/actions/collect-logs
  if: failure()
  with:
    project-name: ci-${{ github.run_id }}
```

Regra prática para escolher:

| Necessidade | Escolha |
|---|---|
| Vários jobs, matrix própria ou `runs-on` diferente | Reusable workflow |
| Só encapsular alguns steps dentro de um job | Composite action |

---

## 71. Scripts versionados

Exemplo:

```text
scripts/ci/
  lint.sh
  integration.sh
  collect-logs.sh
```

Testáveis localmente.

---

## 72. ShellCheck

Scripts shell podem ser analisados por ShellCheck.

É uma ferramenta open source muito útil para pipelines Linux.

---

## 73. YAML lint

Valide YAML para evitar erros simples.

Pode ser executado em PRs que alteram workflows.

---

## 74. actionlint

`actionlint` é uma ferramenta open source especializada em GitHub Actions workflows.

Pode ser incorporada ao CI.

---

## 75. Pipeline self-test

O próprio pipeline precisa de testes.

Mudanças em `.github/workflows` devem receber atenção especial.

---

## 76. Falha de infraestrutura versus código

Classifique:

```text
application failure
infrastructure failure
flaky test
runner failure
external dependency
```

Isso melhora diagnóstico.

---

## 77. Retry de infraestrutura

Um retry automático pode ser aceitável para falhas claramente transitórias de infraestrutura.

Não para assertion funcional.

---

## 78. Métricas do CI

Acompanhe:

- duração média;
- p95;
- fila;
- taxa de sucesso;
- flaky rate;
- tempo de instalação;
- tempo de E2E.

---

## 79. SLO de CI

Pode existir meta interna:

```text
90% das PRs recebem resultado rápido em até X minutos
```

O valor depende da equipe/projeto.

---

## 80. Otimização orientada por métricas

Se 60% do tempo está em:

```text
npm ci
```

otimize cache.

Se está em:

```text
E2E
```

otimize estratégia E2E.

---

## 81. Exemplo de pipeline completo

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
    runs-on: [self-hosted, linux, ci]
    timeout-minutes: 15

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - run: npm ci
      - run: npm run lint
      - run: npm run test:unit
      - run: npm run build

  integration:
    needs: quality
    runs-on: [self-hosted, linux, docker]
    timeout-minutes: 20

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - run: npm ci

      - name: Start stack
        run: |
          docker compose \
            -p ci-${{ github.run_id }} \
            -f compose.test.yml \
            up -d

      - run: npm run test:integration

      - name: Logs
        if: failure()
        run: |
          docker compose \
            -p ci-${{ github.run_id }} \
            -f compose.test.yml \
            logs --no-color

      - name: Cleanup
        if: always()
        run: |
          docker compose \
            -p ci-${{ github.run_id }} \
            -f compose.test.yml \
            down -v

  e2e:
    needs: integration
    runs-on: [self-hosted, linux, e2e]
    timeout-minutes: 30

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

---

## 82. Evolução futura

Quando crescer:

```text
quality em paralelo
integration shards
E2E shards
test impact analysis
runners efêmeros
```

---

## 83. Checklist CI pessoal

- [ ] PR dispara CI.
- [ ] `main` está protegida.
- [ ] Checks obrigatórios.
- [ ] Concurrency configurada.
- [ ] Timeouts.
- [ ] Scripts reproduzíveis.
- [ ] Testes baratos primeiro.
- [ ] Banco/MQTT isolados.
- [ ] Logs em falha.
- [ ] Cleanup.
- [ ] Artifacts úteis, com nomes únicos por matrix.
- [ ] Cache de dependências configurado com chave baseada no lockfile.
- [ ] Outputs entre jobs via `GITHUB_OUTPUT`/`GITHUB_ENV` (nunca `set-output`/`set-env`).
- [ ] Reusable workflows/composite actions versionados por tag.
- [ ] Secrets mínimos.
- [ ] Sem acesso PROD.
- [ ] Métricas acompanhadas.

---

## 84. Próximo volume

**Volume 08 — Desenvolvimento Orientado por Especificação e IA**

Abordará:

- entrevista de requisitos;
- SPEC;
- plano de implementação;
- agentes;
- revisão;
- branches;
- PRs;
- testes;
- refinamentos;
- rastreabilidade;
- prevenção de mudanças fora de escopo.

---

## Fontes

### Matrix e paralelismo

- [Running variations of jobs in a workflow](https://docs.github.com/en/actions/using-jobs/using-a-build-matrix-for-your-jobs) — comprova a sintaxe de `strategy.matrix`, `fail-fast` e `max-parallel` usada na seção 7.1.

### Cache

- [Dependency caching reference](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows) — referência oficial de cache de dependências citada na seção 34.
- [actions/cache](https://github.com/actions/cache) — repositório oficial da action usada em 34.4 e 35 para cache genérico com chave baseada em hash de lockfile e `restore-keys`.

### Artifacts e outputs entre jobs

- [actions/upload-artifact](https://github.com/actions/upload-artifact) — confirma a mudança de comportamento na v4: nomes de artifact precisam ser únicos por run, base da observação em 33.
- [actions/download-artifact](https://github.com/actions/download-artifact) — confirma que omitir `name` baixa todos os artifacts do run, usado em 33.1.
- [Defining outputs for jobs](https://docs.github.com/en/actions/using-jobs/defining-outputs-for-jobs) — comprova a sintaxe `outputs:` do job e `needs.<job>.outputs` usada em 33.2.
- [Workflow commands for GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions) — comprova o uso de `GITHUB_OUTPUT`/`GITHUB_ENV` como substitutos dos comandos depreciados `set-output`/`set-env` citados em 33.2.

### Reusable workflows e composite actions

- [Reusing workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows) — comprova a sintaxe `workflow_call`, inputs/secrets e `secrets: inherit` da seção 69.
- [Creating a composite action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action) — comprova a estrutura `runs: using: composite` usada no exemplo da seção 70.

### Docker build cache

- [GitHub Actions cache for Docker builds](https://docs.docker.com/build/ci/github-actions/cache/) — comprova `cache-from`/`cache-to` com `type=gha` e `type=registry` usados na seção 36.

### Lint de workflows/scripts

- [actionlint](https://github.com/rhysd/actionlint) — repositório oficial da ferramenta citada na seção 74.
- [ShellCheck](https://www.shellcheck.net/) — site oficial da ferramenta citada na seção 72.

---

**Fim do Volume 07 — Pipeline CI Pessoal**
