# Volume 10 — segurança do pipeline CI/CD

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 10_SEGURANCA_DO_PIPELINE.md  
**Versão:** 0.2.0  
**Pré-requisitos:** Volumes 01 a 09

---

## 1. objetivo

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

## 2. princípio central

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

## 3. threat model

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

## 4. superfícies de ataque

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

## 5. conta GitHub

Proteções fundamentais:

| Proteção | Detalhe |
|---|---|
| MFA | obrigatório para quem administra o repositório ou a organização |
| Senha única | gerenciador de senhas, nunca reaproveitada |
| Recovery codes seguros | fora do repositório, fora do disco compartilhado |
| Sessões revisadas periodicamente | Settings → Sessions |
| Tokens antigos removidos | Personal Access Tokens e SSH keys não usados há meses |

---

## 6. tokens

Nunca criar token com escopo maior apenas por conveniência.

Prefira **fine-grained Personal Access Tokens** (escopo por repositório e por permissão) a *classic tokens* (escopo amplo, praticamente "tudo ou nada").

Separe por finalidade:

```text
read
write
deploy
admin
```

Defina expiração. Um token sem data de expiração é uma dívida técnica de segurança.

---

## 7. GITHUB_TOKEN

O `GITHUB_TOKEN` é gerado automaticamente pelo GitHub para cada execução de workflow e expira ao final do job. Ele **não** deve ser confundido com um PAT.

Desde 2023, novos repositórios podem ter o padrão configurado como somente leitura (`Settings → Actions → Workflow permissions`), mas isso é configuração de repositório/organização — não assuma, **declare explicitamente** no workflow.

Boa prática: definir `permissions` no nível do **workflow** com o mínimo comum, e elevar apenas no **job** que realmente precisa:

```yaml
permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4

  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write   # só este job precisa criar release/tag
    steps:
      - run: echo "cria release"
```

Sem o bloco `permissions:`, o comportamento herda o padrão da organização/repositório — que pode ser mais permissivo do que o pipeline precisa. Não deixe implícito.

---

## 8. secrets

Armazene credenciais em **GitHub Secrets** (repositório, ambiente ou organização) — nunca em texto plano no código.

Nunca em:

```text
Git (arquivos versionados)
README
Dockerfile
logs
SPEC
prompt (chat com IA)
comentário de PR
```

Isso vale mesmo para repositórios privados: privado não é sinônimo de seguro, e qualquer colaborador com acesso de leitura passa a ter acesso ao segredo.

---

## 9. rotação

Se um secret for exposto — versionado por engano, colado em log, colado em um prompt de IA — o procedimento correto é:

```text
1. revogar a credencial exposta imediatamente
2. gerar uma nova credencial (rotacionar)
3. investigar uso indevido (logs de acesso, billing, atividade anômala)
4. remover do histórico do git quando aplicável (git filter-repo, BFG)
```

Ponto crítico: **remover do histórico não neutraliza o vazamento**. `git filter-repo` e o BFG Repo-Cleaner reescrevem o histórico local e podem forçar um novo push, mas não apagam cópias que já foram clonadas, indexadas por scrapers automatizados (existem bots que varrem GitHub em busca de segredos em tempo real) ou cacheadas por serviços de terceiros. Uma vez exposto, o segredo deve ser tratado como comprometido — a única mitigação real é a **rotação**. Limpar o histórico é higiene depois da rotação, não substituto dela.

---

## 10. Environment secrets

Use **GitHub Environments** (`Settings → Environments`) para segregar:

```text
development
staging
production
```

Cada Environment pode ter:

| Recurso | Descrição |
|---|---|
| Secrets próprios | isolados dos demais |
| *Required reviewers* | aprovação humana antes do job rodar |
| *Deployment branches* | só `main`/`release/*` pode disparar deploy no ambiente |
| *Wait timer* | — |

