# Volume 11 — Observabilidade do Pipeline e das Aplicações

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 11_OBSERVABILIDADE.md  
**Versão:** 0.1.0

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

# 2. Monitoramento versus observabilidade

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

# 3. Pipeline observável

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

# 4. Aplicação observável

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

# 5. Logs estruturados

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

---

# 6. Correlação

Inclua identificadores:

```text
requestId
traceId
user/session technical id
job id
```

Sem expor dados pessoais desnecessários.

---

# 7. Níveis

```text
DEBUG
INFO
WARN
ERROR
```

Produção não deveria depender de DEBUG permanente para funcionar.

---

# 8. Logs Docker

Aplicação escreve:

```text
stdout/stderr
```

Docker coleta.

Depois podemos encaminhar para sistema central.

---

# 9. Loki

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

# 10. Promtail e agentes

O ecossistema de coleta evolui. Ao implementar, consulte componentes atualmente recomendados pelo projeto Grafana.

O conceito permanece:

```text
host logs -> collector -> Loki
```

---

# 11. Prometheus

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

# 12. Exporters

Exemplos:

```text
node exporter
cAdvisor
application /metrics
```

---

# 13. Node Exporter

Métricas do host:

- CPU;
- RAM;
- filesystem;
- network;
- load.

Útil para runner e servidores.

---

# 14. cAdvisor

Pode fornecer métricas de containers.

Avalie compatibilidade e segurança do ambiente.

---

# 15. Métricas da aplicação

Exemplos:

```text
http_requests_total
http_request_duration_seconds
active_connections
mqtt_messages_total
```

---

# 16. RED Method

Para serviços:

```text
Rate
Errors
Duration
```

---

# 17. USE Method

Para recursos:

```text
Utilization
Saturation
Errors
```

---

# 18. Golden Signals

```text
Latency
Traffic
Errors
Saturation
```

---

# 19. Grafana

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

# 20. Dashboard de runner

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

# 21. Disk alert

Self-hosted runner pode parar por disco cheio.

Alerta exemplo:

```text
filesystem > 80%
```

O limite real depende da capacidade.

---

# 22. Runner offline

Crie verificação para detectar indisponibilidade.

---

# 23. Health checks

Endpoint:

```text
/health
```

deve responder rapidamente.

---

# 24. Readiness

```text
/ready
```

pode indicar capacidade de receber tráfego.

---

# 25. Version endpoint

```text
/version
```

ajuda correlação com deploy.

---

# 26. Deployment marker

Ao fazer deploy, registre evento:

```text
v2.3.1
14:32
```

No dashboard, uma linha vertical pode ajudar a correlacionar aumento de erros.

---

# 27. Alertas

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

# 28. Alert fatigue

Muitos alertas inúteis fazem a equipe ignorar todos.

---

# 29. SLI

Service Level Indicator.

Exemplo:

```text
percentual de requests bem-sucedidas
```

---

# 30. SLO

Service Level Objective.

Exemplo:

```text
99.9% de disponibilidade mensal
```

---

# 31. SLA

Acordo externo/contratual.

Não confunda com SLO interno.

---

# 32. Error budget

Se SLO permite 0,1% de falha, isso define orçamento.

Pode orientar ritmo de mudanças.

---

# 33. Latência

Use percentis:

```text
p50
p95
p99
```

Média pode esconder caudas.

---

# 34. Banco

Observe:

- conexões;
- queries lentas;
- locks;
- disk;
- buffer/cache;
- erros.

---

# 35. MQTT

Observe:

- conexões;
- mensagens;
- disconnects;
- retained;
- filas;
- erros.

---

# 36. Logs de deploy

Registre:

```text
version
actor
start
end
result
```

---

# 37. Logs de CI

GitHub já possui logs de Actions.

Para análise de longo prazo, métricas agregadas podem ser úteis.

---

# 38. Test duration

Registre tendências.

```text
E2E:
5 min -> 8 -> 12 -> 20
```

