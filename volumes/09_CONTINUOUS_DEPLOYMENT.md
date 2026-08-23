# Volume 09 — Continuous Deployment: DEV, Aprovação e Produção

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 09_CONTINUOUS_DEPLOYMENT.md  
**Versão:** 0.1.0  
**Pré-requisitos:** Volumes 01 a 08

---

## 1. Objetivo

Implementar o fluxo operacional:

```text
PR
 |
 v
CI
 |
 v
Merge
 |
 v
Artifact imutável
 |
 v
Deploy DEV
 |
 v
Validação
 |
 v
Gate humano
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

O fluxo assume que não existe staging separado. DEV é o ambiente de validação antes da produção.

---

## 2. Continuous Delivery versus Deployment

Continuous Delivery:

```text
software está sempre pronto para deploy
```

Continuous Deployment:

```text
software aprovado é implantado automaticamente
```

Neste guia, produção mantém gate humano.

Isso não torna Continuous Deployment automático "errado" — é uma opção igualmente válida, adotada por muitos times maduros. A diferença é o risco que cada modelo aceita:

```text
Deploy com gate manual
  humano aprova cada ida para PROD
  mais lento, mais controle por evento

Deploy automático (CD puro)
  merge aprovado já implica produção
  mais rápido, exige suíte de testes e observabilidade muito confiáveis
```

Se optar por automático, o GitHub exige que a proteção não dependa de disciplina humana informal. Configure o *environment* `production` com **required reviewers**:

```text
Settings -> Environments -> production
  Required reviewers: 1+ pessoas
```

Com isso, mesmo um workflow disparado automaticamente por push em `main` **pausa** antes de rodar o job com `environment: production`, aguardando aprovação na própria interface do GitHub. Ou seja: "CD automático" e "gate humano" não são mutuamente exclusivos — required reviewers é a forma do GitHub aplicar gate humano dentro de um pipeline desenhado como automático. A escolha real é onde a aprovação vive:

```text
gate via workflow_dispatch manual   -> operador decide quando disparar
gate via required reviewers         -> pipeline dispara sozinho, mas pausa para aprovação
sem nenhum dos dois                 -> deploy automático "puro", sem freio humano
```

Este guia usa `workflow_dispatch` para produção (Seção 21) porque é explícito e fácil de auditar, mas o mesmo resultado de segurança pode ser obtido com push automático + required reviewers no environment.

---

## 3. Build versus Deploy

Build:

```text
código -> artifact
```

Deploy:

```text
artifact -> ambiente
```

Não misture os conceitos.

---

## 4. Artifact imutável

Exemplo Docker:

```text
app:sha-a91c302
```

A mesma imagem deve seguir:

```text
DEV -> PROD
```

---

## 5. Build oficial

Após merge:

```text
main
 |
 v
commit SHA
 |
 v
build
 |
 v
registry
```

A imagem oficial deve estar ligada ao commit integrado.

---

## 6. GitHub Environment

Defina:

```text
development
production
```

Cada environment pode possuir:

- secrets;
- variables;
- regras de proteção;
- histórico de deploy.

---

## 7. Secrets por ambiente

```text
development
  DEV_HOST
  DEV_SSH_KEY

production
  PROD_HOST
  PROD_SSH_KEY
```

Não reutilize credencial administrativa global sem necessidade.

---

## 8. Deploy DEV automático

Evento:

```yaml
on:
  push:
    branches:
      - main
```

Depois do merge:

```text
main atualizado
 |
 v
deploy DEV
```

---

## 9. Workflow DEV conceitual

```yaml
name: Deploy DEV

on:
  push:
    branches:
      - main

jobs:
  deploy:
    environment:
      name: development

    runs-on:
      - self-hosted
      - linux
      - deploy

    steps:
      - uses: actions/checkout@v4

      - name: Deploy
        run: ./scripts/deploy/deploy-dev.sh "${{ github.sha }}"

      - name: Health
        run: ./scripts/deploy/health-check.sh
```

---

## 10. Deploy via SSH

Arquitetura:

```text
deploy runner
 |
 | SSH
 v
