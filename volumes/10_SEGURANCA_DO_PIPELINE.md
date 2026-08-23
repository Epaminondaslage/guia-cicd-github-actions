# Volume 10 — Segurança do Pipeline CI/CD

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 10_SEGURANCA_DO_PIPELINE.md  
**Versão:** 0.1.0  
**Pré-requisitos:** Volumes 01 a 09

---

## 1. Objetivo

Este volume trata o pipeline de CI/CD como infraestrutura crítica.

Fluxo de risco:

```text
Código
 |
 v
GitHub
 |
 v
Workflow
 |
 v
Runner
 |
 v
Artifact
 |
 v
Registry
 |
 v
DEV / PROD
```

Um comprometimento em qualquer elo pode afetar os seguintes.

---

# 2. Princípio central

```text
Least Privilege
```

Cada identidade deve receber somente o necessário.

Exemplo:

```text
CI runner:
read repo
write artifact

Deploy runner:
read artifact
deploy environment

PROD:
read registry
```

---

# 3. Threat Model

Perguntas:

- quem pode alterar workflow?
- quem pode abrir PR?
- runner executa código não confiável?
- quais secrets estão disponíveis?
- runner alcança rede interna?
- quem pode publicar imagem?
- quem pode aprovar produção?
- como revogar credenciais?

---

# 4. Superfícies de ataque

```text
GitHub account
tokens
Actions de terceiros
dependencies
runner host
Docker daemon
registry
SSH
secrets
deploy scripts
artifact
```

---

# 5. Conta GitHub

Proteções fundamentais:

- MFA;
- senha única;
- recovery codes seguros;
- sessões revisadas;
- tokens antigos removidos.

---

# 6. Tokens

Nunca criar token com escopo maior apenas por conveniência.

Separe:

```text
read
write
deploy
admin
```

---

# 7. GITHUB_TOKEN

Declare permissões:

```yaml
permissions:
  contents: read
```

Expanda somente quando necessário.

---

# 8. Secrets

Armazene credenciais no mecanismo adequado.

Nunca em:

```text
Git
README
Dockerfile
logs
SPEC
prompt
```

---

# 9. Rotação

Se secret for exposto:

```text
1. revogar
2. substituir
3. investigar uso
4. remover do histórico quando necessário
```

Não basta apagar o arquivo atual.

---

# 10. Environment secrets

Separe:

```text
development
production
```

Secret de produção não deve ser disponibilizado ao CI de PR.

---

# 11. SSH

Use chaves dedicadas.

Evite:

```text
chave pessoal do administrador
```

como chave de deploy.

---

# 12. Usuário de deploy

Exemplo:

```text
deploy
```

Permissões mínimas.

---

# 13. sudoers

Se precisa de um comando específico, permita somente esse comando quando tecnicamente viável.

Evite:

```text
NOPASSWD: ALL
```

---

# 14. Host key verification

Proteja contra MITM.

Mantenha `known_hosts`.

---

# 15. Self-hosted runner

É uma superfície privilegiada porque executa código.

Nunca trate runner como computador comum.

---

# 16. Runner persistente

Resíduos podem permanecer:

```text
workspace
cache
containers
files
processes
```

Implemente cleanup e isolamento.

---

# 17. PR não confiável

Não execute automaticamente código externo em runner interno com acesso privilegiado.

Especialmente se:

```text
Docker
LAN
SSH
secrets
```

estão disponíveis.

---

# 18. Runner dedicado

Arquitetura recomendada:

```text
runner-ci
 |
 sem PROD

runner-deploy
 |
 acesso PROD
```

---

# 19. Runner efêmero

Meta futura:

```text
job
 |
 runner novo
 |
 execute
 |
 destruir
```

Melhora isolamento.

---

# 20. Docker group

Usuário no grupo Docker tem poder equivalente a privilégios elevados.

Trate como conta privilegiada.

---

# 21. Docker socket

Nunca montar:

```text
/var/run/docker.sock
```

em containers não confiáveis sem compreender impacto.

---

# 22. Actions de terceiros

Uma linha:

```yaml
uses: vendor/action@v1
```

executa código externo.

Avalie reputação, manutenção e permissões.

---

# 23. Pinning

Em ambientes de segurança elevada, fixe Action por SHA.

Tags podem mudar.

---

# 24. Dependabot

Use para dependências e Actions conforme estratégia.

Atualização deve passar por PR + CI.

---

# 25. Supply Chain

Ataque pode ocorrer em:

```text
npm package
Composer package
base image
Action
registry
build server
```

---

# 26. Lockfiles

Versione lockfiles.

Eles aumentam previsibilidade.

---

# 27. npm

Use:

```bash
npm ci
```

no CI.

---

# 28. Composer

Use `composer.lock` e instalação reproduzível.

---

# 29. Dependency scanning

Adote ferramenta que identifique vulnerabilidades conhecidas.

Defina política por severidade.

---

# 30. Falso positivo

