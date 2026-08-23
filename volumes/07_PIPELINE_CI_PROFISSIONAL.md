# Volume 07 — Pipeline CI Profissional

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 07_PIPELINE_CI_PROFISSIONAL.md  
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

# 2. O que é CI

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

# 3. CI não é deploy

Separação:

```text
CI = validar
CD = entregar/implantar
```

Uma PR deve poder executar CI sem possuir qualquer acesso a produção.

---

# 4. Estrutura de workflows

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

# 5. Quality Gates

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

# 6. Ordem dos testes

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

# 7. Quando paralelizar

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

---

# 8. Pipeline Node.js básico

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

# 9. Scripts do package.json

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

# 10. Pipeline PHP básico

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

# 11. Composer lock

Versione:

```text
composer.lock
```

quando o projeto é uma aplicação.

CI deve instalar versões reproduzíveis.

---

# 12. npm lockfile

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

# 13. Integração com MariaDB

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

# 14. Exemplo Compose

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

# 15. Subir ambiente

```yaml
- name: Start integration stack
  run: |
    docker compose \
      -p ci-${{ github.run_id }} \
      -f compose.test.yml \
      up -d
```

---

# 16. Executar migrations

Exemplo:

```yaml
- name: Run migrations
  run: npm run db:migrate:test
```

O banco deve nascer vazio.

---

# 17. Executar integração

```yaml
- name: Integration tests
  run: npm run test:integration
```

---

# 18. Coletar logs

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

# 19. Cleanup

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

# 20. MQTT no CI

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

# 21. Redis

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

# 22. Build Docker

Depois dos testes:

```yaml
- name: Build Docker image
  run: |
    docker build \
      -t app:${{ github.sha }} \
      .
```

---

# 23. Smoke da imagem

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

# 24. Health check

```bash
curl --fail http://ENDERECO/health
```

O objetivo é testar o artifact real.

---

# 25. E2E separado

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

# 26. Dependencies

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

# 27. Required Checks

No GitHub, configure checks obrigatórios da branch principal.

Exemplo:

```text
CI / quality
CI / integration
E2E Smoke / e2e
```

Sem PASS, merge bloqueado.

---

# 28. Concurrency

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

# 29. Timeout

```yaml
timeout-minutes: 20
```

Defina por job.

Isso evita pipelines presos.

---

# 30. Retry

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

# 31. Artifacts

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

# 32. Coverage artifact

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: coverage
    path: coverage/
```

---

# 33. Playwright report

```yaml
- uses: actions/upload-artifact@v4
  if: always()
  with:
    name: playwright-report
    path: playwright-report/
```

---

# 34. Cache npm

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 22
    cache: npm
```

---

# 35. Cache Composer

A estratégia pode utilizar cache do diretório adequado, mas deve seguir a versão atual das Actions e política do projeto.

Evite caches globais sem chave apropriada.

---

# 36. Docker layer cache

Em builds frequentes, BuildKit/buildx pode reutilizar layers.

Primeiro otimize Dockerfile.

Depois adicione cache distribuído.

---

# 37. CI híbrido Node + PHP

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

# 38. Paths filters

Podemos evitar jobs quando nenhum arquivo relevante mudou.

Exemplo conceitual:

```yaml
paths:
  - 'frontend/**'
```

Mas dependências compartilhadas exigem cuidado.

---

# 39. Monorepo

Em monorepo:

```text
apps/frontend
apps/api
packages/common
```

Uma mudança em `packages/common` pode exigir ambos os pipelines.

Documente a matriz de dependências.

---

# 40. Path filtering conservador

Comece com filtros simples.

Só adote seleção avançada quando:

- pipeline já está correto;
- tempos são medidos;
- dependências são conhecidas.

---

# 41. Lint antes do E2E

Não faz sentido executar 20 minutos de browser se:

```text
syntax error
```

poderia ser detectado em segundos.

---

# 42. Typecheck

TypeScript:

```bash
tsc --noEmit
```

pode ser um gate rápido e valioso.

---

# 43. Static analysis PHP

Ferramentas como PHPStan/Psalm podem detectar problemas antes de runtime.

Podem ser adicionadas ao quality job.

---

# 44. Security baseline

CI pode incluir verificações rápidas:

```text
dependency audit
secret scan
static analysis
```

Checks pesados ficam em fluxo separado.

---

# 45. npm audit

Pode fornecer sinal de dependências vulneráveis.

Não configure bloqueio cego sem política de severidade e capacidade de tratamento.

---

# 46. Build metadata

Gere:

```json
{
  "commit": "a91c302",
  "build": "12345"
}
```

Isso pode ser incorporado à aplicação.

---

# 47. Endpoint version

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

# 48. CI summary

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

# 49. Status vermelho é útil

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

# 50. CI local

