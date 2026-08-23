# Volume 14 — Otimização e Escalabilidade do CI/CD

**Projeto:** Guia Pessoal de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 14_OTIMIZACAO_E_ESCALABILIDADE.md  
**Versão:** 0.1.0

---

## 1. Objetivo

Reduzir tempo de feedback e aumentar capacidade sem sacrificar confiabilidade.

Princípio:

```text
primeiro medir
depois otimizar
```

---

## 2. Métricas fundamentais

Meça:

```text
queue time
setup time
dependency install
unit duration
integration duration
E2E duration
build duration
deploy duration
```

---

## 3. Gargalo

Não assuma que E2E é sempre o maior problema.

Pode ser:

- `npm ci`;
- Docker build;
- fila;
- browser install;
- banco;
- checkout.

---

## 4. Pareto

Frequentemente 20% das etapas consomem 80% do tempo.

Ataque primeiro os maiores custos.

---

## 5. Cache npm

Use `actions/cache@v4` com chave derivada do lockfile via `hashFiles`, e `restore-keys` como fallback parcial:

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: npm-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      npm-${{ runner.os }}-
```

Alternativa mais simples para Node: `actions/setup-node@v4` com `cache: 'npm'` já cuida do cache do diretório de downloads (`~/.npm`) usando o lockfile como chave, sem precisar declarar `actions/cache` manualmente.

Cache o diretório de downloads do gerenciador de pacotes (`~/.npm`), não `node_modules`. `node_modules` cacheado direto engessa binários nativos, symlinks e a resolução do lockfile entre runners com SO/arch diferentes, e tende a divergir silenciosamente do lockfile.

Limites do cache do GitHub Actions: repositório tem teto de 10 GB (entradas mais antigas são evictadas automaticamente para abrir espaço); uma entrada não usada por 7 dias é removida; a chave de cache é imutável. Uma vez gravada não é sobrescrita, por isso a chave precisa mudar (via `hashFiles`) quando o conteúdo muda.

---

## 6. Cache Composer

Mesmo padrão do npm, com o cache de downloads do Composer (`~/.composer/cache` ou o path retornado por `composer config cache-files-dir`) chaveado por `hashFiles('**/composer.lock')` e `restore-keys` sem o hash para aproveitar parcialmente uma chave antiga.

O lockfile continua fonte de versões; o cache só evita rebaixar pacotes já resolvidos.

---

## 7. Docker layer cache

Dockerfile bem estruturado continua a primeira otimização:

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

Isso separa a camada de dependências (que muda pouco) da camada de código (que muda a cada commit), mas só ajuda de fato quando o cache de camadas persiste entre execuções. Isso exige BuildKit com backend de cache explícito via `docker buildx build` (ou `docker/build-push-action@v6`, que já usa buildx por padrão):

```yaml
- uses: docker/setup-buildx-action@v3

- uses: docker/build-push-action@v6
  with:
    context: .
    push: true
    tags: registry.exemplo/app:${{ github.sha }}
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

Opções de backend de cache:

| Backend | Descrição |
|---|---|
| `type=gha` | Cache hospedado pelo GitHub Actions (por repositório, mesmos limites/eviction do `actions/cache`) |
| `type=registry` | Cache publicado como imagem/manifesto num registry (bom p/ compartilhar entre runners self-hosted) |
| `type=local` | Cache em diretório do disco local (rápido em runner persistente, mas é o disco que se quer monitorar) |

`mode=max` grava todas as camadas intermediárias no cache (não só a camada final), o que é necessário para builds multi-stage aproveitarem cache entre estágios. Custa mais espaço/tempo de export em troca de mais acertos de cache.

Em runner self-hosted persistente, `type=local` com `--cache-to type=local,dest=/caminho,mode=max` evita depender de rede, mas o diretório cresce sem limite se nada fizer `docker buildx prune` (ver seção 39).

---

## 8. Browser cache

Playwright baixa browsers grandes (Chromium/Firefox/WebKit, várias centenas de MB no total).

