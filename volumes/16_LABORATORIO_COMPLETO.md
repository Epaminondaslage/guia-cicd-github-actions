# Volume 16 — Laboratório Completo: do Repositório à Produção

**Projeto:** Guia Pessoal de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 16_LABORATORIO_COMPLETO.md  
**Versão:** 0.1.0  
**Objetivo:** consolidar todo o guia em um laboratório executável.

---

## 1. Cenário do laboratório

Aplicação:

```text
Node.js API
MariaDB
Frontend
Playwright
Docker
GitHub Actions
Self-hosted runner
```

Ambientes:

```text
LOCAL
DEV
PROD
```

Fluxo:

```text
SPEC
 |
 v
Branch
 |
 v
PR
 |
 v
CI
 |
 v
Merge
 |
 v
Build image
 |
 v
DEV
 |
 v
Approval
 |
 v
PROD
```

---

## 2. Infraestrutura mínima

Runner:

```text
Ubuntu Server LTS
4 vCPU
8 GB RAM
SSD
Docker
GitHub Actions Runner
```

DEV e PROD podem ser VMs separadas.

---

## 3. Repositório

Estrutura:

```text
.
├── .github/
│   └── workflows/
├── docs/
│   ├── specs/
│   ├── plans/
│   └── adr/
├── src/
├── tests/
├── scripts/
│   ├── ci/
│   └── deploy/
├── Dockerfile
├── compose.test.yml
├── package.json
└── README.md
```

---

## 4. Criar SPEC

```text
docs/specs/SPEC-001-health.md
```

Critério:

```text
GET /health retorna 200 e status ok
```

---

## 5. Criar branch

```bash
git switch main
git pull
git switch -c feature/001-health
```

---

## 6. Implementar endpoint

Exemplo:

```text
GET /health
```

Resposta:

```json
{
  "status": "ok"
}
```

---

## 7. Unit test

Adicione teste da lógica quando aplicável.

---

## 8. API test

Valide:

```text
GET /health -> 200
```

---

## 9. Commit

```bash
git add .
git commit -m "feat: adiciona endpoint de health"
```

---

## 10. Push

```bash
git push -u origin feature/001-health
```

---

## 11. Abrir PR

Descrição:

```text
SPEC-001
Objetivo
Como testar
Critérios
```

---

## 12. CI workflow

Arquivo:

```text
.github/workflows/ci.yml
```

---

## 13. CI mínimo

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

## 14. Registrar runner

No GitHub:

```text
Settings
Actions
Runners
New self-hosted runner
```

O GitHub gera um token de registro (`--token`) válido por tempo limitado e uma URL específicas para o seu repositório/organização. Copie os comandos exatamente como exibidos na tela, pois o token muda a cada geração. O roteiro abaixo é o padrão do runner oficial (`actions/runner`), executado como usuário dedicado (nunca root):

```bash
mkdir actions-runner && cd actions-runner

curl -o actions-runner-linux-x64.tar.gz -L \
  https://github.com/actions/runner/releases/latest/download/actions-runner-linux-x64-2.319.1.tar.gz

tar xzf ./actions-runner-linux-x64.tar.gz

./config.sh --url https://github.com/ORG/REPO --token SEU_TOKEN_AQUI
```

Ajuste a versão do pacote para a mais recente publicada em `actions/runner/releases` no momento da instalação.

Para transformar o runner em serviço persistente (sobrevive a reboot, reinicia sozinho):

```bash
sudo ./svc.sh install
sudo ./svc.sh start
sudo ./svc.sh status
```

---

## 15. Labels

Adicionar:

```text
ci
docker
```

---

## 16. Executar PR

Confirme:

```text
CI PASS
```

---

## 17. Branch protection

Configure `main` para exigir:

```text
PR
CI PASS
```

---

## 18. MariaDB de integração

`compose.test.yml`:

```yaml
services:
  db:
    image: mariadb:11
    environment:
      MARIADB_ROOT_PASSWORD: test
      MARIADB_DATABASE: app_test
```

---

## 19. Integration job

Subir Compose, aplicar migrations e testar.

---

## 20. Cleanup

Sempre:

```bash
docker compose down -v
```

com `if: always()` no workflow.

---