Secret de produção não deve ser disponibilizado ao CI de PR nem a jobs que rodam a partir de forks — um workflow disparado por `pull_request` de um fork não tem acesso aos secrets do Environment por padrão, e essa proteção não deve ser contornada sem necessidade real.

---

## 11. SSH

Use chaves dedicadas por finalidade (deploy, backup, monitoramento).

Evite:

```text
chave pessoal do administrador
```

como chave de deploy. Se a pessoa sai da equipe ou troca de notebook, a chave de deploy não deve precisar ser trocada junto.

A chave de deploy deve ter escopo mínimo (deploy key do repositório específico, ou usuário dedicado com permissões restritas no host de destino), nunca a chave usada para acesso administrativo geral.

---

## 12. usuário de deploy

Exemplo:

```text
deploy
```

Permissões mínimas: acesso de escrita apenas ao diretório da aplicação, nunca root.

Um usuário de deploy dedicado (não-root) que executa `git pull` e reinicia o serviço reduz drasticamente o raio de explosão de uma chave SSH vazada ou de um comando de deploy malformado.

---

## 13. sudoers

Se precisa de um comando específico, permita somente esse comando quando tecnicamente viável.

Evite:

```text
NOPASSWD: ALL
```

---

## 14. host key verification

Proteja contra MITM.

Mantenha `known_hosts` versionado ou provisionado, e nunca use `StrictHostKeyChecking=no` em pipelines de produção só para "resolver" um prompt interativo.

---

## 15. self-hosted runner

É uma superfície privilegiada porque executa código.

Nunca trate runner como computador comum.

---

## 16. runner persistente

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

## 17. PR não confiável

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

## 18. runner dedicado

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

## 19. runner efêmero

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

## 20. Docker group

Usuário no grupo Docker tem poder equivalente a privilégios elevados.

Trate como conta privilegiada.

---

## 21. Docker socket

Nunca montar:

```text
/var/run/docker.sock
```

em containers não confiáveis sem compreender impacto.

---

## 22. Actions de terceiros

Uma linha:

```yaml
uses: vendor/action@v1
```

executa código externo com acesso ao contexto do job, incluindo secrets injetados nas etapas anteriores.

Avalie reputação, manutenção e permissões antes de adicionar uma Action ao pipeline.

---

## 23. pinning

Em ambientes de segurança elevada, fixe Action por SHA de commit, não por tag:

```yaml
uses: actions/checkout@8f4b7f84864484a7bf31766abe9204da3cbe65b3  # v4.1.1
```

Tags (`@v4`, `@v4.1.1`) podem ser movidas pelo mantenedor da Action — inclusive de forma maliciosa, em caso de comprometimento da conta do mantenedor. SHA de commit é imutável.

---

## 24. atualização de dependências (Dependabot / Renovate)

Automatize a atualização de dependências e de Actions de terceiros com **Dependabot** (nativo do GitHub) ou **Renovate** (mais configurável, roda como app ou self-hosted).

Configuração mínima com Dependabot (`.github/dependabot.yml`):

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

Pontos de atenção:

- toda atualização deve passar por PR + CI, nunca merge automático sem checks;
- separe atualizações de patch/minor (baixo risco, podem ter merge mais leve) de major (revisão manual);
- Dependabot também cobre `github-actions` como ecossistema — mantenha as próprias Actions atualizadas, não só `package.json`/`composer.json`.

---

## 25. supply chain

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

## 26. lockfiles

Versione lockfiles.

Eles aumentam previsibilidade.

---

## 27. npm

Use:

```bash
npm ci
```

no CI. `npm ci` instala exatamente o que está no lockfile e falha se `package.json`/`package-lock.json` estiverem dessincronizados — diferente de `npm install`, que pode atualizar o lockfile silenciosamente.

---

## 28. Composer

Use `composer.lock` e instalação reproduzível (`composer install --no-dev` em produção, sem `composer update` no pipeline de deploy).

---

## 29. dependency scanning (SCA)

