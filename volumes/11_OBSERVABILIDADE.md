# Volume 11 — Observabilidade do Pipeline e das Aplicações

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 11_OBSERVABILIDADE.md  
**Versão:** 0.2.0

---

## 1. Objetivo

Observabilidade responde:

```text
o sistema está saudável?
se não, por quê?
o problema começou após qual deploy?
```

Três pilares tradicionais:

```text
Logs
Métricas
Traces
```

Neste guia, começaremos por logs e métricas.

---

## 2. Monitoramento versus observabilidade

Monitoramento:

```text
CPU > 90%
```

Observabilidade:

```text
qual serviço?
desde quando?
qual deploy?
qual endpoint?
qual dependência?
```

---

## 3. Pipeline observável

Acompanhar:

- duração de workflows;
- tempo em fila;
- falhas;
- flaky tests;
- runner offline;
- consumo de disco;
- deploys;
- rollbacks.

---

## 4. Aplicação observável

Acompanhar:

- disponibilidade;
- latência;
- taxa de erros;
- throughput;
- CPU;
- memória;
- banco;
- filas;
- MQTT.

---

## 5. Logs estruturados

Prefira:

```json
{
  "level": "error",
  "service": "api",
  "requestId": "abc123",
  "message": "database timeout"
}
```

a logs livres difíceis de consultar.

Campos recomendados em cada linha de log:

| Campo | Descrição |
|---|---|
| `timestamp` | ISO 8601, UTC |
| `level` | Nível do log |
| `service` | Serviço que emitiu o log |
| `version` | SHA curto ou tag do deploy |
| `environment` | dev/staging/prod |
| `requestId` ou `traceId` | Identificador de correlação |
| `message` | Mensagem do evento |

O formato JSON por linha (NDJSON — um objeto por linha) é o mais fácil de indexar em Loki, Elasticsearch/OpenSearch, CloudWatch Logs Insights ou qualquer coletor. Emitir `console.log` de string livre funciona para debug local, mas não escala para consulta e correlação em produção.

Bibliotecas comuns para logging estruturado:

| Linguagem | Bibliotecas |
|---|---|
| Node.js | pino, winston (formatter JSON) |
| Python | structlog, logging com JSONFormatter |
| Go | slog (stdlib, Go 1.21+), zap, zerolog |
| Java | Logback/Log4j2 com encoder JSON |

---

## 6. Correlação entre pipeline e aplicação

O objetivo prático é responder "esse erro em produção apareceu depois de qual execução do workflow?". Para isso, o identificador do deploy precisa atravessar a fronteira entre CI/CD e aplicação.

Inclua identificadores nos logs da aplicação:

| Identificador | Descrição |
|---|---|
| `requestId` ou `traceId` | Por requisição |
| `deployId` / `releaseVersion` | SHA do commit ou tag |
| Job id / run id do GitHub Actions | Opcional, para depuração |
| User/session technical id | Identificação técnica do usuário/sessão |

Como propagar o identificador do pipeline para dentro da aplicação:

```yaml
- name: Build com metadata de versão
  run: |
    docker build \
      --build-arg GIT_SHA=${{ github.sha }} \
      --build-arg RUN_ID=${{ github.run_id }} \
      -t minha-app:${{ github.sha }} .
```

A aplicação lê essas variáveis em build time (ou via env var em runtime) e as inclui em todo log estruturado e no endpoint `/version` (ver seção 25). Assim, ao ver um erro em produção, basta olhar o campo `version` do log e localizar o run correspondente em `Actions` no GitHub pelo SHA.

Sem expor dados pessoais desnecessários (PII) nos logs — nem em nome de "correlação".

---

## 7. Níveis

```text
DEBUG
INFO
WARN
ERROR
```

Produção não deveria depender de DEBUG permanente para funcionar.

---

## 8. Logs Docker

Aplicação escreve:

```text
stdout/stderr
```

Docker coleta.

Depois podemos encaminhar para sistema central.

---

## 9. Loki

Grafana Loki é opção open source para agregação de logs.

Arquitetura:

```text
containers
 |
 v
collector
 |
 v
Loki
 |
 v
Grafana
```

---

## 10. Agentes de coleta de logs

O ecossistema de coleta evolui rápido. O Promtail (agente clássico do Loki) entrou em modo de manutenção, e o Grafana recomenda o Grafana Alloy como substituto para novas instalações — Alloy é um coletor único, compatível com OpenTelemetry, que também sabe enviar dados para Loki, Prometheus e Tempo.