## 21. MQTT opcional

Adicionar Mosquitto caso aplicação utilize eventos.

---

## 22. E2E smoke

Criar:

```text
tests/e2e/smoke.spec.ts
```

Fluxo:

```text
abrir app
confirmar dashboard
```

---

## 23. Playwright workflow

Use runner com label:

```text
e2e
```

ou mesmo runner inicial.

---

## 24. Artifacts

Em falha, publicar com `actions/upload-artifact@v4`:

```yaml
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: |
            playwright-report/
            test-results/
          retention-days: 7
```

Conteúdo típico:

```text
playwright-report
trace
logs
```

---

## 25. Merge

Após checks:

```text
Squash and merge
```

---

## 26. Build oficial

Workflow em `main` (`.github/workflows/build.yml`), disparado após o merge:

```yaml
name: Build

on:
  push:
    branches:
      - main

permissions:
  contents: read
  packages: write

jobs:
  build:
    runs-on: [self-hosted, linux, docker]

    steps:
      - uses: actions/checkout@v4

      - name: Log in to registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:sha-${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

Equivalente manual, para rodar localmente ou entender o que o workflow faz:

```bash
docker buildx build \
  --tag ghcr.io/org/app:sha-a91c302 \
  --push \
  .
```

---

## 27. Registry

Utilizar registry escolhido (ex.: GitHub Container Registry, `ghcr.io`).

Artifact:

```text
ghcr.io/org/app:sha-a91c302
```

---

## 28. DEV

Servidor DEV possui:

```text
Docker
Compose
config
```

---

## 29. Deploy DEV

```bash
docker compose pull app
docker compose up -d app
```

---

## 30. Health DEV

```bash
curl --fail https://dev.example/health
```

---

## 31. E2E DEV

Executar smoke contra DEV.

---

## 32. Validação visual

Abrir frontend manualmente.

Verificar:

- desktop;
- mobile;
- fluxo principal.

---

## 33. Nova SPEC se necessário

Se frontend não agradar:

```text
SPEC-002
```

referencia PR anterior.

Nova branch e nova PR.

---

## 34. Production Environment

Criar no GitHub:

```text
production
```

Adicionar regras de proteção disponíveis.

---

## 35. Secrets PROD

Configurar:

```text
PROD_HOST
PROD_SSH_KEY
```

ou mecanismo equivalente.

---

## 36. Deploy PROD manual/gated

Workflow (`.github/workflows/deploy-prod.yml`):

```yaml
name: Deploy PROD

on:
  workflow_dispatch:
    inputs:
      image_tag:
        description: "Tag da imagem já validada em DEV (ex.: sha-a91c302)"
        required: true
        type: string

permissions:
  contents: read

jobs:
  deploy:
    runs-on: [self-hosted, linux, deploy]
    environment: production

    steps:
      - uses: actions/checkout@v4

      - name: Deploy
        run: |
          docker compose pull app
          IMAGE_TAG=${{ inputs.image_tag }} docker compose up -d app

      - name: Health check
        run: curl --fail https://prod.example/health
```

O `environment: production` exige que o *environment* `production` esteja configurado no GitHub (passo 34), com os *required reviewers* aprovando antes do job rodar.

---

## 37. Preflight

Antes de alterar:

- imagem existe;
- espaço em disco;
- Compose válido;
- versão anterior registrada;
- banco compatível.

---

## 38. Deploy

Promover mesma imagem de DEV.

---

## 39. Health PROD

Verificar:

```text
/health
/version
```

---

## 40. Smoke PROD

Fluxo seguro e não destrutivo.

---

## 41. Observabilidade

Instalar:

```text
Prometheus
Grafana
Loki
Node Exporter
```

conforme arquitetura.

---

## 42. Dashboard

Mostrar:

```text
version
errors
latency
CPU
RAM
disk
```

---

## 43. Deployment marker

Registrar horário e versão.

---

## 44. Simular falha

Faça em laboratório uma versão com health falhando.

Deploy em DEV.

---

## 45. Detectar

Pipeline deve marcar falha.

---

## 46. Rollback DEV

Voltar versão anterior.

---

## 47. Simular PROD em ambiente seguro

Não provoque falha deliberada em produção real.

Use VM de laboratório ou ambiente de treinamento.

---

## 48. Script rollback

Recebe:

```text
previous SHA
```

e reaplica.

---

## 49. Testar restore do banco

Em ambiente isolado:

```text
backup
restore
validation
```

---

## 50. Security scan

Adicionar ao workflow de CI:

```yaml
      - name: Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Trivy (imagem)
        uses: aquasecurity/trivy-action@0.24.0
        with:
          image-ref: ghcr.io/${{ github.repository }}:sha-${{ github.sha }}
          severity: HIGH,CRITICAL
          exit-code: 1