Adote uma ferramenta de *Software Composition Analysis* que identifique vulnerabilidades conhecidas (CVEs) em dependências diretas e transitivas. Opções comuns:

```text
Dependabot alerts (nativo GitHub, gratuito)
npm audit / npm audit signatures
Trivy (também cobre dependências, não só imagem)
Grype (Anchore)
Snyk
```

Defina política por severidade (ver seção 94).

---

## 30. falso positivo

Scanner não deve ser ignorado nem bloquear cegamente.

Tenha processo:

```text
detectar
avaliar
corrigir/aceitar temporariamente
documentar
```

---

## 31. secret scanning

Automatize busca por:

- tokens;
- private keys;
- passwords;
- credentials.

GitHub possui **secret scanning** nativo (ativado por padrão em repositórios públicos, disponível para privados conforme plano) e **push protection** (bloqueia o push antes que o segredo entre no histórico). Habilite ambos em `Settings → Code security`. Isso não substitui a rotação (seção 9) quando algo escapa — só reduz a chance de escapar.

---

## 32. Gitleaks

Gitleaks é opção open source para secret scanning, roda bem como step de CI ou como pre-commit hook local.

---

## 33. TruffleHog

Outra ferramenta conhecida para descoberta de secrets, com verificação ativa de validade de algumas credenciais encontradas (não só padrão de string).

---

## 34. static analysis (SAST)

| Linguagem | Ferramentas |
|---|---|
| JavaScript/TypeScript | ESLint (com plugins de segurança, ex. `eslint-plugin-security`); Semgrep; CodeQL |
| PHP | PHPStan; Psalm; Semgrep |

**CodeQL** é a ferramenta de SAST nativa do GitHub (gratuita para repositórios públicos, incluída no GitHub Advanced Security para privados). Exemplo mínimo de workflow:

```yaml
name: CodeQL
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 3 * * 1'

jobs:
  analyze:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read
    strategy:
      matrix:
        language: ['javascript-typescript']
    steps:
      - uses: actions/checkout@v4
      - uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
      - uses: github/codeql-action/analyze@v3
```

---

## 35. Semgrep

Permite regras de análise estática e possui opção open source (`semgrep --config auto` ou registro de regras próprio).

Útil para padrões de segurança específicos do projeto, além dos padrões genéricos de linguagem.

---

## 36. container scan

Imagens devem ser analisadas antes de ir para o registry de produção.

Ferramentas open source incluem Trivy e Grype.

---

## 37. Trivy

Pode verificar:

- imagens de container;
- filesystem;
- dependências (SCA);
- configurações (Dockerfile, Kubernetes manifests, Terraform).

Exemplo de step de CI:

```yaml
- name: Scan de imagem
  uses: aquasecurity/trivy-action@0.28.0
  with:
    image-ref: 'registro/app:${{ github.sha }}'
    severity: 'CRITICAL,HIGH'
    exit-code: '1'
```

Integre gradualmente: comece só reportando (`exit-code: '0'`), depois passe a bloquear por severidade.

---

## 38. Grype

Alternativa/complemento ao Trivy, também open source (Anchore), focada em SCA de imagens e SBOMs — combina bem com Syft (seção 52), do mesmo projeto.

---

## 39. base images

Prefira imagens oficiais/confiáveis (`node:20-slim`, `php:8.3-fpm`, imagens publicadas por mantenedores reconhecidos).

Evite:

```text
randomuser/node-super-complete
```

sem auditoria.

---

## 40. imagens mínimas

Menor imagem pode reduzir superfície, mas não sacrifique depuração/compatibilidade sem motivo.

Segurança é mais que tamanho.

---

## 41. non-root container

Aplicação deve rodar como usuário não-root sempre que viável:

```dockerfile
USER node
```

---

## 42. privileged container

Evite:

```yaml
privileged: true
```

sem necessidade forte.

---

## 43. capabilities