DEV server
```

Use:

- chave dedicada;
- usuário dedicado;
- privilégios mínimos;
- host key verification.

---

## 11. Evitar senha em comando

Não:

```bash
sshpass -p senha ...
```

Prefira chaves e secret management.

---

## 12. Usuário de deploy

Exemplo:

```text
deploy
```

Esse usuário deve ter apenas o necessário para:

- atualizar aplicação;
- operar Compose específico;
- consultar logs necessários.

---

## 13. Docker deploy

Servidor recebe referência:

```text
APP_VERSION=sha-a91c302
```

Depois:

```bash
docker compose pull app
docker compose up -d app
```

---

## 14. Health check

Após atualização:

```text
container up
 |
 v
readiness
 |
 v
HTTP /health
```

O deploy só deve ser considerado concluído após validação.

---

## 15. Retry de health

```bash
for i in $(seq 1 30); do
  curl --fail http://localhost/health && exit 0
  sleep 2
done
exit 1
```

Ajuste ao comportamento real.

---

## 16. Smoke DEV

Depois do health:

```text
login
consulta principal
ação segura
```

Pode ser automatizado por Playwright.

---

## 17. Validação visual

Como não existe staging:

```text
DEV
 |
 v
humano revisa frontend
```

Essa etapa é particularmente importante para mudanças visuais.

---

## 18. Gate humano

Produção só inicia quando existe autorização explícita.

```text
DEV PASS
 |
 v
Approval
 |
 +-- reject
 |
 +-- approve -> PROD
```

---

## 19. Environment production

Job:

```yaml
environment:
  name: production
```

Configure protection rules disponíveis para a conta/repositório.

Na prática, o recurso mais importante do environment `production` é:

```text
Required reviewers
```

Sem reviewers configurados, `environment: production` no YAML é só um rótulo — não bloqueia nada sozinho. É a combinação `environment de proteção com reviewers` + `job aponta para esse environment` que efetivamente impede o job de rodar sem aprovação, mesmo em planos gratuitos de repositório público (em repositórios privados, esse recurso exige GitHub Team/Enterprise).

---

## 20. Workflow PROD

Pode ser separado:

```text
deploy-prod.yml
```

Isso reduz risco de lógica confusa.

---

## 21. workflow_dispatch para PROD

Uma alternativa clara:

```yaml
on:
  workflow_dispatch:
    inputs:
      version:
        required: true
```

O operador seleciona a versão validada.

---

## 22. Não buildar novamente

PROD recebe:

```text
sha-a91c302
```

e não:

```text
docker build .
```

no servidor.

---

## 23. Registro de versão

Mantenha:

```text
current_version
previous_version
```

para rollback.

---

## 23a. Estratégia de rollback

Rollback rápido depende de decisões tomadas *antes* do incidente, não durante ele:

```text
1. Toda imagem publicada é versionada e imutável (Seção 4).
   app:sha-a91c302, nunca app:latest em PROD.

2. A versão anterior continua disponível no registry
   e no host (imagem já puxada não é removida agressivamente).

3. "Reverter" é trocar a tag ativa e reiniciar o serviço,
   nunca rebuildar ou recriar a versão antiga a partir do código.
```

Isso implica não fazer `docker image prune` agressivo em PROD logo após o deploy — mantenha pelo menos a imagem N-1 disponível localmente por um período, para que o rollback não dependa de baixar a imagem do registry sob pressão (e não falhe se o registry estiver indisponível justamente durante o incidente que motivou o rollback).

Rollback bom é aquele que:

- não exige decisão de arquitetura no momento da falha;
- é o mesmo mecanismo usado no dia a dia (deploy de uma versão-alvo), só que apontando para trás;
- é testado antes de ser necessário de verdade (pelo menos uma vez em DEV).

---

## 24. Endpoint /version

Produção pode responder:

```json
{
  "commit": "a91c302",
  "version": "2.3.1"
}
```

Facilita auditoria.

---

## 25. Rollback

Se deploy falhar:

```text
B
 |
 FAIL
 |
 v
A
```

O procedimento deve ser tão simples quanto o deploy.

---

## 26. Rollback automático

Pode ser aplicado quando health check falha imediatamente:

```text
deploy B
 |
 health FAIL
 |
 rollback A