```

conforme política.

---

## 51. Secret leak exercise

Use uma string falsa de teste detectável, nunca credencial real.

Confirme scanner.

---

## 52. Dependency update exercise

Abrir PR de atualização.

Passar por CI.

---

## 53. Runner outage exercise

Parar runner:

```bash
sudo ./svc.sh stop
```

Observar job em fila.

Religar.

---

## 54. Disk pressure exercise

Não encha disco real intencionalmente.

Use monitoramento/limites em ambiente de laboratório.

---

## 55. Multiple runners

Adicionar segundo runner.

Observar paralelismo.

---

## 56. E2E sharding

Dividir suite quando existirem runners suficientes.

---

## 57. Nightly

Criar `.github/workflows/e2e-nightly.yml`:

```yaml
name: E2E Nightly

on:
  schedule:
    - cron: "0 3 * * *"
  workflow_dispatch:

jobs:
  e2e:
    runs-on: [self-hosted, linux, e2e]

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run test:e2e -- --grep-invert @smoke

      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
```

Executar full regression (horário em UTC).

---

## 58. Release

Criar tag:

```bash
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```

---

## 59. Release notes

Incluir:

- SPECs;
- PRs;
- SHA;
- mudanças.

---

## 60. Audit exercise

Responder:

```text
qual versão está em PROD?
qual PR introduziu?
qual SPEC?
quais testes passaram?
quem aprovou?
```

Se não consegue responder, rastreabilidade precisa melhorar.

---

## 61. Runbook runner offline

Criar:

```text
docs/runbooks/runner-offline.md
```

---

## 62. Runbook deploy rollback

Criar:

```text
docs/runbooks/rollback.md
```

---

## 63. Post-mortem exercise

Simular incidente em laboratório e escrever post-mortem.

---

## 64. Checklist final de CI

- [ ] PR.
- [ ] Lint.
- [ ] Unit.
- [ ] Integration.
- [ ] Build.
- [ ] E2E smoke.
- [ ] Required checks.
- [ ] Concurrency.
- [ ] Artifacts.
- [ ] Cleanup.

---

## 65. Checklist final do runner

- [ ] Usuário dedicado.
- [ ] Docker.
- [ ] systemd.
- [ ] labels.
- [ ] disk monitoring.
- [ ] patches.
- [ ] sem secrets persistentes.
- [ ] rebuild docs.

---

## 66. Checklist DEV

- [ ] Artifact por SHA.
- [ ] Deploy automático.
- [ ] Health.
- [ ] E2E.
- [ ] Visual validation.
- [ ] Logs.
- [ ] Version endpoint.

---

## 67. Checklist PROD

- [ ] Gate humano.
- [ ] Artifact igual ao DEV.
- [ ] Preflight.
- [ ] Health.
- [ ] Smoke.
- [ ] Monitoring.
- [ ] Rollback.
- [ ] Audit.

---

## 68. Checklist de segurança

- [ ] MFA.
- [ ] Least privilege.
- [ ] CI sem PROD secrets.
- [ ] Deploy runner separado quando possível.
- [ ] Secret scanning.
- [ ] Dependency scanning.
- [ ] Container scanning.
- [ ] SSH keys dedicadas.
- [ ] Firewall.

---

## 69. Checklist de documentação

- [ ] README.
- [ ] CONTRIBUTING.
- [ ] SPEC template.
- [ ] PLAN template.
- [ ] ADR.
- [ ] Runbooks.
- [ ] Architecture.
- [ ] Recovery.

---

## 70. Resultado final esperado

```text
IDEIA
 |
 SPEC
 |
 PLAN
 |
 BRANCH
 |
 CODE
 |
 PR
 |
 CI
 |
 E2E
 |
 MERGE
 |
 IMAGE
 |
 DEV
 |
 VALIDATION
 |
 APPROVAL
 |
 PROD
 |
 OBSERVABILITY
 |
 FEEDBACK
