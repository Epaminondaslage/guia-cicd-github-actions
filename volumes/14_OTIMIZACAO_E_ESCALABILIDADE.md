# Volume 14 — Otimização e Escalabilidade do CI/CD

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
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

# 2. Métricas fundamentais

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

# 3. Gargalo

Não assuma que E2E é sempre o maior problema.

Pode ser:

- `npm ci`;
- Docker build;
- fila;
- browser install;
- banco;
- checkout.

---

# 4. Pareto

Frequentemente 20% das etapas consomem 80% do tempo.

Ataque primeiro os maiores custos.

---

# 5. Cache npm

Use cache com lockfile.

Não cache `node_modules` cegamente sem necessidade.

---

# 6. Cache Composer

Cache de downloads pode reduzir instalação.

O lockfile continua fonte de versões.

---

# 7. Docker layer cache

Dockerfile bem estruturado:

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

é a primeira otimização.

---

# 8. Browser cache

Playwright baixa browsers grandes.

Em runner persistente, versões podem ser reaproveitadas quando compatíveis.

Gerencie espaço e versões.

---

# 9. Imagem de runner preparada

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

# 10. Runners efêmeros com imagem pronta

Combina:

```text
isolamento
+
setup rápido
```

Exige provisioning automatizado.

---

# 11. Concurrency cancel

```yaml
cancel-in-progress: true
```

economiza execução de commits antigos.

---

# 12. Fail fast

```text
lint -> unit -> expensive tests
```

Falhar cedo economiza recursos.

---

# 13. Paralelização

Com capacidade:

```text
lint
unit
integration
```

podem rodar em paralelo.

---

# 14. Dependência correta

Não coloque `needs` onde não existe dependência lógica.

Dependências artificiais aumentam tempo.

---

# 15. E2E shards

```text
1/4
2/4
3/4
4/4
```

Com quatro runners, reduz wall-clock.

---

# 16. Workers

Dentro do mesmo runner:

```bash
playwright test --workers=4
```

Ajuste por CPU/RAM.

---

# 17. Oversubscription

8 workers em máquina de 4 cores pode piorar tempo e flakiness.

---

# 18. Banco compartilhado

Paralelismo exige isolamento.

Use bancos/schemas/dados únicos.

---

# 19. Test impact

Mapeie mudança para testes.

---

# 20. Full regression como rede de segurança

Mesmo com seleção de testes:

```text
nightly full
```

permanece.

---

# 21. Path filters

Evite workflow frontend se apenas docs independentes mudaram.

---

# 22. Docs-only PR

Pode executar apenas:

```text
markdown lint
link check
```

quando seguro.

---

# 23. Monorepo graph

Ferramentas de monorepo podem calcular dependências.

Adote somente se projeto justificar.

---

# 24. Build seletivo

Não reconstruir serviços não afetados.

---

# 25. Artifact reuse

Não repetir:

```text
npm build
```

em jobs diferentes.

Produza artifact em um job e reutilize.

---

# 26. Upload/download artifact

Útil quando jobs estão em runners diferentes.

Compare custo de transferência com rebuild.

---

# 27. Workspace compartilhado

Em self-hosted persistente, não dependa implicitamente do workspace entre jobs.

Isso reduz portabilidade.

---

# 28. Runner count

Se queue time cresce:

```text
adicionar runner
```

pode ser melhor que otimizar testes.

---

# 29. Runner especializado

```text
ci
e2e
deploy
arm64
```

---

# 30. Scale-up

Mais CPU/RAM.

Bom quando job único é pesado.

---

# 31. Scale-out

Mais runners.

Bom quando muitos jobs ficam em fila.

---

# 32. Auto-scaling

Em ambiente cloud/virtualização, runners podem ser criados sob demanda.

É fase avançada.

---

# 33. Actions Runner Controller

Em ambientes Kubernetes, existem soluções de autoscaling de runners.