```

Depois notificar.

---

## 27. Rollback humano

Para falhas funcionais detectadas depois:

```text
workflow_dispatch
 |
 version anterior
 |
 deploy
```

---

## 28. Banco de dados

Rollback da aplicação pode falhar se migration B tornou o banco incompatível com A.

Por isso:

```text
migrations backward compatible
```

são essenciais.

---

## 29. Expand/Contract

```text
Release 1:
adiciona coluna nova

Release 2:
passa a usar coluna nova

Release 3:
remove coluna antiga
```

Melhor que uma migration destrutiva instantânea.

---

## 30. Backup antes de migration crítica

Fluxo:

```text
backup
 |
 verify
 |
 migration
 |
 deploy
```

Mas backup não substitui migration segura.

Trate como regra absoluta, não como recomendação: **nenhuma mudança de dado em produção acontece sem dump/backup verificado imediatamente antes**. Isso vale tanto para migration automatizada no pipeline quanto para qualquer correção manual/script rodado à mão contra o banco de PROD.

```text
dump prévio
 |
 script de correção versionado (não comando solto interativo)
 |
 aplicar
 |
 conferir resultado
```

"Verificado" significa confirmar que o dump foi gerado com sucesso (tamanho não-zero, sem erro no `stderr` do processo de backup) antes de prosseguir — um backup que falhou em silêncio é o mesmo que não ter backup.

---

## 31. Deploy locking

Dois deploys simultâneos devem ser impedidos.

GitHub:

```yaml
concurrency:
  group: deploy-production
  cancel-in-progress: false
```

---

## 32. DEV concurrency

Pode cancelar deploy antigo se uma versão nova ainda não iniciou etapa irreversível, mas isso deve ser projetado conscientemente.

---

## 33. Production concurrency

Nunca sobrepor deploys.

---

## 34. Approval context

Quem aprova deve saber:

- versão;
- PRs incluídas;
- CI;
- E2E;
- resultado DEV;
- migrations;
- rollback.

---

## 35. Release notes

Gere resumo:

```text
Version: v2.3.1
Commit: a91c302
PRs:
#120
#124
```

Ajuda decisão.

---

## 36. Changelog

Pode ser gerado a partir de PRs/commits convencionais.

Não dependa exclusivamente de geração automática sem revisão.

---

## 37. Tags Git

Release:

```bash
git tag -a v2.3.1 -m "Release v2.3.1"
```

A tag conecta versão e commit.

---

## 38. Deploy por tag

Uma estratégia:

```text
tag v2.3.1
 |
 v
workflow release
```

Ainda pode existir gate.

---

## 39. Sem staging

Riscos:

- DEV precisa ser estável;
- testes integrados precisam ser bons;
- validação visual precisa ocorrer;
- rollback precisa ser rápido.

---

## 40. DEV não é laboratório permanente quebrado

Se DEV está sempre instável, ele deixa de ser ambiente de validação.

Tenha disciplina de qualidade também em DEV.

---

## 41. Dados DEV

Devem permitir validar fluxos sem expor dados sensíveis de produção.

Idealmente:

- dados sintéticos;
- dados anonimizados quando permitido;
- reset controlado.

---

## 42. Produção

Deploy deve ser:

- rastreável;
- repetível;
- reversível;
- observável.

---

## 43. SSH host verification

Não desabilite:

```text
StrictHostKeyChecking
```

indiscriminadamente.

Gerencie `known_hosts`.

---

## 44. Deploy keys

Use credencial dedicada ao servidor/ambiente.

Rotacione conforme política.

---

## 45. sudo

Usuário de deploy não deveria possuir `sudo ALL` se só precisa reiniciar um serviço específico.

---

## 46. Docker group

Acesso ao Docker implica privilégios elevados.

Trate usuário de deploy como privilegiado.

---

## 47. Runner dedicado

Preferência:

```text
runner-ci
sem PROD