Remova capabilities desnecessárias (`--cap-drop=ALL` e adicione de volta só o estritamente necessário).

---

## 44. read-only

Considere filesystem read-only para serviços compatíveis (`read_only: true` no Compose, com volumes explícitos para o que precisa gravar).

---

## 45. network segmentation

CI não precisa acessar toda LAN.

Separe redes:

```text
CI network
DEV network
PROD network
```

---

## 46. firewall

Permita somente tráfego necessário.

---

## 47. egress

Controle de saída também é relevante para ambientes críticos.

Código comprometido (dependência maliciosa, Action comprometida) pode tentar exfiltrar secrets via requisição de rede. Bloquear egress não previsto é uma defesa em profundidade, não uma solução isolada.

---

## 48. Registry

Separe permissões:

```text
CI -> push
PROD -> pull
```

Um runner de deploy comprometido não deveria conseguir sobrescrever imagens no registry.

---

## 49. imagens imutáveis

Use tags por SHA/digest (`registro/app@sha256:...`), não apenas `latest` ou uma tag mutável.

Isso reduz ambiguidade sobre o que está de fato rodando em cada ambiente.

---

## 50. artifact integrity

Uma evolução é assinar artifacts/imagens, de forma que o ambiente de destino só aceite artefatos com origem verificável.

---

## 51. Cosign / Sigstore

**Cosign** (projeto Sigstore) é a ferramenta open source de referência para assinatura e verificação de imagens de container e outros artefatos.

Fluxo típico com *keyless signing* (usa OIDC do próprio GitHub Actions, sem gerenciar chave privada):

```yaml
permissions:
  id-token: write   # necessário para o OIDC do cosign keyless
  contents: read
  packages: write

steps:
  - uses: sigstore/cosign-installer@v3
  - name: Assinar imagem
    run: cosign sign --yes registro/app@${{ steps.build.outputs.digest }}
```

E, no lado do deploy, verificar antes de rodar:

```bash
cosign verify \
  --certificate-identity-regexp "https://github.com/ORG/REPO/.github/workflows/.*" \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com" \
  registro/app@sha256:...
```

Isso fecha o ciclo: a imagem só é aceita em produção se foi assinada pelo próprio workflow confiável do repositório.

---

## 52. SBOM

Gere inventário dos componentes de cada imagem/build (*Software Bill of Materials*) — útil para responder rapidamente "estamos expostos à CVE X?" sem precisar re-escanear tudo.

---

## 53. Syft

Gera SBOM para imagens/filesystems em formatos padronizados (SPDX, CycloneDX).

Pode combinar com Grype/Trivy para checar o SBOM gerado contra bases de vulnerabilidade.

---

## 54. provenance

Registre:

```text
repo
commit
workflow
builder
artifact digest
```

O GitHub oferece **artifact attestations** nativas (`actions/attest-build-provenance`), que geram e assinam metadados de proveniência sem precisar orquestrar Sigstore manualmente:

```yaml
permissions:
  id-token: write
  contents: read
  attestations: write

steps:
  - uses: actions/attest-build-provenance@v1
    with:
      subject-path: 'dist/app.tar.gz'
```

Ajuda investigação e responde "esse artefato realmente veio deste pipeline?".

---

## 55. OIDC para autenticação com nuvem

Ao integrar o pipeline com AWS, Azure ou GCP (deploy de infraestrutura, push para registry gerenciado, etc.), **prefira OIDC a credenciais estáticas de longa duração**.

Com credenciais estáticas (`AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY` guardadas como secret), qualquer vazamento do secret dá acesso indefinido até rotação manual. Com OIDC, o GitHub Actions emite um token de curta duração assinado, o provedor de nuvem valida esse token contra uma *trust policy* (que amarra o acesso a um repositório e branch/environment específicos) e concede credenciais temporárias só para aquela execução.

Exemplo com AWS:

```yaml
permissions:
  id-token: write   # obrigatório para o OIDC
  contents: read

steps:
  - uses: actions/checkout@v4
  - uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: arn:aws:iam::123456789012:role/deploy-role
      aws-region: us-east-1
  - run: aws s3 sync ./dist s3://meu-bucket
```

Sem `secrets.AWS_ACCESS_KEY_ID` nenhum. A *trust policy* do lado da AWS (IAM) restringe qual repositório/branch pode assumir a role — o equivalente existe para Azure (`azure/login` com Federated Credentials) e GCP (Workload Identity Federation).

Quando o provedor de nuvem não suportar OIDC, ao menos: credencial com escopo mínimo, em Environment secret, com rotação programada.

---

## 56. branch protection

`main` protegida:

- PR obrigatório (nenhum push direto, nem de admins, se a política assim exigir);
- *required status checks* (CI precisa passar — lint, testes, scans — antes do merge ser permitido);
- *required reviews* (ao menos um revisor aprovando; considerar exigir revisão de code owner);
- sem force push;
- sem exclusão da branch;
- resolução de conversas obrigatória antes do merge.

Configuração em `Settings → Branches → Branch protection rules`. Vale revisar também **rulesets** (mecanismo mais novo do GitHub, permite aplicar a múltiplos branches/tags com um único conjunto de regras).

---

## 57. CODEOWNERS

Arquivos sensíveis podem exigir responsável específico como revisor obrigatório.

Exemplos (`.github/CODEOWNERS`):

```text
.github/workflows/ @time-plataforma
Dockerfile         @time-plataforma
infra/             @time-plataforma
scripts/deploy/    @time-plataforma
```

Combine com "Require review from Code Owners" na branch protection — sem isso, o CODEOWNERS é só documentação.

---

## 58. workflow changes

Mudanças em workflows merecem revisão rigorosa porque podem alterar:

- permissions;
- secrets acessíveis;
- deploy;
- runner utilizado.

Um workflow malicioso ou mal revisado pode, por exemplo, exfiltrar secrets adicionando um step de `curl` para um endpoint externo.

---

## 59. pull request target

O evento `pull_request_target` roda com o contexto (e os secrets) do repositório base, mesmo para PRs vindos de forks — diferente de `pull_request`, que roda com contexto restrito.

Isso é privilegiado por design e deve ser usado com extremo cuidado. Nunca faça checkout do código do PR (`ref: ${{ github.event.pull_request.head.sha }}`) e execute esse código com secrets disponíveis em um workflow disparado por `pull_request_target` — é o padrão clássico de comprometimento via PR malicioso.

---

## 60. forks

Política deve definir se contribuições externas:

- rodam em GitHub-hosted runners (nunca self-hosted);
- exigem aprovação de um mantenedor antes de rodar (`Settings → Actions → Fork pull request workflows`);
- não recebem secrets de Environment;
- nunca rodam em runner interno com acesso à LAN.

---

## 61. logs

Secrets podem vazar por:

```bash
set -x
env
printenv
echo
```

Evite imprimir ambientes completos. O GitHub mascara valores que batem exatamente com um secret registrado, mas isso não cobre secrets transformados (base64, concatenados, parcialmente impressos).

---

## 62. masking

Não confie apenas no mascaramento automático do GitHub Actions.

A melhor defesa é não imprimir o secret em lugar nenhum — nem em `echo`, nem em mensagem de erro, nem em corpo de requisição logado.

---

## 63. artifacts sensíveis

Relatórios podem conter:

- cookies;
- headers de autenticação;
- screenshots com dados de sessão;
- traces.

Defina retenção (`retention-days`) e acesso (artifacts de workflow privado não são públicos, mas ficam acessíveis a qualquer colaborador com leitura no repositório).

---

## 64. Playwright traces

Podem capturar dados de sessão (cookies, tokens em localStorage, requisições completas).

Não publique artifacts de execução contra produção publicamente, e considere retenção curta para traces que tocam ambientes reais.