```

---

## 71. Maturidade futura

Depois do laboratório:

```text
ephemeral runners
autoscaling
preview environments
visual regression
OpenTelemetry
SBOM/signing
blue/green
```

Somente quando necessidade justificar.

---

## 72. Encerramento

O objetivo do guia não é maximizar quantidade de ferramentas.

É criar um processo em que:

```text
mudanças são compreendidas
testadas
rastreáveis
seguras
reversíveis
```

---

**Fim do Volume 16 — Laboratório Completo**

---

## Fontes

### Self-hosted runners

- [Adding self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners) — comprova o fluxo de registro do runner com `config.sh` e o token de registro gerado automaticamente com validade limitada (passo 14).
- [Configuring the self-hosted runner application as a service](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/configuring-the-self-hosted-runner-application-as-a-service) — comprova os comandos `sudo ./svc.sh install/start/status` para rodar o runner como serviço systemd persistente (passo 14, exercício 53).
- [Using labels with self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/using-labels-with-self-hosted-runners) — comprova o uso de labels em runners self-hosted para roteamento de jobs (passo 15, `runs-on: [self-hosted, linux, ci]`).

### Workflows e sintaxe do GitHub Actions

- [Using concurrency](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/using-concurrency) — comprova a chave `concurrency` com `group` e `cancel-in-progress: true` usada no workflow de CI (passo 13).
- [Storing workflow data as artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts) — comprova o uso de `actions/upload-artifact@v4` para publicar relatórios do Playwright em caso de falha (passo 24, passo 57).
- [Events that trigger workflows — schedule](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#schedule) — comprova a sintaxe `on.schedule` com cron para o workflow nightly de E2E (passo 57).
- [Automatic token authentication (GITHUB_TOKEN)](https://docs.github.com/en/actions/security-guides/automatic-token-authentication) — comprova o uso da chave `permissions` (`contents: read`, `packages: write`) e o token automático `GITHUB_TOKEN` usado no login do registry (passos 13, 26).
- [Using environments for deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment) — comprova a criação de *environments* no GitHub com *required reviewers* que gateiam a execução do job antes de rodar, base do `environment: production` no workflow de deploy (passos 34 e 36).

### Branch protection

- [About protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches) — comprova a exigência de PR e de status checks (CI PASS) antes do merge em `main` (passo 17).

### Build e push de imagens Docker

- [docker/build-push-action](https://github.com/docker/build-push-action) — comprova os inputs `context`, `push`, `tags`, `cache-from` e `cache-to` usados no workflow de build (passo 26), incluindo o uso recomendado em conjunto com `docker/setup-buildx-action`.
- [docker/login-action](https://github.com/docker/login-action) — comprova o login no `ghcr.io` usando `github.actor` e `secrets.GITHUB_TOKEN` (passo 26).
- [GitHub Actions cache for Buildx](https://docs.docker.com/build/ci/github-actions/cache/) — comprova a sintaxe `cache-from: type=gha` e `cache-to: type=gha,mode=max` usada no build oficial (passo 26).
- [docker buildx build (Docker CLI reference)](https://docs.docker.com/reference/cli/docker/buildx/build/) — comprova as flags `--tag` e `--push` do comando manual equivalente ao workflow (passo 26).
- [Working with the Container registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry) — comprova o uso do GitHub Container Registry (`ghcr.io`), o formato de tag `ghcr.io/OWNER/IMAGE` e a autenticação via `GITHUB_TOKEN` (passo 27).

### Segurança: scanning de segredos e de imagens

- [gitleaks/gitleaks-action](https://github.com/gitleaks/gitleaks-action) — comprova a action usada para detecção de segredos hardcoded no workflow de CI (passo 50).
- [aquasecurity/trivy-action](https://github.com/aquasecurity/trivy-action) — comprova os inputs `image-ref`, `severity` e `exit-code` usados para escanear a imagem publicada (passo 50).