runner-deploy
acesso PROD
```

---

## 48. Firewall

Servidor PROD deve aceitar SSH apenas de origens necessárias quando possível.

---

## 49. VPN/rede privada

Uma arquitetura melhor pode manter deploy dentro de rede privada.

---

## 50. Artifact registry

O servidor PROD precisa apenas:

```text
pull artifact aprovado
```

e não acesso completo ao repositório.

---

## 51. Registry credentials

Use token read-only no servidor se possível.

CI de publicação usa permissão de escrita separada.

---

## 52. Principle of least privilege

```text
CI push registry
PROD pull registry
```

Não:

```text
PROD token com admin do registry
```

---

## 53. Blue/Green

Arquitetura:

```text
Proxy
 |
 +-- BLUE current
 |
 +-- GREEN new
```

Depois de validar GREEN:

```text
proxy -> GREEN
```

Rollback:

```text
proxy -> BLUE
```

---

## 54. Quando usar blue/green

Quando downtime e rollback precisam ser muito baixos.

Custa mais recursos e complexidade: dobro de instâncias rodando durante a troca, proxy/load balancer capaz de trocar o alvo, e gestão de estado (sessões, filas, migrations) compatível com as duas versões coexistindo brevemente.

---

## 54a. Quando NÃO vale a pena

Para a maioria dos setups pequenos/médios — um único servidor de aplicação, uma equipe pequena, um domínio de negócio que tolera alguns segundos de indisponibilidade em janela combinada — blue/green e canary custam mais do que entregam:

```text
setup simples:
  1 servidor, 1 stack Compose
  deploy = trocar imagem + reiniciar container
  downtime de poucos segundos, aceitável
  -> blue/green é complexidade desnecessária
```

Nesses casos, o que realmente reduz risco é o que já foi descrito neste volume: artifact imutável, health check obrigatório, rollback rápido testado e gate humano antes de produção (Seções 4, 14, 18, 23a). Blue/green e canary resolvem um problema diferente — *impacto de usuários durante a janela de deploy em sistemas com tráfego contínuo e várias instâncias* — não são pré-requisito de um pipeline de CI/CD maduro.

Adote blue/green ou canary quando o custo de alguns segundos de indisponibilidade for realmente alto (SLA contratual, tráfego 24/7 relevante) ou quando já existir infraestrutura (load balancer, múltiplas instâncias) que os suporte sem esforço extra significativo.

---

## 55. Rolling deployment

Com múltiplas instâncias:

```text
A1 A2 A3
```

atualizar uma por vez.

Requer load balancer e health checks.

---

## 56. Canary

Liberar nova versão para pequena parcela do tráfego, observar métricas, e só então liberar para o restante.

Adequado para sistemas com volume de tráfego suficiente para que uma fração pequena já gere sinal estatístico útil, e com infraestrutura de roteamento (proxy, service mesh, feature flag por percentual) que suporte o split.

Não é requisito inicial: para um setup simples (Seção 54a), a "canary" mais barata e eficaz continua sendo a validação em DEV seguida de observação atenta na janela pós-deploy em PROD (Seção 60).

---

## 57. Feature flags

Permitem separar deploy de release:

```text
código em PROD
 |
 flag OFF