Desenvolvedor deve poder executar:

```bash
npm run lint
npm run test:unit
npm run test:integration
```

antes do push.

---

# 51. Pre-commit

Hooks locais podem executar verificações rápidas.

Não substituem o CI porque podem ser ignorados ou variar por máquina.

---

# 52. Pre-push

Pode executar:

```text
lint
unit
```

para reduzir falhas óbvias no servidor.

Evite hooks locais que demoram demais.

---

# 53. IA antes do push

Agente pode executar:

```text
lint
unit
diff review
```

antes de publicar branch.

Ainda assim o GitHub Actions repete os gates essenciais.

---

# 54. Independência da validação

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

# 55. CI para PR em Draft

Estratégia opcional:

```text
Draft -> checks rápidos
Ready -> E2E
```

Pode reduzir carga.

A lógica deve ser implementada explicitamente.

---

# 56. Cancelamento de runs antigas

Especialmente útil com agentes que fazem vários pushes durante refinamento.

```text
push 1 -> cancelado
push 2 -> cancelado
push 3 -> atual
```

---

# 57. Job queue

Métrica:

```text
quanto tempo o job esperou por runner?
```

Se fila cresce, gargalo é capacidade de runner, não teste.

---

# 58. Runner labels

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

# 59. Runner groups

Em organizações maiores, grupos controlam quais repositórios podem utilizar determinados runners.

Útil para separar:

```text
CI
deploy
produção
```

---

# 60. Segredos no CI

PR CI idealmente usa poucos secrets.

Quanto menos, menor superfície de risco.

Banco de teste:

```text
senha descartável
```

não precisa ser secret crítico.

---

# 61. Secrets de PROD

Nunca disponibilize secrets de produção em job de PR.

Separação absoluta:

```text
PR CI
       X
PROD secrets
```

---

# 62. Pull Requests externas

Código não confiável não deve executar livremente em runner interno com privilégios.

Configure política conforme origem das contribuições.

---

# 63. CI e branch protection

A branch protection transforma workflow em regra operacional:

```text
não passou
 |
 v
não integra
```

Sem proteção, pipeline pode virar apenas informativo.

---

# 64. Merge após PASS

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

# 65. Não fazer deploy dentro do CI de PR

Evite:

```text
PR -> production
```

CI deve ser seguro para experimentação.

---

# 66. Build oficial em main

Após merge:

```text
main SHA
 |
 v
build artifact oficial
```

Esse artifact será promovido.

---

# 67. Reuso de artifact

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

# 68. Pipeline modular

Evite um único YAML gigantesco.

Separe responsabilidades.

---

# 69. Reusable workflows

Projetos semelhantes podem reutilizar lógica.

Exemplo:

```text
workflow-node-ci.yml
```

centralizado.

Use versionamento para não quebrar todos os projetos simultaneamente.

---

# 70. Composite actions

Úteis para sequências repetidas:

```text
setup interno
collect logs
health check
```

---

# 71. Scripts versionados

Exemplo:

```text
scripts/ci/
  lint.sh
  integration.sh
  collect-logs.sh
```

Testáveis localmente.

---

# 72. ShellCheck

Scripts shell podem ser analisados por ShellCheck.

É uma ferramenta open source muito útil para pipelines Linux.

---

# 73. YAML lint

Valide YAML para evitar erros simples.

Pode ser executado em PRs que alteram workflows.

---

# 74. actionlint

`actionlint` é uma ferramenta open source especializada em GitHub Actions workflows.

Pode ser incorporada ao CI.

---

# 75. Pipeline self-test

O próprio pipeline precisa de testes.

Mudanças em `.github/workflows` devem receber atenção especial.

---

# 76. Falha de infraestrutura versus código

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

# 77. Retry de infraestrutura

Um retry automático pode ser aceitável para falhas claramente transitórias de infraestrutura.

Não para assertion funcional.

---

# 78. Métricas do CI

Acompanhe:

- duração média;
- p95;
- fila;
- taxa de sucesso;
- flaky rate;
- tempo de instalação;
- tempo de E2E.

---

# 79. SLO de CI

Pode existir meta interna:

```text
90% das PRs recebem resultado rápido em até X minutos
```

O valor depende da equipe/projeto.

---

# 80. Otimização orientada por métricas

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

# 81. Exemplo de pipeline completo

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

# 82. Evolução futura

Quando crescer:

```text
quality em paralelo
integration shards
E2E shards
test impact analysis
runners efêmeros
```

---

# 83. Checklist CI profissional

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
- [ ] Artifacts úteis.
- [ ] Secrets mínimos.
- [ ] Sem acesso PROD.
- [ ] Métricas acompanhadas.

---

# 84. Próximo volume

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

**Fim do Volume 07 — Pipeline CI Profissional**
