# Volume 13 — Arquiteturas de Referência

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 13_ARQUITETURAS_DE_REFERENCIA.md  
**Versão:** 0.2.0

---

## 1. Objetivo

Este volume apresenta arquiteturas práticas reutilizáveis para diferentes tipos de projeto.

---

## 2. Referência A — Node.js Simples

```text
GitHub
 |
 v
CI Runner
 |
 +-- npm ci
 +-- lint
 +-- unit
 +-- build
 |
 v
Docker image
 |
 v
DEV
 |
 v
PROD
```

Estrutura:

```text
src/
tests/
Dockerfile
package.json
.github/workflows/
```

---

## 3. Node.js + MariaDB

```text
PR
 |
 v
CI
 |
 +-- Node
 +-- MariaDB container
 |
 v
migrations
 |
 v
integration
 |
 v
build image
```

DEV:

```text
app container
db DEV
```

PROD:

```text
app artifact promovido
db PROD persistente
```

---

## 4. Node.js + Redis

Use Redis container em integração.

Teste:

- cache hit;
- cache miss;
- expiração;
- indisponibilidade controlada.

---

## 5. Node.js + MQTT

```text
Node app
 |
 v
Mosquitto
 |
 +-- commands
 +-- status
```

CI sobe broker isolado.

Testes verificam publish/subscribe.

---

## 6. PHP + MariaDB

```text
GitHub
 |
 v
CI Runner
 |
 +-- composer install
 +-- PHPStan/Psalm
 +-- PHPUnit
 +-- MariaDB
 |
 v
Docker image
 |
 v
DEV/PROD
```

---

## 7. PHP Legado sem Docker Runtime

Mesmo que produção execute em Apache/PHP tradicional, CI pode usar containers para dependências.

Migração para Docker pode ser gradual.

---

## 8. PHP + MQTT

Arquitetura:

```text
PHP backend
 |
 v
broker MQTT
```

Testes podem utilizar Mosquitto isolado.

Não usar broker PROD.

---

## 9. Frontend SPA + API

```text
Browser
 |
 v
Frontend
 |
 v
API
 |
 v
DB
```

CI:

```text
frontend lint/unit
backend unit/integration
contract
E2E smoke
```

---

## 10. Frontend Separado do Backend (Polyrepo)

Repos separados:

```text
frontend repo
backend repo
```

Cada um possui CI próprio, com seu ciclo de release independente.

**Vantagens reais:**

- ownership claro por time/produto (cada repo tem seus próprios CODEOWNERS e branch protection);
- pipelines mais rápidos, porque cada CI só constrói o que mudou;
- versionamento independente (frontend pode fazer release sem esperar backend);
- menor blast radius: um erro de configuração de CI num repo não trava os outros.

**Custos reais:**

- mudanças que cruzam frontend+backend exigem coordenar dois PRs, em dois repos;
- duplicação de configuração de workflow entre repos (mitigada com **reusable workflows**, ver 10.1);
- contrato entre serviços precisa de disciplina (versionamento de API, contract testing — ver seção 18);
- dependências compartilhadas (design system, tipos, SDK interno) exigem publicação em pacote (npm/registry privado) em vez de import direto de arquivo.

Integração pode rodar em DEV ou em pipeline coordenado (workflow que dispara o deploy de ambos após os dois CIs passarem).

---

### 10.1 Reusable Workflows entre Repositórios da Mesma Organização

Quando vários repositórios da mesma organização repetem a mesma sequência de jobs (lint, build, testes, publicação de imagem), a duplicação de YAML vira dívida técnica. O GitHub Actions resolve isso com **reusable workflows** (`workflow_call`).

Repositório central (ex.: `org/actions-workflows`), arquivo `.github/workflows/node-ci.yml`:

```yaml
on:
  workflow_call:
    inputs:
      node-version:
        required: false
        type: string
        default: "20"
    secrets:
      NPM_TOKEN:
        required: false

jobs:
  build-test:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci
      - run: npm test
```

Repositório consumidor (`org/produto-a`), chamando o workflow acima:

```yaml
jobs:
  ci:
    uses: org/actions-workflows/.github/workflows/node-ci.yml@main
    with:
      node-version: "20"
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

Pontos importantes:

- o repositório que expõe o workflow reusável precisa ser público, ou a organização precisa habilitar explicitamente "Access to this repository" para os repositórios consumidores (Settings → Actions → General);
- fixar a referência (`@main`, `@v1`, ou melhor, um SHA) define o quão "congelado" o comportamento fica; usar uma tag semver (`@v1`) é o meio-termo recomendado entre reprodutibilidade e manutenção;
- secrets **não** são herdados automaticamente entre organizações/repos — cada chamador precisa repassar explicitamente o que o workflow reusável declara em `secrets:`;
- reusable workflows podem encadear (um workflow reusável pode chamar outro), mas o GitHub limita a profundidade de aninhamento (4 níveis) e o número de workflows chamados por execução.

---

## 11. Monorepo Frontend/Backend

```text
apps/
  frontend/
  api/
packages/
  common/
```

**Vantagens reais:**

- refactors que cruzam frontend+backend cabem em um único PR, com um único CI validando tudo junto;
- dependências internas (`packages/common`) são importadas diretamente, sem publicação em registry;
- histórico e busca de código unificados; um `git blame` enxerga o sistema inteiro.

**Custos reais:**

- sem filtro de paths, todo PR dispara CI completo (frontend + backend + todos os pacotes), o que fica caro e lento conforme o repo cresce;
- branch protection e CODEOWNERS ficam mais grosseiros (por path, não por repo inteiro);
- um runner self-hosted lento afeta todos os times ao mesmo tempo.

CI deve entender dependências compartilhadas e, na prática, isso significa **paths-filter** + **workflows por pacote**.

### paths-filter

Evita rodar o CI de um pacote quando só outro pacote mudou:

```yaml
jobs:
  changes:
    runs-on: self-hosted
    outputs:
      frontend: ${{ steps.filter.outputs.frontend }}
      api: ${{ steps.filter.outputs.api }}
    steps:
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            frontend:
              - 'apps/frontend/**'
              - 'packages/common/**'
            api:
              - 'apps/api/**'
              - 'packages/common/**'

  ci-frontend:
    needs: changes
    if: needs.changes.outputs.frontend == 'true'
    uses: ./.github/workflows/frontend-ci.yml

  ci-api:
    needs: changes
    if: needs.changes.outputs.api == 'true'
    uses: ./.github/workflows/api-ci.yml