```

Depois:

```text
flag ON
```

Requer governança.

---

## 58. Deploy versus release

Deploy:

```text
software chegou ao ambiente
```

Release:

```text
funcionalidade ficou disponível ao usuário
```

Feature flags separam os dois.

---

## 59. Post-deploy verification

Verificar:

- health;
- versão;
- logs;
- métricas;
- erros;
- smoke.

---

## 60. Observação inicial

Após PROD, acompanhe uma janela de maior atenção, sem depender de trabalho manual eterno.

Observabilidade automatizada deve assumir isso.

---

## 61. Error budget

Em sistemas maduros, deploys podem ser condicionados à saúde global do serviço.

Será aprofundado em observabilidade.

---

## 62. Deployment markers

Registre em monitoramento:

```text
deploy v2.3.1 às 14:32
```

Assim aumento de erros pode ser correlacionado.

---

## 63. Logs de deploy

Guarde:

```text
version
timestamp
actor
result
health
rollback
```

---

## 64. Notificação

Após deploy:

```text
DEV success
PROD success
PROD rollback
```

podem gerar notificação útil.

---

## 65. Não notificar cada step

Evite ruído.

Notifique eventos acionáveis.

---

## 66. Dry run

Scripts podem suportar modo de validação:

```bash
deploy.sh --check
```

para confirmar pré-requisitos sem alterar serviço.

---

## 67. Preflight checks

Antes de PROD:

```text
imagem existe?
disco suficiente?
Compose válido?
backup necessário?
migration compatível?
```

---

## 68. Disk space

Deploy pode falhar por falta de disco ao puxar imagem.

Verifique previamente.

---

## 69. Docker pull failure

Se registry indisponível:

```text
não derrubar versão atual
```

Primeiro obter artifact, depois atualizar.

---

## 70. Atomicidade

Minimize período intermediário inconsistente.

---

## 71. Config validation

Antes:

```bash
docker compose config --quiet
```

quando adequado.

---

## 72. Secret validation

Confirme presença de secrets necessários sem imprimir valores.

---

## 73. Migration lock

Evite duas instâncias aplicando migration simultaneamente.

Framework ou script deve possuir locking apropriado.

---

## 74. Zero-downtime migrations

Evite renomear/remover campos usados pela versão ainda ativa.

---

## 75. Deployment checklist DEV

- [ ] CI PASS.
- [ ] Artifact existe.
- [ ] Version identificada.
- [ ] Deploy serializado.
- [ ] Config validada.
- [ ] Health PASS.
- [ ] Smoke PASS.
- [ ] E2E relacionado.
- [ ] Evidência disponível.

---

## 76. Approval checklist

- [ ] DEV funcional.
- [ ] Frontend aprovado.
- [ ] Migrations conhecidas.
- [ ] Rollback conhecido.
- [ ] Janela adequada.
- [ ] Não há incidente ativo relevante.
- [ ] Versão correta.

---

## 77. PROD checklist

- [ ] Approval.
- [ ] Artifact exato.
- [ ] Backup quando aplicável.
- [ ] Preflight.
- [ ] Deploy.
- [ ] Health.
- [ ] Smoke.
- [ ] Version check.
- [ ] Monitoramento.
- [ ] Registro.

---

## 78. Rollback checklist

- [ ] Identificar versão anterior.
- [ ] Confirmar compatibilidade de banco.
- [ ] Reverter artifact.
- [ ] Health.
- [ ] Smoke.
- [ ] Registrar incidente.
- [ ] Preservar evidências.

---

## 79. Exemplo de scripts

```text
scripts/deploy/
├── deploy-dev.sh
├── deploy-prod.sh
├── preflight.sh
├── health-check.sh
├── smoke.sh
└── rollback.sh
```

---

## 80. Exemplo PROD conceitual

```yaml
name: Deploy PROD

on:
  workflow_dispatch:
    inputs:
      version:
        description: Version/SHA
        required: true

concurrency:
  group: deploy-production
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: production

    runs-on: [self-hosted, linux, deploy]

    steps:
      - uses: actions/checkout@v4

      - name: Preflight
        run: ./scripts/deploy/preflight.sh "${{ inputs.version }}"

      - name: Deploy
        run: ./scripts/deploy/deploy-prod.sh "${{ inputs.version }}"

      - name: Verify
        run: ./scripts/deploy/health-check.sh
```

---

## 81. Rollback automático conceitual

Script:

```bash
PREVIOUS=$(cat previous-version)

if ! deploy "$NEW"; then
  deploy "$PREVIOUS"
  exit 1
fi
```

O mecanismo real deve ser robusto contra falhas parciais.

---

## 82. Deploy auditável

Meta:

```text
Quem?
Quando?
Qual commit?
Qual artifact?
Qual resultado?
```

Tudo deve ser respondível.

---

## 83. Próximo volume

**Volume 10 — Segurança do Pipeline**

Cobrirá:

- threat modeling;
- secrets;
- tokens;
- SSH;
- runners;
- supply chain;
- Actions;
- dependências;
- containers;
- scans;
- least privilege;
- hardening.

---

**Fim do Volume 09 — Continuous Deployment**