---

## 65. dados reais

CI deve usar dados sintéticos.

Evite copiar banco PROD integral para testes.

---

## 66. anonimização

Quando dados reais forem necessários em ambientes controlados, anonimize conforme requisitos legais e de negócio (LGPD/GDPR conforme jurisdição).

---

## 67. database credentials

Separe:

```text
test
dev
prod
```

Nunca reutilize senha PROD em DEV.

---

## 68. database user

Aplicação não precisa normalmente de usuário root/admin do banco.

Conceda apenas schema/operações necessárias (SELECT/INSERT/UPDATE/DELETE no schema da aplicação, sem DDL em runtime).

---

## 69. migration user

Pode ser separado do usuário runtime — usuário de migration tem permissão de DDL, usuário da aplicação não.

---

## 70. backup credentials

Também devem ser segregadas, com acesso restrito a quem realmente opera backup/restore.

---

## 71. MQTT security

Para produção:

- autenticação;
- ACL;
- TLS quando apropriado;
- tópicos com menor privilégio.

CI usa broker isolado, nunca o broker de produção.

---

## 72. MQTT wildcard

Evite permitir:

```text
#
```

para qualquer cliente sem necessidade — um cliente comprometido com wildcard total lê e potencialmente publica em qualquer tópico.

---

## 73. webhooks

Valide assinatura/autenticidade quando o provedor suporta (ex.: `X-Hub-Signature-256` do GitHub).

---

## 74. replay attacks

Para webhooks críticos, considere timestamp/nonce/idempotência para impedir reprocessamento de uma requisição capturada.

---

## 75. API keys

Não colocar no frontend se são secrets de servidor.

Código executado no browser não pode guardar secret real — qualquer valor embutido em JS enviado ao cliente deve ser tratado como público.

---

## 76. CORS

Não usar:

```text
*
```

indiscriminadamente quando credenciais/sensibilidade existem.

---

## 77. headers

Reverse proxy pode adicionar headers de segurança (`Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options`).

Configuração depende da aplicação.

---

## 78. dependency policy

Antes de adicionar pacote:

- necessidade real;
- manutenção (última publicação, issues abertas);
- licença compatível;
- vulnerabilidades conhecidas;
- transitive dependencies (o pacote pequeno pode trazer uma árvore grande).

---

## 79. AI-generated dependencies

Agentes podem sugerir pacotes inexistentes, obsoletos ou desnecessários — inclusive nomes plausíveis que não existem no registro real (risco de *slopsquatting*, quando alguém registra depois o nome sugerido com conteúdo malicioso).

Sempre valide antes de instalar o que uma IA sugeriu.

---

## 80. prompt injection em repositórios

Agentes lendo Issues/PRs/docs podem encontrar instruções maliciosas embutidas em conteúdo aparentemente inofensivo (um comentário de issue, um README de dependência).

Ferramentas com capacidade de escrita/deploy devem tratar conteúdo do repositório como dado não totalmente confiável, não como instrução do operador.

---

## 81. IA e secrets

Não copie secrets em chats/prompts de IA.

Use conectores/secret stores e referências — nunca cole uma chave de API ou senha em uma conversa para "a IA testar", mesmo que o histórico pareça privado.

---

## 82. autonomia da IA

Produção deve continuar protegida por gate humano e por `permissions:` restritivas, independentemente de quanta autonomia um agente de IA tenha no restante do fluxo.

---

## 83. audit log

Registre:

```text
quem alterou workflow
quem aprovou
quem fez deploy
qual artifact
```

O GitHub mantém audit log de organização (`Settings → Audit log`) — vale revisar periodicamente, não só em caso de incidente.

---

## 84. incident response

Se pipeline comprometido:

```text
1. bloquear deploy
2. revogar tokens e credenciais expostas
3. desligar runner afetado
4. preservar logs
5. verificar integridade dos artifacts publicados
6. reconstruir ambiente limpo
```

---