Em runner efêmero, cachear o diretório de browsers (`~/.cache/ms-playwright` no Linux) com `actions/cache@v4` chaveado pela versão do Playwright (ex.: `hashFiles('**/package-lock.json')` ou a versão do pacote) evita rebaixar a cada job; some com `restore-keys` para não falhar quando a versão exata não bate.

Em runner persistente, os browsers já instalados podem ser reaproveitados diretamente do disco entre jobs quando a versão instalada é compatível com a exigida pelo lockfile. Nesse caso o cache do Actions é dispensável, mas cabe checar a versão antes de assumir compatibilidade (`playwright install --with-deps` falha ou reinstala quando diverge).

Gerencie espaço: cada versão nova do Playwright normalmente não remove a anterior sozinha.

---

## 9. Imagem de runner preparada

Uma evolução:

```text
runner image
 |
 +-- Docker
 +-- Node
 +-- browsers
```

reduz setup por job.

---

## 10. Runners efêmeros com imagem pronta

Combina:

```text
isolamento
+
setup rápido
```

Exige provisioning automatizado.

Trade-off central entre runner efêmero e persistente:

| Modo | Vantagens | Desvantagens |
|---|---|---|
| Efêmero (registra e desregistra a cada job) | Isolamento forte: job nunca herda estado/segredo/cache de job anterior; reduz superfície de ataque de repo público/PR externo | Custo de I/O e boot a cada execução: clonar/baixar imagem, subir VM/container, instalar toolchain; depende de provisioning automatizado (imagem pronta, template, orquestrador) |
| Persistente (fica registrado por dias/semanas) | Setup por job quase zero, sem custo de boot repetido; mais simples de operar sem orquestrador | Risco de vazamento entre jobs (cache, variável de ambiente, processo zumbi, credencial em disco); acumula lixo (containers, imagens, workspaces, logs) — exige limpeza ativa (seções 37-39); superfície maior para repo público ou runner que roda PR de fork |

Para repositório público ou que aceita PR de fork, prefira efêmero. Persistente com segredo em ambiente compartilhado é vetor de exfiltração. Para repositório privado com poucos contribuidores confiáveis, persistente pode ser aceitável e mais barato em I/O.

---

## 11. Concurrency cancel

```yaml
cancel-in-progress: true
```

economiza execução de commits antigos.

---

## 12. Fail fast

```text
lint -> unit -> expensive tests
```

Falhar cedo economiza recursos.

---

## 13. Paralelização

Com capacidade:

```text
lint
unit
integration
```

podem rodar em paralelo.

---

## 14. Dependência correta

Não coloque `needs` onde não existe dependência lógica.

Dependências artificiais aumentam tempo.

---

## 15. E2E shards

```text
1/4
2/4
3/4
4/4
```

Com quatro runners, reduz wall-clock.

---

## 16. Workers

Dentro do mesmo runner:

```bash
playwright test --workers=4
```

Ajuste por CPU/RAM.

---

## 17. Oversubscription

8 workers em máquina de 4 cores pode piorar tempo e flakiness.

---

## 18. Banco compartilhado

Paralelismo exige isolamento.

Use bancos/schemas/dados únicos.

---

## 19. Test impact

Mapeie mudança para testes.

---

## 20. Full regression como rede de segurança

Mesmo com seleção de testes:

```text
nightly full
```

permanece.

---

## 21. Path filters

Evite workflow frontend se apenas docs independentes mudaram.

---

## 22. Docs-only PR

Pode executar apenas:

```text
markdown lint
link check
```

quando seguro.

---

## 23. Monorepo graph

Ferramentas de monorepo podem calcular dependências.

Adote somente se projeto justificar.

---

## 24. Build seletivo

Não reconstruir serviços não afetados.

---

## 25. Artifact reuse

Não repetir:

```text
npm build
```

em jobs diferentes.

Produza artifact em um job e reutilize.

---

## 26. Upload/download artifact

Útil quando jobs estão em runners diferentes.

Compare custo de transferência com rebuild.

---

## 27. Workspace compartilhado

Em self-hosted persistente, não dependa implicitamente do workspace entre jobs.

Isso reduz portabilidade.