Scanner não deve ser ignorado nem bloquear cegamente.

Tenha processo:

```text
detectar
avaliar
corrigir/aceitar temporariamente
documentar
```

---

# 31. Secret scanning

Automatize busca por:

- tokens;
- private keys;
- passwords;
- credentials.

GitHub possui mecanismos próprios e existem ferramentas open source.

---

# 32. Gitleaks

Gitleaks é opção open source para secret scanning.

Pode executar em CI.

---

# 33. TruffleHog

Outra ferramenta conhecida para descoberta de secrets.

Avalie integração conforme projeto.

---

# 34. Static Analysis

JavaScript/TypeScript:

- ESLint;
- Semgrep;
- CodeQL em contextos aplicáveis.

PHP:

- PHPStan;
- Psalm;
- Semgrep.

---

# 35. Semgrep

Permite regras de análise estática e possui opção open source.

Útil para padrões de segurança específicos.

---

# 36. Container scan

Imagens devem ser analisadas.

Ferramentas open source incluem Trivy.

---

# 37. Trivy

Pode verificar:

- imagens;
- filesystem;
- dependências;
- configurações.

Integre gradualmente.

---

# 38. Base images

Prefira imagens oficiais/confiáveis.

Evite:

```text
randomuser/node-super-complete
```

sem auditoria.

---

# 39. Imagens mínimas

Menor imagem pode reduzir superfície, mas não sacrifique depuração/compatibilidade sem motivo.

Segurança é mais que tamanho.

---

# 40. Non-root container

Aplicação deve rodar como usuário não-root sempre que viável.

---

# 41. Privileged container

Evite:

```yaml
privileged: true
```

sem necessidade forte.

---

# 42. Capabilities

Remova capabilities desnecessárias.

---

# 43. Read-only

Considere filesystem read-only para serviços compatíveis.

---

# 44. Network segmentation

CI não precisa acessar toda LAN.

Separe redes:

```text
CI network
DEV network
PROD network
```

---

# 45. Firewall

Permita somente tráfego necessário.

---

# 46. Egress

Controle de saída também é relevante para ambientes críticos.

Código comprometido pode tentar exfiltrar secrets.

---

# 47. Registry

Separe permissões:

```text
CI -> push
PROD -> pull
```

---

# 48. Imagens imutáveis

Use tags por SHA/digest.

Isso reduz ambiguidade.

---

# 49. Artifact integrity

Uma evolução é assinar artifacts/imagens.

---

# 50. Cosign

Sigstore Cosign é uma ferramenta open source popular para assinatura/verificação de containers/artifacts.

Pode ser incorporada em fase avançada.

---

# 51. SBOM

Gere inventário dos componentes.

Ferramentas como Syft podem ajudar.

---

# 52. Syft

Gera SBOM para imagens/filesystems.

Pode combinar com scanners.

---

# 53. Provenance

Registre:

```text
repo
commit
workflow
builder
artifact
```

Ajuda investigação.

---

# 54. Branch protection

`main` protegida:

- PR obrigatório;
- checks;
- review conforme política;
- sem force push;
- resolução de conversas.

---

# 55. CODEOWNERS

Arquivos sensíveis podem exigir responsável específico.

Exemplos:

```text
.github/workflows/
Dockerfile
infra/
scripts/deploy/
```

---

# 56. Workflow changes

Mudanças em workflows merecem revisão rigorosa porque podem alterar:

- permissions;
- secrets;
- deploy;
- runner.

---

# 57. Pull request target

Eventos privilegiados devem ser usados com extremo cuidado.

Nunca execute checkout/código não confiável com secrets privilegiados sem compreender semântica.

---

# 58. Forks

Política deve definir se contribuições externas:

- rodam em GitHub-hosted;
- exigem aprovação;
- não recebem secrets;
- nunca rodam em runner interno.

---

# 59. Logs

Secrets podem vazar por:

```bash
set -x
env
printenv
echo
```

Evite imprimir ambientes completos.

---

# 60. Masking

Não confie apenas em mascaramento automático.

A melhor defesa é não imprimir.

---

# 61. Artifacts sensíveis

Relatórios podem conter:

- cookies;
- headers;
- screenshots com dados;
- traces.

Defina retenção e acesso.

---

# 62. Playwright traces

Podem capturar dados de sessão.

Não publique artifacts de produção publicamente.

---

# 63. Dados reais

CI deve usar dados sintéticos.

Evite copiar banco PROD integral para testes.

---

# 64. Anonimização

Quando dados reais forem necessários em ambientes controlados, anonimize conforme requisitos legais e de negócio.

---

# 65. Database credentials

Separe:

```text
test
dev
prod
```

Nunca reutilize senha PROD em DEV.

---

# 66. Database user

Aplicação não precisa normalmente de usuário root.

Conceda apenas schema/operações necessárias.

---

# 67. Migration user

Pode ser separado do usuário runtime.

---

# 68. Backup credentials

Também devem ser segregadas.

---

# 69. MQTT security