Ao implementar, consulte a documentação atual do projeto Grafana antes de escolher o agente — o nome do componente recomendado muda com mais frequência do que o conceito por trás dele.

O conceito permanece:

```text
host/container logs -> agente de coleta -> Loki
```

---

## 11. Prometheus

Prometheus coleta métricas.

Arquitetura:

```text
targets
 |
 v
Prometheus
 |
 v
Grafana
```

---

## 12. Exporters

Exemplos:

```text
node exporter
cAdvisor
application /metrics
```

---

## 13. Node Exporter

Métricas do host:

- CPU;
- RAM;
- filesystem;
- network;
- load.

Útil para runner e servidores.

---

## 14. cAdvisor

Pode fornecer métricas de containers.

Avalie compatibilidade e segurança do ambiente.

---

## 15. Métricas da aplicação

Exemplos:

```text
http_requests_total
http_request_duration_seconds
active_connections
mqtt_messages_total
```

---

## 16. RED method

Para serviços:

```text
Rate
Errors
Duration
```

---

## 17. USE method

Para recursos:

```text
Utilization
Saturation
Errors
```

---

## 18. Golden signals

```text
Latency
Traffic
Errors
Saturation
```

---

## 19. Grafana

Crie dashboards para:

```text
CI runners
DEV
PROD
application
database
MQTT
```

Evite dashboard com centenas de gráficos sem objetivo.

---

## 20. Dashboard de runner

Painéis:

```text
CPU
RAM
disk
load
Docker disk
runner status
```

---

## 21. Disk alert

Self-hosted runner pode parar por disco cheio.

Alerta exemplo:

```text
filesystem > 80%
```

O limite real depende da capacidade.

---

## 22. Runner offline

Crie verificação para detectar indisponibilidade.

---

## 23. Health checks

Endpoint:

```text
/health
```

deve responder rapidamente.

---

## 24. Readiness

```text
/ready
```

pode indicar capacidade de receber tráfego.

---

## 25. Version endpoint

```text
/version
```

ajuda correlação com deploy.

---

## 26. Deployment marker

Ao fazer deploy, registre evento:

```text
v2.3.1
14:32
```

No dashboard, uma linha vertical pode ajudar a correlacionar aumento de erros.

---

## 27. Alertas

Um alerta deve ser:

```text
acionável
```

Ruim:

```text
CPU 51%
```

Bom:

```text
erro HTTP > 5% por 10 minutos
```

quando isso realmente requer ação.

---

## 27a. Notificações de falha de pipeline e deploy

Além de alertas de infraestrutura (Prometheus/Alertmanager), o pipeline deve notificar diretamente quando falha, sem depender de alguém checar a aba Actions manualmente.

Opções mais comuns:

| Canal | Descrição |
|---|---|
| E-mail | Notificação padrão do GitHub para quem disparou o workflow (ativado por padrão em Settings -> Notifications) |
| Slack | Webhook de entrada + step dedicado no job |
| Microsoft Teams | Webhook de entrada, formato similar ao Slack |
| GitHub | Status check + comentário automático no PR |

Exemplo de step em Slack usando um webhook, disparado apenas em falha:

```yaml
- name: Notificar falha no Slack
  if: failure()
  uses: slackapi/slack-github-action@v2
  with:
    webhook: ${{ secrets.SLACK_WEBHOOK_URL }}
    webhook-type: incoming-webhook
    payload: |
      {
        "text": "Falha no deploy de ${{ github.repository }} (${{ github.ref_name }}) — run ${{ github.run_id }}"
      }
```

Pontos de atenção:

- não commite a URL do webhook — use secrets;
- notifique falha, não sucesso repetitivo (evita ruído — ver Alert fatigue);
- inclua link direto para o run (github.server_url/github.repository/actions/runs/github.run_id);
- diferencie canal de falha de CI (build/test) do canal de falha de deploy em produção — severidades diferentes.

Para falhas de deploy especificamente, é comum um canal com resposta mais rápida (ex.: canal de on-call) separado do canal de CI geral.

---

## 28. Alert fatigue

Muitos alertas inúteis fazem a equipe ignorar todos.

---

## 29. SLI

Service Level Indicator.

Exemplo:

```text
percentual de requests bem-sucedidas
```

---

## 30. SLO

Service Level Objective.

Exemplo:

```text
99.9% de disponibilidade mensal
```

---

## 31. SLA

Acordo externo/contratual.

Não confunda com SLO interno.