---

## 28. Runner count

Se queue time cresce:

```text
adicionar runner
```

pode ser melhor que otimizar testes.

---

## 29. Runner especializado

```text
ci
e2e
deploy
arm64
```

---

## 30. Scale-up

Mais CPU/RAM.

Bom quando job único é pesado.

---

## 31. Scale-out

Mais runners.

Bom quando muitos jobs ficam em fila.

---

## 32. Auto-scaling

Em ambiente cloud/virtualização, runners podem ser criados sob demanda a partir de um template/imagem, atendem um job (idealmente como efêmero) e são destruídos em seguida.

Sem Kubernetes, isso normalmente é implementado com um webhook do GitHub (evento `workflow_job` com status `queued`) disparando a criação de VM/container a partir de um template (ver seção 35). É fase avançada: exige orquestração própria, imagem pronta e monitoramento de falha de provisionamento.

---

## 33. Actions Runner Controller

O **Actions Runner Controller (ARC)** é a solução oficial da GitHub para autoscaling de runners self-hosted em Kubernetes. Ele expõe dois modos de escala:

| Modo | Descrição |
|---|---|
| `RunnerDeployment` (legado) | Pool de runners com scale-up/down por polling |
| `RunnerScaleSet` (atual) | Escuta o job diretamente e escala com granularidade de 1 job = 1 pod efêmero |

O modelo atual (`RunnerScaleSet`, instalado via chart Helm `gha-runner-scale-set`) usa runners efêmeros por padrão: cada job sobe um pod, registra, executa e é descartado. Isso resolve o problema de acúmulo de lixo em disco (seção 37) porque não há disco persistente entre jobs, mas paga o custo de I/O/boot descrito na seção 10 a cada execução (pull de imagem do runner, cold start do pod).

Só considerar ARC quando a escala e a operação já justificarem manter um cluster Kubernetes. Não adote Kubernetes só para ganhar autoscaling de runner (ver seção 34). Para poucos runners, VM templates em Proxmox/cloud (seção 35) ou runners persistentes bem geridos (seções 37-39) costumam ser mais simples de operar.

---

## 34. Não adotar Kubernetes cedo

Kubernetes pode adicionar mais complexidade que valor em projetos pequenos.

---

## 35. VM templates

Em Proxmox/cloud:

```text
template runner
 |
 clone
 |
 register
```

é uma alternativa simples.

---

## 36. Ephemeral registration

Runners descartáveis devem ser registrados com a flag `--ephemeral` no `config.sh`/`config.cmd` do runner (ou equivalente no ARC/`RunnerScaleSet`, que já é efêmero por padrão). Um runner efêmero se desregistra automaticamente após concluir um único job. Não confie em script externo para desregistrar manualmente, pois um crash no meio do job pode deixar o registro órfão no GitHub sem que o processo tenha rodado o cleanup.

Combine com fetch raso e limpeza de workspace (seção 36-A) para manter o custo de I/O por job baixo mesmo em provisionamento automatizado.

---

## 36-A. Checkout raso e limpeza de workspace

`actions/checkout@v4` clona por padrão com `fetch-depth: 1` (raso), o que já evita baixar todo o histórico. Ajuste explicitamente quando precisar de mais:

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 1        # padrão; suficiente para build/test comuns
    # fetch-depth: 0      # histórico completo — só quando necessário (changelog, git blame, tags)