Só considerar quando escala justificar Kubernetes.

---

# 34. Não adotar Kubernetes cedo

Kubernetes pode adicionar mais complexidade que valor em projetos pequenos.

---

# 35. VM templates

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

# 36. Ephemeral registration

Runners descartáveis devem ser registrados de forma adequada para uso único conforme recursos do GitHub Runner.

---

# 37. Cleanup automático

Runner persistente:

```text
containers
images
workspaces
logs
```

precisa manutenção.

---

# 38. Disk budget

Defina:

```text
alert 70%
cleanup 75%
critical 90%
```

Valores são exemplos e devem ser ajustados.

---

# 39. Docker build cache pruning

Limpe de forma controlada.

Não apagar cache durante horário de maior uso sem necessidade.

---

# 40. Test duration budget

Defina metas por camada.

---

# 41. Slow tests

Gere ranking.

---

# 42. Split slow suites

Uma suíte com 1 teste de 10 minutos pode precisar redesign.

---

# 43. Mock external services

Na PR, substitua dependências lentas quando integração real não é o objetivo.

---

# 44. Local service containers

Redis/MQTT local em Docker é mais previsível que serviços remotos.

---

# 45. Network

Runner local pode ter vantagem de baixa latência para DEV interno.

---

# 46. E2E headless

CI normalmente usa browser headless.

Debug local pode usar headed.

---

# 47. Video/trace

Não gravar vídeo de todos os testes se armazenamento e I/O forem gargalos.

Configure em falha/retry.

---

# 48. Screenshot policy

Capture apenas quando útil.

---

# 49. Retry cost

Retries multiplicam tempo.

Corrija flakiness.

---

# 50. Test ordering

Testes mais rápidos ou de maior sinal podem rodar primeiro.

---

# 51. Changed tests first

Uma estratégia:

```text
affected tests
 |
 v
remaining smoke
```

---

# 52. PR labels para testes

Projetos maduros podem usar label:

```text
full-e2e
```

para exigir suíte ampliada.

Não permita bypass de testes críticos.

---

# 53. Manual full E2E

Disponibilize `workflow_dispatch`.

---

# 54. Nightly capacity

Suíte pesada pode usar horário fora do expediente.

---

# 55. Cost model

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

# 56. Hosted versus self-hosted

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

# 57. Burst capacity

Jobs comuns self-hosted e picos em hosted podem ser estratégia, se orçamento permitir.

---

# 58. Security constraint

Não mova job privilegiado para qualquer runner apenas por velocidade.

---

# 59. Benchmark

Antes/depois:

```text
baseline 18 min
optimized 9 min
```

Registre resultado.

---

# 60. Evitar micro-otimização

Não gastar dias para economizar 5 segundos num job diário.

---

# 61. Developer experience

CI rápido melhora:

- frequência de PR;
- confiança;
- correção rápida.

---

# 62. Lead time

Meça:

```text
primeiro commit -> produção
```

---

# 63. Cycle time

Tempo de implementação/revisão.

CI é apenas componente.

---

# 64. Queue discipline

Prioridade de deploy PROD pode ser diferente de full nightly.

Runners especializados evitam bloqueio.

---

# 65. E2E runner separado

Evita:

```text
E2E de 30 min
bloquear lint urgente
```

---

# 66. Deploy runner separado

Deploy não espera fila de testes.

---

# 67. Observabilidade do CI

Dashboard:

```text
queue
duration
failure
runner utilization
```

---

# 68. Utilização

Runner 100% ocupado durante todo dia sugere expansão.

Runner 2% pode ser consolidado se segurança permitir.

---

# 69. Capacity headroom

Mantenha margem para picos.

---

# 70. Checklist otimização

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

# 71. Roadmap de escala

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

# 72. Próximo volume

**Volume 15 — Governança e Operação**

---

**Fim do Volume 14 — Otimização e Escalabilidade do CI/CD**