---

## 32. Error budget

Se SLO permite 0,1% de falha, isso define orçamento.

Pode orientar ritmo de mudanças.

---

## 33. Latência

Use percentis:

```text
p50
p95
p99
```

Média pode esconder caudas.

---

## 34. Banco

Observe:

- conexões;
- queries lentas;
- locks;
- disk;
- buffer/cache;
- erros.

---

## 35. MQTT

Observe:

- conexões;
- mensagens;
- disconnects;
- retained;
- filas;
- erros.

---

## 36. Logs de deploy

Registre:

```text
version
actor
start
end
result
```

---

## 37. Logs de CI

GitHub já possui logs de Actions, disponíveis pela UI, pelo `gh run view --log` (GitHub CLI) e pela API REST/GraphQL.

Para análise de longo prazo, métricas agregadas podem ser úteis, já que os logs brutos expiram conforme a retenção configurada (padrão 90 dias em repositórios públicos/privados, ajustável em Settings → Actions).

---

## 37a. GitHub Actions: métricas via API e insights nativos

O próprio GitHub expõe dados de execução que dispensam, em boa parte dos casos, montar uma stack própria só para observar o pipeline:

```text
Aba "Insights -> Actions" do repositório
  - duração de workflows ao longo do tempo
  - taxa de sucesso/falha por workflow
  - uso de minutos por runner (hosted)

API REST
  GET /repos/{owner}/{repo}/actions/runs
  GET /repos/{owner}/{repo}/actions/runs/{run_id}/timing
  GET /repos/{owner}/{repo}/actions/workflows/{workflow_id}/runs

API GraphQL
  campos de CheckRun/WorkflowRun com startedAt/completedAt/conclusion
```

Métricas de pipeline que valem a pena extrair e acompanhar ao longo do tempo:

```text
duração total do workflow (created_at -> updated_at)
duração por job e por step (útil para achar o gargalo real)
tempo em fila até um runner pegar o job (queue time)
taxa de falha por workflow/branch
tempo até deploy (do commit/merge até o job de deploy concluir com sucesso)
taxa de uso de retry / re-run manual
```

Exemplo de script simples para extrair duração de jobs via `gh` e a API:

```bash
gh api repos/OWNER/REPO/actions/runs/RUN_ID/jobs \
  --jq '.jobs[] | {name, started_at, completed_at, conclusion}'
```

Para série histórica (dashboard próprio), um job agendado pode consultar a API periodicamente e gravar o resultado em um banco de métricas (Prometheus via Pushgateway, um banco de séries temporais, ou até uma tabela simples), alimentando o "CI dashboard" da seção 57.

Ferramentas de terceiros (avalie antes de adotar) que já fazem essa coleta:

```text
GitHub Actions exporter para Prometheus (comunidade)
Datadog CI Visibility
Grafana Cloud + integração GitHub
```

Comece pelos Insights nativos do GitHub; monte coleta própria só quando precisar cruzar esses dados com métricas de aplicação/infra no mesmo dashboard.

---

## 38. Test duration

Registre tendências.

```text
E2E:
5 min -> 8 -> 12 -> 20
```

Isso sinaliza necessidade de otimização.

---

## 39. Flaky rate

Métrica:

```text
testes que passam após retry / total
```

---

## 40. Mean time to detect

MTTD:

```text
quanto tempo até perceber falha
```

---

## 41. Mean time to recover

MTTR:

```text
quanto tempo até restaurar serviço
```

CI/CD e rollback devem reduzir MTTR.

---

## 42. Synthetic monitoring

Execute fluxos seguros periodicamente.

```text
login técnico
consulta
logout
```

---

## 43. Uptime check

Teste HTTP simples de fora da infraestrutura.

Ajuda detectar falha de rede/proxy.

---

## 44. Black-box versus white-box

Black-box:

```text
usuário consegue acessar?
```

White-box:

```text
CPU, logs, internals
```

Use ambos.

---

## 45. Alertas por deploy

Se erro cresce imediatamente após deploy, correlacione.

Uma evolução é automatizar rollback para falhas muito claras, com cautela.

---

## 46. Logs sensíveis

Não grave:

- passwords;
- tokens;
- cookies completos;
- dados pessoais desnecessários.

---

## 47. Retenção

Defina quanto tempo guardar:

```text
CI logs
app logs
security logs
```

com base em custo e necessidade.

---

## 48. Backup do observability stack

Dashboards e configurações também devem ser versionados quando possível.