```

Fetch completo (`fetch-depth: 0`) em repositório grande é um dos custos de I/O mais fáceis de esquecer. Pesa toda vez que o job roda, não só na primeira vez. Reserve para os jobs que realmente precisam de histórico (ex.: gerar changelog, calcular versão semântica a partir de tags).

Em runner persistente, o workspace de um job pode ficar em disco entre execuções. Isso acelera builds incrementais, mas acumula lixo (`node_modules` antigos, artefatos de build, arquivos temporários de teste) se nada limpar. Avalie:

```text
runner efêmero          workspace descartado junto com o runner — sem acúmulo, mas sem cache de disco entre jobs
runner persistente       workspace sobrevive — ganho de I/O no build incremental, exige limpeza periódica (git clean -ffdx, ou clean: true no checkout)
```

`actions/checkout@v4` já limpa arquivos não rastreados do próprio checkout por padrão (`clean: true`), mas isso não remove diretórios fora do controle do Git (ex.: cache de build em path customizado). Essa limpeza é responsabilidade de um passo explícito ou de rotina de manutenção do runner (seção 37).

---

## 37. Cleanup automático

Runner persistente acumula:

```text
containers parados
imagens não usadas (dangling e não referenciadas)
volumes órfãos
cache de build do BuildKit (seção 7)
workspaces antigos de jobs
logs do runner
```

Automatize a limpeza (cron ou hook pós-job) em vez de depender de intervenção manual. É justamente o cenário que gera custo de I/O silencioso em disco compartilhado com outros produtos na mesma máquina.

---

## 38. Disk budget

Defina limiares de disco e a ação em cada um:

| Limiar | Ação |
|---|---|
| Alert 70% | Notificar, sem ação automática |
| Cleanup 75% | Disparar rotina de limpeza automática (containers/imagens/workspaces) |
| Critical 90% | Bloquear novos jobs até liberar espaço |

Valores são exemplos e devem ser ajustados ao tamanho real do disco e ao ritmo de crescimento observado. Monitore antes de fixar o número.

---

## 39. Docker build cache pruning

`docker buildx prune` (cache do BuildKit) e `docker system prune` (containers/imagens/redes/volumes não usados) limpam de forma controlada. Prefira flags explícitas a `--all` cego:

```bash
docker buildx prune --filter "until=168h"   # remove cache do BuildKit com mais de 7 dias
docker system prune -f                       # remove containers parados, redes e imagens dangling
```

Não apagar cache durante horário de maior uso sem necessidade. Um prune agressivo no meio do expediente invalida o cache que outros jobs concorrentes esperavam reaproveitar, gerando rebuilds completos e pico de I/O justamente no horário mais sensível.

---

## 40. Test duration budget

Defina metas por camada.

---

## 41. Slow tests

Gere ranking.

---

## 42. Split slow suites

Uma suíte com 1 teste de 10 minutos pode precisar redesign.

---

## 43. Mock external services

Na PR, substitua dependências lentas quando integração real não é o objetivo.

---

## 44. Local service containers

Redis/MQTT local em Docker é mais previsível que serviços remotos.

---

## 45. Network

Runner local pode ter vantagem de baixa latência para DEV interno.

---

## 46. E2E headless

CI normalmente usa browser headless.

Debug local pode usar headed.

---

## 47. Video/trace

Não gravar vídeo de todos os testes se armazenamento e I/O forem gargalos.

Configure em falha/retry.

---

## 48. Screenshot policy

Capture apenas quando útil.

---

## 49. Retry cost

Retries multiplicam tempo.

Corrija flakiness.

---

## 50. Test ordering

Testes mais rápidos ou de maior sinal podem rodar primeiro.

---

## 51. Changed tests first

Uma estratégia:

```text
affected tests
 |
 v