## 85. runner rebuild

Não tente necessariamente "limpar" host comprometido.

Para incidentes sérios:

```text
reinstalar/recriar
```

é mais confiável.

---

## 86. Golden image

Uma evolução é automatizar criação do runner a partir de imagem conhecida.

---

## 87. patch management

Atualize:

- Ubuntu;
- Docker;
- runner;
- Node;
- PHP;
- browsers.

---

## 88. unattended upgrades

Podem ser úteis para patches de segurança, mas avalie impacto em runners críticos (um reboot automático no meio de um job é pior que o risco que ele mitiga).

---

## 89. reboot

Tenha procedimento para reiniciar runner sem deixar pipeline inconsistente.

---

## 90. monitoring

Detecte:

```text
runner offline
disk full
high load
Docker failed
```

---

## 91. backups

Backup de configuração não deve incluir secrets em texto claro sem proteção.

---

## 92. restore test

Teste reconstrução do runner periodicamente — um backup nunca testado é uma suposição, não uma garantia.

---

## 93. security gate PR

Sugestão inicial:

```text
lint
unit
secret scan (gitleaks/push protection)
dependency scan básico (npm audit / Dependabot alerts)
```

---

## 94. security gate nightly

```text
container scan (Trivy/Grype)
static analysis ampliado (CodeQL/Semgrep)
full dependency scan
```

---

## 95. release security

Antes de release crítica:

```text
SBOM (Syft)
scan completo (Trivy/Grype)
artifact digest fixado
assinatura (cosign), se aplicável
```

---

## 96. não bloquear por tudo

Crie política de severidade.

Exemplo:

```text
Critical -> block
High -> review
Medium -> backlog
```

A política depende do risco real do projeto, não de um padrão genérico copiado de outro contexto.

---

## 97. exception process

Se vulnerabilidade não pode ser corrigida imediatamente:

- documentar;
- prazo;
- mitigação;
- responsável;
- revisão futura.

---

## 98. security checklist workflow

- [ ] `permissions:` mínimas, declaradas explicitamente no workflow e por job.
- [ ] Actions de terceiros confiáveis e, quando crítico, pinadas por SHA.
- [ ] secrets mínimos, escopados por Environment.
- [ ] sem logs sensíveis (`set -x`, `env`, `echo` de secret).
- [ ] timeout definido por job.
- [ ] runner correto (CI sem acesso a PROD).
- [ ] PR externa/fork tratada (sem secrets, sem `pull_request_target` perigoso).
- [ ] artifact protegido (retenção, acesso, assinatura quando aplicável).
- [ ] OIDC em vez de credencial estática de nuvem, quando aplicável.

---

## 99. security checklist runner

- [ ] usuário dedicado.
- [ ] SSH por chave dedicada, nunca chave pessoal.
- [ ] firewall.
- [ ] patches em dia.
- [ ] Docker controlado (grupo docker tratado como privilégio).
- [ ] sem PROD no CI.
- [ ] disk monitoring.
- [ ] rebuild documentado.

---

## 100. security checklist deploy

- [ ] gate humano (aprovação de Environment).
- [ ] chave dedicada (nunca root, nunca pessoal).
- [ ] environment secrets segregados por ambiente.
- [ ] artifact imutável (tag por digest).
- [ ] registry least privilege (CI push, PROD pull).
- [ ] health check pós-deploy.
- [ ] rollback definido.
- [ ] audit (quem, quando, o quê).

---

## 101. branch protection e proteção de main — checklist rápido

- [ ] PR obrigatório para `main`.
- [ ] required status checks (CI verde antes do merge).
- [ ] required reviews (ao menos 1, ou conforme política do time).
- [ ] Require review from Code Owners, se `CODEOWNERS` existir.
- [ ] sem force push.
- [ ] sem exclusão de branch.
- [ ] conversas resolvidas antes do merge.

---

## 102. arquitetura alvo

