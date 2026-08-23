# Volume 13 — Arquiteturas de Referência

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 13_ARQUITETURAS_DE_REFERENCIA.md  
**Versão:** 0.1.0

---

## 1. Objetivo

Este volume apresenta arquiteturas práticas reutilizáveis para diferentes tipos de projeto.

---

# 2. Referência A — Node.js simples

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

# 3. Node.js + MariaDB

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

# 4. Node.js + Redis

Use Redis container em integração.

Teste:

- cache hit;
- cache miss;
- expiração;
- indisponibilidade controlada.

---

# 5. Node.js + MQTT

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

# 6. PHP + MariaDB

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

# 7. PHP legado sem Docker runtime

Mesmo que produção execute em Apache/PHP tradicional, CI pode usar containers para dependências.

Migração para Docker pode ser gradual.

---

# 8. PHP + MQTT

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

# 9. Frontend SPA + API

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

# 10. Frontend separado do backend

Repos separados:

```text
frontend repo
backend repo
```

Cada um possui CI.

Integração pode rodar em DEV ou pipeline coordenado.

---

# 11. Monorepo frontend/backend

```text
apps/
  frontend/
  api/
packages/
  common/
```

CI deve entender dependências compartilhadas.

---

# 12. Sistema com worker

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

# 13. Sistema com webhook

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

# 14. Sistema IoT

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

# 15. Aplicação com ESP32

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

# 16. Hardware-in-the-loop

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

# 17. Microsserviços

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

# 18. Contract testing em microsserviços

Reduz dependência de E2E gigantesco.

Valide contratos entre serviços.

---

# 19. Banco por serviço

Em arquitetura de microsserviços, compartilhamento de banco aumenta acoplamento.

Decisão depende do sistema.

---

# 20. Reverse proxy

```text
Nginx/Traefik
 |
 +-- frontend
 +-- API
```

Configuração versionada.

---

# 21. Traefik

Alternativa open source para roteamento dinâmico em ambientes containerizados.

Nginx continua excelente opção.

---

# 22. Registry

Todos os modelos containerizados convergem para:

```text
CI -> Registry -> environments
```

---

# 23. DEV compartilhado

Se apenas um DEV:

```text
main
 |
 v
deploy DEV
```

PRs são validadas principalmente no CI antes do merge.

---

# 24. Preview environment

Para maior paralelismo:

```text
PR #42 -> preview-42
PR #43 -> preview-43
```

Mais caro e complexo.

---

# 25. Sem staging

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

# 26. Banco DEV

Não deve conter única cópia de informação importante.

É ambiente reconstruível.

---

# 27. Banco PROD

Backup, restore, monitoring, access control.

---

# 28. Deploy Node.js com Compose

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

# 29. Deploy PHP com Compose

Possível stack:

```text
nginx
php-fpm
worker
```

Banco separado/persistente.

---

# 30. CI para PHP sem container final

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

# 31. Releases directory

Modelo tradicional:

```text
/var/www/app/releases/
  a91c302/
  b72e510/

current -> b72e510
```

Rollback troca symlink.

---

# 32. Atomic deploy tradicional

```text
upload release
install dependencies
validate
switch current
```

Evita editar diretório ativo arquivo por arquivo.

---

# 33. Arquitetura Docker preferencial

Para novos projetos:

```text
image imutável
```

simplifica artifact.

---

# 34. Observabilidade comum

Todas as arquiteturas devem oferecer:

```text
/health
/version
structured logs
metrics
```

---

# 35. Segurança comum

Todas:

- secrets externos;
- least privilege;
- branch protection;
- CI sem PROD;
- deploy protegido.

---

# 36. Template de projeto Node

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

# 37. Template PHP

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

# 38. Template IoT backend

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

# 39. Separar adapter MQTT

Encapsular broker facilita testes.

---

# 40. Separar repository

Encapsular banco facilita unit/integration testing.

---

# 41. Ports and adapters

Arquitetura hexagonal pode ser útil em sistemas com muitas integrações.

Não é obrigatória.

---

# 42. Referência mínima recomendada

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

# 43. Referência automação

```text
Node API
MariaDB
Mosquitto
WebSocket
Frontend
Docker
```

---

# 44. Testes dessa referência

```text
unit business
integration DB
integration MQTT
API
WebSocket integration
E2E browser
```

---

# 45. WebSocket

Teste:

- conexão;
- autenticação;
- evento;
- reconexão;
- disconnect.

---

# 46. Real-time E2E

Evite sleeps fixos.

Espere evento/estado.

---

# 47. Arquitetura de runners

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

# 48. Escolha por risco

Projeto simples não precisa de arquitetura de multinacional.

Use a menor arquitetura que mantenha:

```text
qualidade
segurança
rastreabilidade
rollback
```

---

# 49. Checklist de nova arquitetura

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

# 50. Próximo volume

**Volume 14 — Otimização e Escalabilidade do CI/CD**

---

**Fim do Volume 13 — Arquiteturas de Referência**