```

Note que `packages/common/**` aparece nos dois filtros: um pacote compartilhado precisa disparar CI de todo mundo que depende dele, senão uma quebra de contrato passa despercebida.

### Workflow por Pacote

Cada `apps/*` mantém seu próprio arquivo de workflow reusável (`frontend-ci.yml`, `api-ci.yml`), chamado condicionalmente pelo orquestrador acima. Isso combina bem com a técnica de reusable workflows da seção 10.1: o mesmo `node-ci.yml` central pode ser chamado tanto pelo monorepo quanto por repositórios externos.

Alternativas de ferramenta para detectar mudanças em monorepo: Nx e Turborepo oferecem grafo de dependências e cache de build "affected-only", mais sofisticado que `paths-filter` puro, mas trazem uma camada extra de ferramenta e convenção de projeto.

---

### 11.1 Composite Actions vs Reusable Workflows

As duas mecânicas resolvem duplicação, mas em escopos diferentes — confundi-las é o erro mais comum ao organizar CI em GitHub Actions.

| | Composite action | Reusable workflow |
|---|---|---|
| Escopo | um conjunto de **steps** dentro de um job | um ou mais **jobs** completos |
| Onde roda | dentro do job que a chama, no mesmo runner | em job(s) próprio(s), podem ter `runs-on` diferente |
| Pode ter matrix própria? | não | sim |
| Pode ter `permissions`/`environment` próprios? | não (herda do job chamador) | sim |
| Sintaxe de chamada | `uses:` dentro de `steps:` | `uses:` dentro de `jobs.<id>` com `workflow_call` |
| Uso típico | "checkout + setup + cache" repetido em vários jobs | pipeline inteiro de CI/CD compartilhado entre repos |

Use **composite action** quando o que se repete é uma sequência curta de steps dentro do mesmo job (ex.: configurar Node + instalar dependências + restaurar cache), especialmente se ela é chamada várias vezes no mesmo workflow ou combinada com outros steps específicos do job.

Use **reusable workflow** quando o que se repete é o job inteiro (ou a pipeline inteira), especialmente quando isso precisa valer para repositórios diferentes da organização, com seus próprios secrets e ambientes.

Exemplo de composite action (`actions/setup-node-cache/action.yml`):

```yaml
runs:
  using: "composite"
  steps:
    - uses: actions/setup-node@v4
      with:
        node-version: "20"
    - run: npm ci
      shell: bash
```

Chamada dentro de um job comum:

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: ./.github/actions/setup-node-cache
  - run: npm run build
```

Nada impede combinar os dois: um reusable workflow, por dentro, pode usar composite actions em seus steps.

---

## 12. Sistema com Worker

```text
API
 |
 v
queue
 |
 v
worker
```

Teste integração deve validar processamento assíncrono.

---

## 13. Sistema com Webhook

```text
External
 |
 v
Webhook endpoint
 |
 v
queue/service
 |
 v
result
```

Teste:

- assinatura;
- idempotência;
- replay;
- payload inválido.

---

## 14. Sistema IoT

```text
Devices
 |
 MQTT
 |
 Backend
 |
 DB
 |
 Dashboard
```

Camadas de teste:

```text
unit protocol
MQTT integration
API
E2E dashboard
```

---

## 15. Aplicação com ESP32

Firmware e backend podem ter pipelines separados.

Firmware:

```text
compile
static checks
artifact firmware
```

Backend:

```text
normal CI/CD
```

Integração hardware real pode ser laboratório específico.

---

## 16. Hardware-in-the-Loop

Uma evolução:

```text
dedicated runner
 |
 USB/serial
 |
 ESP32 real
```

Testes devem ser isolados e não rodar em toda PR inicialmente.

---

## 17. Microsserviços

```text
gateway
 |
 +-- service A
 +-- service B
 +-- service C
```

Cada serviço:

```text
CI independente
artifact independente
```

E2E integrado separado.

---

### 17.1 Matrix Strategy

Quando o mesmo workflow precisa rodar contra combinações de versão de runtime, sistema operacional ou serviço, `strategy.matrix` evita duplicar jobs manualmente:

```yaml
jobs:
  test:
    strategy:
      fail-fast: false
      max-parallel: 4
      matrix:
        node-version: ["18", "20", "22"]
        service: [service-a, service-b, service-c]
        include:
          - node-version: "22"
            service: service-a
            experimental: true
        exclude:
          - node-version: "18"
            service: service-c
    runs-on: self-hosted
    continue-on-error: ${{ matrix.experimental == true }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test --workspace=${{ matrix.service }}
```

Pontos que valem a pena fixar:

- `fail-fast: true` (padrão) cancela toda a matrix assim que uma combinação falha — bom para feedback rápido em PR, ruim quando se quer ver o resultado de todas as combinações (ex.: matriz de compatibilidade de versões). Usar `fail-fast: false` nesse segundo caso;
- `max-parallel` limita quantos jobs da matrix rodam ao mesmo tempo — essencial em runners self-hosted com capacidade finita, para não estourar todos os runners de uma vez com uma única matrix grande;
- `include` adiciona combinações extras (ou acrescenta campos a uma combinação já gerada), `exclude` remove combinações específicas do produto cartesiano;
- combinar `continue-on-error` com uma entrada marcada (ex.: `experimental: true` via `include`) implementa "allow failure" seletivo, útil para testar versões futuras (Node "current") sem quebrar o pipeline;
- em microsserviços, uma matrix por serviço só faz sentido quando os serviços compartilham o mesmo tipo de build/teste; serviços com stacks muito diferentes (Node vs PHP) são melhor servidos por workflows separados (ver seção 10.1).

---

## 18. Contract Testing em Microsserviços

Reduz dependência de E2E gigantesco.

Valide contratos entre serviços.

---

## 19. Banco por Serviço

Em arquitetura de microsserviços, compartilhamento de banco aumenta acoplamento.

Decisão depende do sistema.

---

## 20. Reverse Proxy

```text
Nginx/Traefik
 |
 +-- frontend
 +-- API
```

Configuração versionada.

---

## 21. Traefik

Proxy reverso com descoberta dinâmica de serviços: lê labels do Docker (ou da API do Coolify, do Kubernetes, etc.) e atualiza suas rotas sem reload manual nem restart. Isso o torna a opção natural quando produtos são implantados e removidos com frequência via CI/CD, cada um como um container/stack diferente na mesma infraestrutura.

Nginx continua excelente opção quando a topologia é estável (poucos serviços, mudam raramente) e não vale o custo operacional extra de manter descoberta dinâmica.

### 21.1 Isolamento por Produto/Stack com Traefik

Em uma infraestrutura com múltiplos produtos na mesma borda, cada stack deve expor apenas as labels do seu próprio roteamento — nunca reutilizar rede, banco ou fila de outro produto só porque estão na mesma máquina. Exemplo de labels típicas por serviço:

```yaml
services:
  produto-a-api:
    image: registry/produto-a:sha
    networks:
      - produto-a-net
    labels:
      - traefik.enable=true
      - traefik.http.routers.produto-a.rule=Host(`produto-a.exemplo.com`)
      - traefik.http.services.produto-a.loadbalancer.server.port=3000
```

Cada produto com sua própria rede Docker (`produto-a-net`, `produto-b-net`) garante que um container de um produto não alcança o banco/fila de outro por engano, mesmo compartilhando o mesmo host físico ou LXC.

### 21.2 Estratégias de Deploy com Coolify/Traefik

Coolify orquestra o ciclo `build → deploy → healthcheck → troca de tráfego`, delegando o roteamento a um Traefik na borda. As três estratégias clássicas mapeiam assim:

**Rolling deploy (padrão do Coolify para a maioria das aplicações):**

```text
build nova imagem
 |
 v
sobe novo container
 |
 v
healthcheck OK?
 |
 +-- não --> mantém container antigo, marca deploy como falho
 |
 v (sim)
Traefik atualiza rota para o novo container
 |
 v
container antigo é removido
```

Simples e barato, mas há uma janela (curta) em que ambas as versões podem receber tráfego, e não há como testar a versão nova isoladamente antes de expor todo o tráfego a ela.

**Blue-green:**

```text
green (versão atual) recebe 100% do tráfego
 |
 v
build e sobe blue (versão nova) em paralelo, mesma rede
 |
 v
smoke test direto em blue (via URL interna, sem passar pelo Traefik)
 |
 v
troca label/rule do Traefik: 100% do tráfego passa a ir para blue
 |
 v
mantém green de pé por um tempo (rollback = trocar a rota de volta)
 |
 v
depois de validado, desliga green
```

Em Coolify isso é obtido subindo uma segunda instância da mesma aplicação (ou usando "Preview Deployments"/ambientes distintos) e trocando o `Host()`/prioridade da rota no Traefik só depois de validar a instância nova — o corte de tráfego é atômico porque é o Traefik reconfigurando o roteamento, não um restart de container. O custo é rodar as duas versões simultaneamente durante a janela de validação (recursos duplicados).

**Canary:**

```text
100% tráfego -> versão estável
 |
 v
sobe versão canary em paralelo
 |
 v
Traefik divide tráfego: 95% estável / 5% canary
 (traefik.http.services.<nome>.loadbalancer.server.weight)
 |
 v
observa métricas/erros do canary
 |
 v
 +-- ok --> aumenta peso gradualmente até 100%
 |
 +-- não --> peso do canary volta a 0%, sem downtime
```

O Traefik suporta isso nativamente via **weighted round robin** entre serviços (um "serviço" lógico do Traefik apontando para múltiplos backends com pesos diferentes). Coolify não expõe canary por peso como recurso de UI de primeira classe; na prática, times que precisam disso versionam as labels de peso do Traefik diretamente no `docker-compose` da aplicação, ou usam um segundo recurso de aplicação no Coolify e ajustam manualmente/via script o peso das rotas.

Recomendação prática: rolling é suficiente para a maioria dos serviços internos com boa suíte de testes automatizados antes do deploy; blue-green vale para serviços com migração de schema arriscada ou que precisam de validação humana antes de virar 100% do tráfego; canary vale quando o custo de um bug em produção é alto e há telemetria suficiente para decidir automaticamente entre aumentar ou reverter o peso.

---

## 22. Registry

Todos os modelos containerizados convergem para:

```text
CI -> Registry -> environments
```

---

## 23. DEV Compartilhado

Se apenas um DEV:

```text
main
 |
 v
deploy DEV
```

PRs são validadas principalmente no CI antes do merge.

---

## 24. Preview Environment

Para maior paralelismo:

```text
PR #42 -> preview-42
PR #43 -> preview-43
```

Mais caro e complexo.

---

## 25. Sem Staging

Arquitetura de referência:

```text
CI forte
 |
 v
DEV
 |
 v
human validation
 |
 v
PROD
```

---

### 25.1 Ambientes Isolados por Produto/Stack com GitHub Environments

Quando a organização mantém vários produtos (cada um com seu próprio DEV/PROD, banco e fila) na mesma conta do GitHub, o recurso **Environments** (Settings → Environments no repositório) modela isso de forma nativa, em vez de depender só de secrets no nível do repositório:

```text
Repositório produto-a
  Environments:
    produto-a-dev
    produto-a-prod

Repositório produto-b
  Environments:
    produto-b-dev
    produto-b-prod
```

Cada environment carrega seu próprio conjunto de secrets e variáveis — um secret `DB_URL` em `produto-a-prod` nunca vaza para um job rodando em `produto-a-dev`, mesmo que ambos os jobs estejam no mesmo workflow:

```yaml
jobs:
  deploy-prod:
    environment: produto-a-prod
    runs-on: self-hosted
    steps:
      - run: ./deploy.sh
        env:
          DB_URL: ${{ secrets.DB_URL }}
```

Regras de proteção configuráveis por environment:

- **required reviewers** — o job fica pendente até alguém aprovar manualmente (essencial para o passo "human validation" antes de PROD);
- **wait timer** — atraso mínimo obrigatório antes do job rodar, útil como janela de "arrependimento" antes de um deploy automático;
- **deployment branches** — restringe quais branches podem disparar deploy para aquele environment (ex.: só `main` pode implantar em `produto-a-prod`), impedindo que uma branch de feature acidentalmente rode um job com `environment: produto-a-prod` e acesse os secrets de produção.

Isso substitui a prática antiga de prefixar secrets manualmente (`PRODUTO_A_PROD_DB_URL`, `PRODUTO_A_DEV_DB_URL`) no nível do repositório: com Environments, o nome do secret é o mesmo (`DB_URL`) em todos os ambientes, e o isolamento vem de qual environment o job declara — reduzindo o risco de copiar/colar o secret errado ao escrever o workflow. Para múltiplos produtos na mesma infraestrutura, o mesmo princípio de "nunca cruzar banco/fila/container entre produtos" se aplica um nível acima: nunca reaproveitar o mesmo environment do GitHub para dois produtos diferentes, mesmo que compartilhem o host.

---

## 26. Banco DEV

Não deve conter única cópia de informação importante.

É ambiente reconstruível.

---

## 27. Banco PROD

Backup, restore, monitoring, access control.

---

## 28. Deploy Node.js com Compose

```text
registry/app:sha
 |
 v
docker compose pull
 |
 v
docker compose up -d
 |
 v
health
```

---

## 29. Deploy PHP com Compose

Possível stack:

```text
nginx
php-fpm
worker
```

Banco separado/persistente.

---

## 30. CI para PHP sem Container Final

```text
composer artifact
 |
 v
rsync/SSH
```

Ainda deve possuir:

- version;
- rollback;
- releases directory.

---

## 31. Releases Directory

Modelo tradicional:

```text
/var/www/app/releases/
  a91c302/
  b72e510/

current -> b72e510
```

Rollback troca symlink.

---

## 32. Atomic Deploy Tradicional

```text
upload release
install dependencies
validate
switch current
```

Evita editar diretório ativo arquivo por arquivo.

---

## 33. Arquitetura Docker Preferencial

Para novos projetos:

```text
image imutável
```

simplifica artifact.

---

## 34. Observabilidade Comum

Todas as arquiteturas devem oferecer:

```text
/health
/version
structured logs
metrics
```

---

## 35. Segurança Comum

Todas:

- secrets externos;
- least privilege;
- branch protection;
- CI sem PROD;
- deploy protegido.

---

## 36. Template de Projeto Node

```text
.
├── src/
├── tests/
├── docs/
├── scripts/
├── Dockerfile
├── compose.test.yml
├── package.json
└── .github/workflows/
```

---

## 37. Template PHP

```text
.
├── src/
├── tests/
├── public/
├── docs/
├── scripts/
├── Dockerfile
├── composer.json
└── .github/workflows/
```

---

## 38. Template IoT Backend

```text
.
├── src/
│   ├── mqtt/
│   ├── api/
│   └── services/
├── tests/
├── compose.test.yml
└── ...
```

---

## 39. Separar Adapter MQTT

Encapsular broker facilita testes.

---

## 40. Separar Repository

Encapsular banco facilita unit/integration testing.

---

## 41. Ports and Adapters

Arquitetura hexagonal pode ser útil em sistemas com muitas integrações.

Não é obrigatória.

---

## 42. Referência Mínima Recomendada

Para um sistema web moderno:

```text
Frontend
API Node
MariaDB
Redis opcional
Docker
GitHub Actions
Self-hosted runner
Playwright
```

---

## 43. Referência Automação

```text
Node API
MariaDB
Mosquitto
WebSocket
Frontend
Docker
```

---

## 44. Testes Dessa Referência

```text
unit business
integration DB
integration MQTT
API
WebSocket integration
E2E browser
```

---

## 45. WebSocket

Teste:

- conexão;
- autenticação;
- evento;
- reconexão;
- disconnect.

---

## 46. Real-Time E2E

Evite sleeps fixos.

Espere evento/estado.

---

## 47. Arquitetura de Runners

Inicial:

```text
runner-01
CI + E2E
```

Evolução:

```text
runner-ci
runner-e2e
runner-deploy
```

---

## 48. Escolha por Risco

Projeto simples não precisa de arquitetura de multinacional.

Use a menor arquitetura que mantenha:

```text
qualidade
segurança
rastreabilidade
rollback
```

---

## 49. Checklist de Nova Arquitetura

- [ ] Qual artifact?
- [ ] Qual banco?
- [ ] Quais integrações?
- [ ] Como testar?
- [ ] Como fazer deploy?
- [ ] Como voltar?
- [ ] Quais secrets?
- [ ] Qual observabilidade?
- [ ] Qual runner?

---

## 50. Próximo Volume

**Volume 14 — Otimização e Escalabilidade do CI/CD**

---

**Fim do Volume 13 — Arquiteturas de Referência**

---

## Fontes

### GitHub Actions — Reusable Workflows e Composite Actions

- [Reusing workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows) — GitHub Docs oficial: sustenta `workflow_call`, a exigência de repassar `secrets:` explicitamente (não há herança automática entre chamador e workflow reusável, salvo `secrets: inherit`) e o limite de aninhamento de workflows reusáveis (a documentação atual indica até 9 níveis de workflows reusáveis mais o chamador, valor a conferir/ajustar no texto que hoje cita "4 níveis").
- [Creating a composite action](https://docs.github.com/en/actions/sharing-automations/creating-actions/creating-a-composite-action) — GitHub Docs oficial: confirma que uma composite action agrupa vários steps em uma única action, executada como um único step dentro do job chamador (seção 11.1).

### GitHub Actions — Matrix e Environments

- [Using a build matrix for your jobs](https://docs.github.com/en/actions/using-jobs/using-a-build-matrix-for-your-jobs) — GitHub Docs oficial: sustenta `strategy.matrix`, `fail-fast`, `max-parallel`, `include`/`exclude` e `continue-on-error` combinados com matrix (seção 17.1).
- [Using environments for deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment) — GitHub Docs oficial: sustenta secrets/variáveis por Environment, required reviewers, wait timer e restrição de deployment branches (seção 25.1).

### Ferramenta de Terceiros — paths-filter

- [dorny/paths-filter](https://github.com/dorny/paths-filter) — repositório oficial da action: sustenta o uso de `paths-filter` para disparar CI condicionalmente por pacote em monorepo (seção 11).

### Traefik

- [Docker provider (Traefik)](https://doc.traefik.io/traefik/reference/install-configuration/providers/docker/) — documentação oficial do Traefik: sustenta a descoberta dinâmica de serviços via labels do Docker, sem necessidade de restart manual (seções 21 e 21.1).
- [HTTP load balancing services (Traefik)](https://doc.traefik.io/traefik/reference/routing-configuration/http/load-balancing/service/) — documentação oficial do Traefik: sustenta o *weighted round robin* entre serviços via peso (`server.weight`), base da estratégia de canary deploy descrita na seção 21.2.

### Coolify

- [Traefik (Coolify Docs)](https://coolify.io/docs/knowledge-base/proxy/traefik) — documentação oficial do Coolify: confirma que o Coolify usa Traefik como proxy reverso padrão na borda, sustentando a descrição do fluxo `build → deploy → healthcheck → troca de tráfego` da seção 21.2.