```text
GitHub
 |
 v
CI Runner
 |
 +-- read code
 +-- tests
 +-- build
 +-- scan (SAST, SCA, container)
 +-- sign (cosign) + SBOM (Syft)
 |
 v
Registry
 |
 v
Deploy Runner
 |
 +-- read artifact (verifica assinatura)
 +-- controlled SSH / OIDC para nuvem
 |
 v
PROD
```

---

## 103. próximo volume

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

## Fontes

### Tokens, permissions e secrets

- [Use GITHUB_TOKEN for authentication in workflows](https://docs.github.com/en/actions/security-guides/automatic-token-authentication) — comprova o comportamento do `GITHUB_TOKEN` e o uso do bloco `permissions` por workflow/job (seção 7).
- [Using secrets in GitHub Actions](https://docs.github.com/en/actions/security-guides/encrypted-secrets) — comprova o funcionamento dos GitHub Secrets (seções 8 a 10).

### Secret scanning

- [Secret scanning](https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning) — comprova o funcionamento do secret scanning nativo do GitHub (seção 31).
- [Push protection](https://docs.github.com/en/code-security/secret-scanning/push-protection-for-repositories-and-organizations) — comprova o bloqueio de push antes de um segredo entrar no histórico (seção 31).

### Hardening de Actions (pinning, pull_request_target, forks)

- [Secure use reference](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions) — comprova a recomendação de pinning por SHA de commit e os riscos de `pull_request_target`/`workflow_run` com checkout de PR não confiável (seções 23, 59, 60).
- [Events that trigger workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows) — comprova a definição e o comportamento do evento `pull_request_target` (seção 59).

### Dependências e SAST/SCA (Dependabot, CodeQL)

- [Dependabot options reference](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/dependabot-options-reference) — comprova as opções de configuração do `dependabot.yml` (seção 24).
- [Code scanning with CodeQL](https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning-with-codeql) — comprova o funcionamento do CodeQL como SAST nativo do GitHub (seção 34).

### Container scan (Trivy/Grype)

- [Trivy — The All-in-One Security Scanner](https://trivy.dev/) — comprova o escopo do Trivy (imagem, filesystem, dependências, configuração) usado nas seções 29, 36 e 37.
- [Grype (anchore/grype)](https://github.com/anchore/grype) — comprova o escopo do Grype como scanner de vulnerabilidades para imagens e SBOMs (seções 29, 36, 38).

### Assinatura e proveniência (Cosign/Sigstore, SBOM, attestations)

- [Cosign — Keyless signing overview (Sigstore docs)](https://docs.sigstore.dev/cosign/signing/overview/) — comprova o fluxo de keyless signing via OIDC usado na seção 51.
- [Syft (anchore/syft)](https://github.com/anchore/syft) — comprova o papel do Syft na geração de SBOM (seções 52, 53).
- [Using artifact attestations to establish provenance for builds](https://docs.github.com/en/actions/security-guides/using-artifact-attestations-to-establish-provenance-for-builds) — comprova o uso de `actions/attest-build-provenance` para proveniência nativa (seção 54).

### OIDC para nuvem

- [OpenID Connect](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect) — comprova o uso de OIDC do GitHub Actions para autenticação com AWS/Azure/GCP em vez de credenciais estáticas (seção 55).

### Branch protection e CODEOWNERS

- [About rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) — comprova o mecanismo de rulesets como evolução da proteção de branch (seções 56, 101).
- [About code owners](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners) — comprova o funcionamento do arquivo `CODEOWNERS` e da revisão obrigatória de code owners (seção 57).

### OWASP CI/CD Security

- [OWASP Top 10 CI/CD Security Risks](https://owasp.org/www-project-top-10-ci-cd-security-risks/) — referência geral para o modelo de ameaças e as superfícies de ataque tratadas neste volume (seções 3, 4, 25).

---

**Fim do Volume 10 — Segurança do Pipeline CI/CD**