Isso sinaliza necessidade de otimização.

---

# 39. Flaky rate

Métrica:

```text
testes que passam após retry / total
```

---

# 40. Mean Time To Detect

MTTD:

```text
quanto tempo até perceber falha
```

---

# 41. Mean Time To Recover

MTTR:

```text
quanto tempo até restaurar serviço
```

CI/CD e rollback devem reduzir MTTR.

---

# 42. Synthetic monitoring

Execute fluxos seguros periodicamente.

```text
login técnico
consulta
logout
```

---

# 43. Uptime check

Teste HTTP simples de fora da infraestrutura.

Ajuda detectar falha de rede/proxy.

---

# 44. Black-box versus white-box

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

# 45. Alertas por deploy

Se erro cresce imediatamente após deploy, correlacione.

Uma evolução é automatizar rollback para falhas muito claras, com cautela.

---

# 46. Logs sensíveis

Não grave:

- passwords;
- tokens;
- cookies completos;
- dados pessoais desnecessários.

---

# 47. Retenção

Defina quanto tempo guardar:

```text
CI logs
app logs
security logs
```

com base em custo e necessidade.

---

# 48. Backup do observability stack

Dashboards e configurações também devem ser versionados quando possível.

---

# 49. Grafana provisioning

Dashboards/datasources podem ser provisionados por arquivos versionados.

---

# 50. Prometheus configuration

Versione:

```text
prometheus.yml
alert rules
```

sem secrets.

---

# 51. Alertmanager

Prometheus Alertmanager organiza e encaminha alertas.

Pode agrupar, silenciar e rotear.

---

# 52. Silences

Durante manutenção planejada, use silences controlados.

Não desligue monitoramento indefinidamente.

---

# 53. Runbooks

Alerta deve apontar para procedimento.

Exemplo:

```text
RunnerDiskLow
 -> runbook docs/runbooks/runner-disk.md
```

---

# 54. Runbook

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

# 55. Dashboard DEV

Mostrar:

- versão;
- health;
- errors;
- latency;
- containers.

---

# 56. Dashboard PROD

Priorize sinais operacionais, não métricas decorativas.

---

# 57. CI dashboard

Exemplo:

```text
PR lead time
build duration
queue duration
success rate
E2E duration
```

---

# 58. Observabilidade no post-mortem

Use dados objetivos para reconstruir timeline.

---

# 59. Tracing

OpenTelemetry pode instrumentar aplicações distribuídas.

Fluxo:

```text
frontend/API
 |
 trace
 |
 services
 |
 database
```

---

# 60. OpenTelemetry

Projeto open source para métricas, logs e traces padronizados.

É uma evolução recomendada para sistemas distribuídos.

---

# 61. Não instrumentar tudo de uma vez

Comece por:

```text
logs estruturados
host metrics
application RED
deployment markers
```

Depois tracing.

---

# 62. Stack inicial sugerida

```text
Prometheus
Grafana
Loki
Node Exporter
```

Adicionar outros componentes conforme necessidade.

---

# 63. Docker Compose observability

Pode existir stack separado:

```text
compose.observability.yml
```

Não misture volumes de aplicação com monitoring sem planejamento.

---

# 64. Segurança

Grafana/Prometheus não devem ficar expostos publicamente sem autenticação e proteção de rede.

---

# 65. Checklist

- [ ] Logs estruturados.
- [ ] Sem secrets.
- [ ] Health.
- [ ] Version.
- [ ] Host metrics.
- [ ] Disk alert.
- [ ] App errors.
- [ ] Latency.
- [ ] Deploy markers.
- [ ] Runbooks.
- [ ] Retenção definida.

---

# 66. Próximo volume

**Volume 12 — Infraestrutura do Servidor CI/CD**

Cobrirá host, VM, Ubuntu, SSH, firewall, storage, rede, backup e disponibilidade.

---

**Fim do Volume 11 — Observabilidade**