Para produção:

- autenticação;
- ACL;
- TLS quando apropriado;
- tópicos com menor privilégio.

CI usa broker isolado.

---

# 70. MQTT wildcard

Evite permitir:

```text
#
```

para qualquer cliente sem necessidade.

---

# 71. Webhooks

Valide assinatura/autenticidade quando o provedor suporta.

---

# 72. Replay attacks

Para webhooks críticos, considere timestamp/nonce/idempotência.

---

# 73. API keys

Não colocar no frontend se são secrets de servidor.

Código executado no browser não pode guardar secret real.

---

# 74. CORS

Não usar:

```text
*
```

indiscriminadamente quando credenciais/sensibilidade existem.

---

# 75. Headers

Reverse proxy pode adicionar headers de segurança.

Configuração depende da aplicação.

---

# 76. Dependency policy

Antes de adicionar pacote:

- necessidade;
- manutenção;
- licença;
- vulnerabilidades;
- transitive dependencies.

---

# 77. AI-generated dependencies

Agentes podem sugerir pacotes inexistentes, obsoletos ou desnecessários.

Sempre valide.

---

# 78. Prompt injection em repositórios

Agentes lendo Issues/docs podem encontrar instruções maliciosas.

Ferramentas com capacidade de escrita/deploy devem tratar conteúdo do repo como dados não totalmente confiáveis.

---

# 79. IA e secrets

Não copie secrets em chats/prompts.

Use conectores/secret stores e referências.

---

# 80. Autonomia da IA

Produção deve continuar protegida por gate e permissions.

---

# 81. Audit log

Registre:

```text
quem alterou workflow
quem aprovou
quem fez deploy
qual artifact
```

---

# 82. Incident response

Se pipeline comprometido:

```text
1. bloquear deploy
2. revogar tokens
3. desligar runner afetado
4. preservar logs
5. verificar artifacts
6. reconstruir ambiente limpo
```

---

# 83. Runner rebuild

Não tente necessariamente "limpar" host comprometido.

Para incidentes sérios:

```text
reinstalar/recriar
```

é mais confiável.

---

# 84. Golden image

Uma evolução é automatizar criação do runner a partir de imagem conhecida.

---

# 85. Patch management

Atualize:

- Ubuntu;
- Docker;
- runner;
- Node;
- PHP;
- browsers.

---

# 86. Unattended upgrades

Podem ser úteis para patches de segurança, mas avalie impacto em runners críticos.

---

# 87. Reboot

Tenha procedimento para reiniciar runner sem deixar pipeline inconsistente.

---

# 88. Monitoring

Detecte:

```text
runner offline
disk full
high load
Docker failed
```

---

# 89. Backups

Backup de configuração não deve incluir secrets em texto claro sem proteção.

---

# 90. Restore test

Teste reconstrução do runner.

---

# 91. Security Gate PR

Sugestão inicial:

```text
lint
unit
secret scan
dependency scan básico
```

---

# 92. Security Gate Nightly

```text
container scan
static analysis ampliado
full dependency scan
```

---

# 93. Release security

Antes de release crítica:

```text
SBOM
scan
artifact digest
```

---

# 94. Não bloquear por tudo

Crie política de severidade.

Exemplo:

```text
Critical -> block
High -> review
Medium -> backlog
```

A política depende do risco.

---

# 95. Exception process

Se vulnerabilidade não pode ser corrigida imediatamente:

- documentar;
- prazo;
- mitigação;
- responsável;
- revisão futura.

---

# 96. Security checklist workflow

- [ ] permissions mínimas.
- [ ] Actions confiáveis.
- [ ] secrets mínimos.
- [ ] sem logs sensíveis.
- [ ] timeout.
- [ ] runner correto.
- [ ] PR externa tratada.
- [ ] artifact protegido.

---

# 97. Security checklist runner

- [ ] usuário dedicado.
- [ ] SSH por chave.
- [ ] firewall.
- [ ] patches.
- [ ] Docker controlado.
- [ ] sem PROD no CI.
- [ ] disk monitoring.
- [ ] rebuild documentado.

---

# 98. Security checklist deploy

- [ ] gate humano.
- [ ] chave dedicada.
- [ ] environment secrets.
- [ ] artifact imutável.
- [ ] registry least privilege.
- [ ] health.
- [ ] rollback.
- [ ] audit.

---

# 99. Arquitetura alvo

```text
GitHub
 |
 v
CI Runner
 |
 +-- read code
 +-- tests
 +-- build
 +-- scan
 |
 v
Registry
 |
 v
Deploy Runner
 |
 +-- read artifact
 +-- controlled SSH
 |
 v
PROD
```

---

# 100. Próximo volume

**Volume 11 — Observabilidade**

Cobrirá:

- logs;
- métricas;
- Prometheus;
- Grafana;
- Loki;
- alertas;
- health;
- deployment markers;
- SLI/SLO.

---

**Fim do Volume 10 — Segurança do Pipeline CI/CD**