remaining smoke
```

---

## 52. PR labels para testes

Projetos maduros podem usar label:

```text
full-e2e
```

para exigir suíte ampliada.

Não permita bypass de testes críticos.

---

## 53. Manual full E2E

Disponibilize `workflow_dispatch`.

---

## 54. Nightly capacity

Suíte pesada pode usar horário fora do expediente.

---

## 55. Cost model

Self-hosted não é custo zero.

Inclua:

```text
hardware
energia
manutenção
backup
administração
```

---

## 56. Hosted versus self-hosted

Hosted:

```text
elasticidade + conveniência
```

Self-hosted:

```text
controle + custo fixo + administração
```

Modelo híbrido pode ser ideal.

---

## 57. Burst capacity

Jobs comuns self-hosted e picos em hosted podem ser estratégia, se orçamento permitir.

---

## 58. Security constraint

Não mova job privilegiado para qualquer runner apenas por velocidade.

---

## 59. Benchmark

Antes/depois:

```text
baseline 18 min
optimized 9 min
```

Registre resultado.

---

## 60. Evitar micro-otimização

Não gastar dias para economizar 5 segundos num job diário.

---

## 61. Developer experience

CI rápido melhora:

- frequência de PR;
- confiança;
- correção rápida.

---

## 62. Lead time

Meça:

```text
primeiro commit -> produção
```

---

## 63. Cycle time

Tempo de implementação/revisão.

CI é apenas componente.

---

## 64. Queue discipline

Prioridade de deploy PROD pode ser diferente de full nightly.

Runners especializados evitam bloqueio.

---

## 65. E2E runner separado

Evita:

```text
E2E de 30 min
bloquear lint urgente
```

---

## 66. Deploy runner separado

Deploy não espera fila de testes.

---

## 67. Observabilidade do CI

Dashboard:

```text
queue
duration
failure
runner utilization
```

---

## 68. Utilização

Runner 100% ocupado durante todo dia sugere expansão.

Runner 2% pode ser consolidado se segurança permitir.

---

## 69. Capacity headroom

Mantenha margem para picos.

---

## 70. Checklist otimização

- [ ] Baseline medido.
- [ ] Gargalo identificado.
- [ ] Cache correto.
- [ ] Concurrency cancel.
- [ ] Fail fast.
- [ ] Tests separados.
- [ ] E2E smoke/full.
- [ ] Flaky reduzido.
- [ ] Queue monitorada.
- [ ] Capacidade adequada.
- [ ] Resultado medido após mudança.

---

## 71. Roadmap de escala

```text
1 runner
 |
 v
labels
 |
 v
runner E2E
 |
 v
runner deploy
 |
 v
sharding
 |
 v
ephemeral/autoscaling
```

---

## 72. Próximo volume

**Volume 15 — Governança e Operação**

---

**Fim do Volume 14 — Otimização e Escalabilidade do CI/CD**

---

## Fontes

### Cache de dependências (npm, Composer)

- [Caching dependencies to speed up workflows](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows) — comprova o uso de `actions/cache@v4`, `hashFiles` para chave, `restore-keys` para fallback parcial, e os limites de 10 GB por repositório com eviction de entradas não acessadas há mais de 7 dias (seção 5).
- [actions/setup-node](https://github.com/actions/setup-node) — comprova o input `cache: 'npm'` (e yarn/pnpm) como alternativa mais simples ao `actions/cache` manual para cache de gerenciador de pacotes (seção 5).

### Docker layer cache e BuildKit

- [docker/build-push-action](https://github.com/docker/build-push-action) — comprova os inputs `cache-from`/`cache-to` usados com buildx no exemplo YAML (seção 7).
- [Docker Build cache backends](https://docs.docker.com/build/cache/backends/) — comprova os backends de cache `gha`, `registry` e `local`, e a diferença entre modo `min` (padrão) e `max` (todas as camadas intermediárias) citada na seção 7.
- [docker buildx prune CLI reference](https://docs.docker.com/reference/cli/docker/buildx/prune/) — comprova a sintaxe `docker buildx prune --filter "until=<duração>"` usada na seção 39.

### Checkout e workspace

- [actions/checkout](https://github.com/actions/checkout) — comprova o padrão `fetch-depth: 1` (raso), a opção `fetch-depth: 0` para histórico completo, e o comportamento padrão `clean: true` citados na seção 36-A.

### Concurrency

- [Using concurrency (GitHub Actions)](https://docs.github.com/en/actions/using-jobs/using-concurrency) — comprova a opção `cancel-in-progress: true` para cancelar execuções em andamento no mesmo grupo de concorrência (seção 11).

### Runners self-hosted e autoscaling

- [actions/actions-runner-controller](https://github.com/actions/actions-runner-controller) — comprova que o Actions Runner Controller (ARC) é a solução para autoscaling de runners self-hosted em Kubernetes, com o modo `RunnerScaleSet` (autoscaling runner scale sets) atual e suportado pela GitHub substituindo os modos legados (`RunnerDeployment`), e a natureza efêmera dos runners provisionados (seção 33).