---

## 49. Grafana provisioning

Dashboards/datasources podem ser provisionados por arquivos versionados.

---

## 50. Prometheus configuration

Versione:

```text
prometheus.yml
alert rules
```

sem secrets.

---

## 51. Alertmanager

Prometheus Alertmanager organiza e encaminha alertas.

Pode agrupar, silenciar e rotear.

---

## 52. Silences

Durante manutenção planejada, use silences controlados.

Não desligue monitoramento indefinidamente.

---

## 53. Runbooks

Alerta deve apontar para procedimento.

Exemplo:

```text
RunnerDiskLow
 -> runbook docs/runbooks/runner-disk.md
```

---

## 54. Runbook

Conteúdo:

```text
sintoma
impacto
diagnóstico
comandos
ação
rollback
escalonamento
```

---

## 55. Dashboard DEV

Mostrar:

- versão;
- health;
- errors;
- latency;
- containers.

---

## 56. Dashboard PROD

Priorize sinais operacionais, não métricas decorativas.

---

## 57. CI dashboard

Exemplo:

```text
PR lead time
build duration
queue duration
success rate
E2E duration
```

---

## 58. Observabilidade no post-mortem

Use dados objetivos para reconstruir timeline.

---

## 59. Tracing

Tracing distribuído mostra o caminho completo de uma requisição atravessando múltiplos serviços, com o tempo gasto em cada etapa (chamado de "span").

Faz sentido adotar tracing quando existe mais de um serviço/processo envolvido numa mesma requisição (frontend -> API -> serviço interno -> banco/fila). Para uma aplicação monolítica simples, logs estruturados com bom RED (seção 16) e métricas já cobrem boa parte da necessidade — tracing tem custo de instrumentação e de armazenamento que só compensa a partir de certa complexidade.

Fluxo típico:

```text
frontend/API
 |
 trace (traceId propagado via header, ex.: traceparent)
 |
 services
 |
 database/fila
```

Cada span carrega o mesmo `traceId`, permitindo reconstruir a linha do tempo completa de uma requisição em ferramentas como Grafana Tempo, Jaeger ou Zipkin.

---

## 60. OpenTelemetry

OpenTelemetry (OTel) é o projeto open source (CNCF) que padronizou coleta de métricas, logs e traces através de uma API e um formato comuns, independentes de fornecedor. Hoje é o padrão de facto — a maioria dos backends de observabilidade (Grafana, Datadog, New Relic, Honeycomb, etc.) aceita dados no formato OTLP (OpenTelemetry Protocol) nativamente.

Componentes principais:

```text
SDKs de instrumentação (por linguagem: Node.js, Python, Go, Java, .NET, ...)
OpenTelemetry Collector (recebe, processa e exporta os dados)
Backends (Grafana Tempo/Loki/Mimir, Jaeger, Prometheus, um APM comercial)
```

Um Collector típico numa stack self-hosted:

```text
aplicação (SDK OTel)
 |
 OTLP (gRPC/HTTP)
 |
 OpenTelemetry Collector
 |
 exporta para: Tempo (traces) / Prometheus (métricas) / Loki (logs)
```

Muitas linguagens oferecem instrumentação automática (auto-instrumentation) que captura chamadas HTTP, queries de banco e chamadas a filas sem alterar o código da aplicação — bom ponto de partida antes de instrumentar manualmente spans customizados.

---

## 60a. Correlação entre deploy e tracing

O mesmo identificador de versão usado nos logs (seção 6) deve aparecer também nos traces, como atributo do span (`service.version` ou `deployment.environment` no padrão de atributos semânticos do OpenTelemetry). Isso permite, ao investigar um span lento ou com erro, saber imediatamente em qual deploy ele ocorreu — fechando o ciclo pipeline -> deploy -> log -> métrica -> trace com o mesmo identificador em todas as camadas.

---

## 61. Não instrumentar tudo de uma vez

Comece por:

```text
logs estruturados
host metrics
application RED
deployment markers
```

Depois tracing.

---

## 62. Stack inicial sugerida

```text
Prometheus
Grafana
Loki
Node Exporter
```

Adicionar outros componentes conforme necessidade.

---

## 63. Docker Compose observability

Pode existir stack separado:

```text
compose.observability.yml
```

Não misture volumes de aplicação com monitoring sem planejamento.

---

## 64. Segurança

Grafana/Prometheus não devem ficar expostos publicamente sem autenticação e proteção de rede.

---

## 65. Checklist

- [ ] Logs estruturados (JSON) com versão/deploy no payload.
- [ ] Sem secrets nos logs.
- [ ] Health.
- [ ] Version.
- [ ] Host metrics.
- [ ] Disk alert.
- [ ] App errors.
- [ ] Latency.
- [ ] Deploy markers.
- [ ] Notificação de falha de pipeline/deploy (Slack/e-mail).
- [ ] Métricas de pipeline (duração de jobs, tempo até deploy, taxa de falha).
- [ ] Runbooks.
- [ ] Retenção definida.

---

## 66. Próximo volume

**Volume 12 — Infraestrutura do Servidor CI/CD**

Cobrirá host, VM, Ubuntu, SSH, firewall, storage, rede, backup e disponibilidade.

---

## Fontes

### Logs (Loki, Promtail/Alloy)

- [Grafana Loki documentation](https://grafana.com/docs/loki/latest/) — confirma Loki como stack de logs open source referenciada na seção 9.
- [Grafana Alloy documentation](https://grafana.com/docs/alloy/latest/) — confirma Alloy como coletor unificado (compatível com OpenTelemetry, Prometheus, Loki e Tempo) citado na seção 10.
- [Promtail documentation](https://grafana.com/docs/loki/latest/send-data/promtail/) — confirma que Promtail está em fim de vida (EOL em 02/03/2026) e que a migração recomendada é para o Alloy, citado na seção 10.

### Métricas (Prometheus, exporters, Alertmanager)

- [Prometheus overview](https://prometheus.io/docs/introduction/overview/) — confirma Prometheus como toolkit de monitoramento e alerta usado na seção 11.
- [prometheus/node_exporter (GitHub)](https://github.com/prometheus/node_exporter) — confirma o Node Exporter como exporter oficial de métricas de host (CPU, RAM, filesystem, network), citado na seção 13.
- [google/cadvisor (GitHub)](https://github.com/google/cadvisor) — confirma cAdvisor como ferramenta oficial do Google para métricas de containers, citado na seção 14.
- [Prometheus Alertmanager documentation](https://prometheus.io/docs/alerting/latest/alertmanager/) — confirma o papel de deduplicação, agrupamento e roteamento de alertas descrito na seção 51.

### Grafana

- [About Grafana](https://grafana.com/docs/grafana/latest/introduction/) — confirma Grafana como ferramenta de visualização/dashboard usada nas seções 19, 49 e 55-57.

### Tracing e OpenTelemetry

- [Grafana Tempo documentation](https://grafana.com/docs/tempo/latest/) — confirma Tempo como backend de tracing distribuído open source citado nas seções 59 e 60.
- [What is OpenTelemetry?](https://opentelemetry.io/docs/what-is-opentelemetry/) — confirma a definição de OpenTelemetry como framework de observabilidade vendor-neutral usado na seção 60.
- [OpenTelemetry Collector documentation](https://opentelemetry.io/docs/collector/) — confirma o Collector como componente que recebe, processa e exporta telemetria, citado na seção 60.
- [Resource semantic conventions — OpenTelemetry](https://opentelemetry.io/docs/specs/semconv/resource/) — confirma o atributo `service.version` usado na seção 60a.
- [Deployment environment — OpenTelemetry semantic conventions](https://opentelemetry.io/docs/specs/semconv/resource/deployment-environment/) — confirma o atributo `deployment.environment.name` usado na seção 60a.

### Métricas de pipeline via GitHub API

- [About GitHub Actions metrics](https://docs.github.com/en/actions/administering-github-actions/viewing-github-actions-metrics) — confirma o acesso via aba Insights (Actions Usage/Performance Metrics) citado na seção 37a.
- [REST API endpoints for GitHub Actions workflow runs](https://docs.github.com/en/rest/actions/workflow-runs) — confirma os endpoints `GET /repos/{owner}/{repo}/actions/runs` e `.../runs/{run_id}/timing` usados na seção 37a.
- [REST API endpoints for GitHub Actions workflow jobs](https://docs.github.com/en/rest/actions/workflow-jobs) — confirma o endpoint de listagem de jobs com campos de timing (`started_at`, `completed_at`) usado no exemplo de script da seção 37a.

### Notificação de falhas (Slack)

- [slackapi/slack-github-action (GitHub)](https://github.com/slackapi/slack-github-action) — confirma a action oficial e o suporte a incoming webhook usado no exemplo da seção 27a.

---

**Fim do Volume 11 — Observabilidade**
