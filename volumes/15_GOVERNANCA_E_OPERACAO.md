# Volume 15 — Governança e Operação do Ciclo de Desenvolvimento

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 15_GOVERNANCA_E_OPERACAO.md  
**Versão:** 0.1.0

---

## 1. Objetivo

Transformar práticas técnicas em regras operacionais consistentes.

Sem governança:

```text
cada PR funciona de um jeito
cada deploy depende da memória
cada incidente gera improviso
```

Com governança:

```text
processo previsível
rastreável
repetível
```

---

## 2. Política de branches

Modelo inicial:

```text
main
 |
 +-- feature/*
 +-- fix/*
 +-- refactor/*
 +-- docs/*
```

`main` representa código integrado.

---

## 3. Push direto

Recomendação:

```text
main protegida
```

Alterações entram por PR.

No GitHub, isso é aplicado por **regras de proteção de branch**. Existem hoje duas interfaces para o mesmo mecanismo:

```text
Branch protection rules (clássico)
Rulesets (repositório ou organização)
```

Rulesets são a interface recomendada atualmente: permitem combinar múltiplos padrões de branch/tag em uma única regra, aplicar em nível de organização (várias repos de uma vez), têm modo "evaluate" (dry-run, só reporta o que bloquearia) e bypass list granular por time/app. Branch protection clássico continua funcionando, mas novas contas devem preferir Rulesets.

Configuração mínima recomendada para `main`:

```text
require pull request before merging
require approvals (>= 1)
dismiss stale approvals on new commits
require status checks to pass
require branches to be up to date before merging
require conversation resolution before merging
block force pushes
restrict deletions
```

Em repositórios com CODEOWNERS, adicione também "Require review from Code Owners" (ver seção 46).

---

## 4. Branch curta

Branches devem viver pelo menor tempo razoável.

Branches longas acumulam divergência.

---

## 5. Política de PR

Toda mudança relevante deve:

- possuir objetivo claro;
- explicar contexto;
- passar CI;
- atender critérios;
- manter escopo;
- possuir estratégia de teste.

---

## 6. Tamanho de PR

Não definir limite rígido universal.

Avaliar:

```text
complexidade cognitiva
risco
capacidade de revisão
```

---

## 7. Draft

Use Draft quando implementação ainda está em andamento.

---

## 8. Review

Review deve verificar:

- requisito;
- arquitetura;
- segurança;
- testes;
- legibilidade;
- impacto operacional.

---

## 9. Required checks

Defina como obrigatórios apenas checks confiáveis.

Exemplo:

```text
lint
unit
integration
smoke E2E
```

Isso é diferente de **required reviews** (revisão humana obrigatória), configurada na mesma regra de proteção/ruleset. Os dois são independentes e complementares:

| Mecanismo | Efeito |
|---|---|
| Required status checks | CI verde |
| Required reviews | Aprovação humana (N pessoas) |
| CODEOWNERS review | Aprovação de dono da área tocada |

Uma PR só pode ser mesclada quando **todos** os requisitos ativos forem satisfeitos. Evite marcar checks flaky como obrigatórios: isso trava merges legítimos e incentiva bypass da proteção.

---

## 10. Definition of Ready

Antes de desenvolver:

- problema compreendido;
- SPEC suficiente;
- critérios definidos;
- dependências conhecidas.

---

## 11. Definition of Done

Antes de concluir:

- código;
- testes;
- documentação;
- CI PASS;
- E2E;
- review;
- DEV validado quando aplicável.

---

## 12. Política de merge

Escolha estratégia padrão.

Sugestão inicial:

```text
Squash and merge
```

para features.

Documente exceções.

---

## 13. Commit convention

```text
feat:
fix:
test:
docs:
refactor:
ci:
chore:
```

---

## 14. Releases

Use versão explícita.

Exemplo:

```text
v2.4.1
```

Tags e GitHub Releases podem ser geradas manualmente ou automatizadas por workflow. Duas ferramentas comuns nesse cenário:

| Ferramenta | Efeito |
|---|---|
| semantic-release | Calcula versão a partir de conventional commits, publica tag/release/changelog |
| release-please | Mantém uma PR de release aberta; ao mesclar, cria tag/release |

Ambas dependem de mensagens de commit padronizadas (ver seção 13) para decidir se o bump é MAJOR, MINOR ou PATCH.

---

## 15. Semantic Versioning

```text
MAJOR.MINOR.PATCH
```

| Nível | Significado |
|---|---|
| MAJOR | Mudança incompatível (breaking change) |
| MINOR | Funcionalidade nova, compatível |
| PATCH | Correção, compatível |

Use quando compatível com o produto. Com conventional commits, o mapeamento típico é:

| Prefixo do commit | Bump |
|---|---|
| `fix:` | PATCH |
| `feat:` | MINOR |
| `BREAKING CHANGE:` / `feat!:` / `fix!:` | MAJOR |

Pré-releases (`v2.4.1-rc.1`) e metadados de build (`v2.4.1+build.5`) seguem a mesma especificação (semver.org) quando necessário.

---

## 16. Changelog

Registre mudanças relevantes ao usuário/operação.

Não listar todo commit técnico necessariamente.

Formato recomendado: [Keep a Changelog](https://keepachangelog.com), com seções fixas por versão:

```text
Added
Changed
Deprecated
Removed
Fixed
Security
```

Quando os commits seguem conventional commits, o changelog pode ser gerado automaticamente (mesma automação da seção 14: semantic-release ou release-please mantêm o `CHANGELOG.md` sincronizado com cada release). Edição manual continua válida para clarear texto destinado ao usuário final.

---

## 17. Release notes

Inclua:

- features;
- fixes;
- migrations;
- riscos;
- rollback.

---

## 18. Política de deploy

```text
main -> DEV
DEV validated -> approval
approval -> PROD
```

### 18.1 Auditoria de deploy

Todo deploy deve responder "quem, quando, o quê, para onde":

| Pergunta | Resposta esperada |
|---|---|
| Quem | Disparou (usuário ou automação) |
| Quando | Timestamp |
| O que | Foi implantado (commit/tag) |
| Para onde | Qual ambiente |
| Resultado | Sucesso/falha |

No GitHub isso é coberto por três mecanismos complementares, não excludentes:

| Mecanismo | Descrição |
|---|---|
| Environments | Histórico de deployments |
| Deployments API | `GET /repos/{owner}/{repo}/deployments` |
| Audit log | Da organização |

Ao usar **Environments** (Settings > Environments) em cada job de deploy do workflow (`environment: producao`), o GitHub registra automaticamente um deployment vinculado ao run, ao actor e ao commit, visível na aba "Environments" do repositório e consultável pela API/GraphQL. Isso é a fonte de verdade de "o que está rodando em cada ambiente agora" e do histórico de quem promoveu cada versão.

Regras de proteção de Environment (reviewers obrigatórios, wait timer, branches permitidas) também servem como o "approval" da política acima: só quem está na lista de reviewers do Environment pode aprovar a promoção para PROD, e essa aprovação fica registrada.

O **audit log** (organização, planos Team/Enterprise) complementa registrando quem alterou configurações sensíveis — proteção de branch, secrets, membros, Environments — não o conteúdo do deploy em si.

### 18.2 Autenticação do runner: prefira OIDC a secrets de longa duração

Quando o deploy precisa autenticar em um provedor externo (nuvem, registry, servidor), evite armazenar chave/senha de longa duração como secret do repositório. Prefira **OpenID Connect (OIDC)**: o GitHub Actions emite um token de curta duração assinado, que o provedor troca por credenciais temporárias.

```text
workflow solicita ID token do GitHub
provedor valida claims (repo, branch, environment)
provedor emite credencial de curta duração
job usa a credencial e ela expira
```

Vantagens: nada fica salvo em segredo permanente para vazar, e o próprio provedor externo passa a registrar por qual repositório/branch/environment cada acesso foi originado — reforçando a trilha de auditoria da seção 18.1.

Para infraestrutura própria sem suporte nativo a OIDC (por exemplo, um runner self-hosted acessando um servidor via SSH), a alternativa é reduzir o TTL do que for possível e aplicar a rotação da seção 37.

---

## 19. Janela de deploy

Mesmo com automação, algumas mudanças podem exigir janela adequada.

Exemplo:

```text
migration grande
mudança de rede
```

---

## 20. Freeze

Em períodos críticos, pode existir freeze temporário.

Exceções devem ser explícitas.

---

## 21. Emergency change

Fluxo de emergência deve continuar rastreável.

Não significa:

```text
editar PROD sem registro
```

---

## 22. Hotfix

Exemplo:

```text
main
 |
 v
fix/critical
 |
 v
PR rápida
 |
 v
CI essencial
 |
 v
deploy
```

---

## 23. Não pular testes sem justificativa

Emergência pode reduzir conjunto, mas decisão precisa ser registrada.

---

## 24. Rollback first

Em incidente grave após deploy:

```text
se rollback seguro for mais rápido
restaure primeiro
investigue depois
```

---

## 25. Incidente

Defina severidade:

```text
SEV1
SEV2
SEV3
```

conforme impacto.

---

## 26. Incident commander

Em equipes maiores, uma pessoa coordena.

Evita múltiplas mudanças simultâneas.

---

## 27. Timeline

Registre:

```text
14:32 deploy
14:35 erro subiu
14:38 rollback
14:42 normalizado
```

---

## 28. Post-mortem

Estrutura:

```text
Resumo
Impacto
Timeline
Causa
Fatores contribuintes
Detecção
Resposta
Ações
```

---

## 29. Blameless

O objetivo é corrigir sistema/processo, não procurar culpado.

Ainda existe responsabilidade técnica.

---

## 30. Action items

Devem possuir:

- responsável;
- prioridade;
- prazo;
- Issue.

---

## 31. Runbooks

Operações recorrentes devem ter documentação.

Exemplos:

```text
runner offline
disk full
rollback
database restore
certificate renewal
```

---

## 32. Change management

Mudanças de infraestrutura também passam por PR quando possível.

---

## 33. Ownership

Defina responsáveis por:

```text
application
CI
runner
database
production
```

Mesmo em equipe pequena, clareza ajuda.

---

## 34. Bus factor

Documentação reduz dependência de uma única pessoa.

---

## 35. Access review

Periodicamente reveja:

- GitHub collaborators;
- SSH keys;
- tokens;
- production access;
- registry.

Onde for possível, prefira OIDC (seção 18.2) a tokens de longa duração: reduz a superfície da própria revisão, já que não há credencial permanente para revogar — apenas a confiança configurada no provedor externo (repo/branch/environment autorizados).

---

## 36. Offboarding

Ao remover alguém:

```text
revogar access
tokens
SSH
secrets compartilhados
```

---

## 37. Secret rotation policy

Defina periodicidade conforme criticidade.

Rotacione imediatamente em exposição.

---

## 38. Dependency management

Tenha rotina para PRs de atualização.

Não deixar centenas acumularem.

---

## 39. Technical debt

Registre como Issues.

Não misture dívida técnica arbitrariamente em qualquer feature.

---

## 40. Architecture governance

Mudança relevante:

```text
ADR
```

---

## 41. ADR lifecycle

Estados possíveis:

```text
proposed
accepted
superseded
deprecated
```

---

## 42. Documentation review

Docs importantes devem mudar junto com sistema.

---

## 43. README

Deve responder:

```text
o que é?
como executar?
como testar?
onde está documentação?
```

---

## 44. CONTRIBUTING

Documente:

- branches;
- commits;
- PR;
- testes;
- code style.

---

## 45. SECURITY.md

Pode documentar como reportar vulnerabilidades.

---

## 46. CODEOWNERS

Use em áreas sensíveis se equipe crescer.

Arquivo `CODEOWNERS`, em `.github/CODEOWNERS`, `CODEOWNERS` (raiz) ou `docs/CODEOWNERS`. Sintaxe (padrões `.gitignore`, avaliados de cima para baixo — a **última** linha que casa vence):

```text
# comentário
*                       @time-padrao
/apps/api/               @time-backend
/apps/web/               @time-frontend
*.tf                    @time-infra
/.github/workflows/      @time-plataforma @fulano
/apps/api/**/migrations/ @time-backend @dba
```

Donos podem ser `@usuario`, `@org/time` (o time precisa ter permissão de escrita/leitura no repo) ou e-mail associado a conta do GitHub.

Isso só passa a bloquear merge quando combinado com a proteção de branch/ruleset da seção 3: ative "Require review from Code Owners" (clássico) ou a regra equivalente em Rulesets. Sem essa opção marcada, o CODEOWNERS apenas sugere revisores automaticamente — não impede merge sem a aprovação deles.

Cuidados comuns:

```text
arquivo precisa estar em main (ou branch padrão) para valer
usuário/time listado precisa ter acesso ao repositório
padrões mais específicos devem vir depois dos genéricos
```

---

## 47. Templates

Padronize:

- Issue bug;
- feature;
- PR;
- incident;
- ADR.

---

## 48. AI governance

Defina o que agentes podem fazer autonomamente.

---

## 49. IA pode

Exemplo:

- criar branch;
- editar código;
- rodar testes;
- abrir PR.

---

## 50. IA não pode sem gate

Exemplo:

- merge crítico;
- apagar dados;
- deploy PROD;
- rotacionar credencial;
- alterar firewall.

---

## 51. Generated code review

Código gerado por IA recebe mesmo padrão de revisão.

Não existe exceção porque "foi a IA".

---

## 52. Prompt logging

Não registre prompts contendo secrets.

---

## 53. AI provenance

Quando útil, PR pode indicar que implementação foi assistida por IA.

Mais importante é manter SPEC, diff e testes.

---

## 54. Policy as code

Algumas políticas podem ser automatizadas:

```text
branch protection
required checks
lint
security scans
```

---

## 55. Manual policy

Outras dependem de julgamento:

```text
UX
risco de negócio
janela de deploy
```

---

## 56. KPI de engenharia

Possíveis:

- lead time;
- deployment frequency;
- change failure rate;
- MTTR.

São métricas DORA conhecidas.

---

## 57. Não gamificar métricas

Métrica vira inútil quando vira meta manipulável.

Use para aprender.

---

## 58. Change failure rate

Percentual de deploys que causam:

- rollback;
- incidente;
- hotfix.

---

## 59. Deployment frequency

Frequência por si só não é qualidade.

Contextualize.

---

## 60. Lead time for changes

Tempo da mudança até produção.

---

## 61. MTTR

Tempo de recuperação.

---

## 62. Quarterly review

Periodicamente revise:

```text
CI duration
security
runners
dependencies
incidents
docs
```

---

## 63. Capacity review

Verifique se hardware continua adequado.

---

## 64. Cost review

Compare:

```text
self-hosted cost
hosted minutes
maintenance
```

---

## 65. Backup review

Confirme:

- backups executando;
- restores testados;
- retenção.

---

## 66. Disaster recovery

Documente perda de:

```text
runner
DEV
PROD
database
registry
```

---

## 67. RTO

Recovery Time Objective:

```text
tempo máximo desejado de recuperação
```

---

## 68. RPO

Recovery Point Objective:

```text
quantidade aceitável de perda de dados
```

---

## 69. CI RTO

Se runner falhar, quanto tempo podemos ficar sem novos builds?

Isso orienta redundância.

---

## 70. PROD RTO/RPO

Muito mais importantes que CI.

---

## 71. Governance checklist

- [ ] Branch policy.
- [ ] PR template.
- [ ] Required checks.
- [ ] Merge strategy.
- [ ] Deploy policy.
- [ ] Approval.
- [ ] Rollback.
- [ ] Incident process.
- [ ] Access review.
- [ ] Backup review.
- [ ] AI permissions.
- [ ] Architecture decisions.

---

## 72. Próximo volume

**Volume 16 — Laboratório Completo: do Repositório à Produção**

---

## Fontes

### Branches, PRs e CODEOWNERS

- [About rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) — comprova bypass list granular e aplicação em nível de organização (seção 3); modo "evaluate" citado no texto não aparece explicitamente nesta página e deve ser tratado como detalhe operacional adicional a confirmar na interface.
- [About protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches) — sustenta a configuração mínima de proteção de `main` (required status checks, required reviews, CODEOWNERS review) da seção 3 e a distinção required checks x required reviews da seção 9.
- [About code owners](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners) — confirma local do arquivo (`.github/CODEOWNERS`, raiz, `docs/`), sintaxe baseada em `.gitignore`, regra "última linha que casa vence" e a necessidade de combinar com "Require review from Code Owners" (seção 46).

### Commits, versionamento e changelog

- [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) — especifica os prefixos `feat:`/`fix:` e a marcação de breaking change (`!` ou `BREAKING CHANGE:`), base da convenção de commit da seção 13 e do mapeamento de bump da seção 15.
- [Semantic Versioning 2.0.0](https://semver.org/) — define formalmente MAJOR.MINOR.PATCH e a sintaxe de pré-release/build metadata (seção 15).
- [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) — origem das seções fixas Added/Changed/Deprecated/Removed/Fixed/Security usadas na seção 16.

### Deploy, ambientes e auditoria

- [Using environments for deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment) — comprova required reviewers, wait timer e restrição de branches em Environments, base do "approval" e da auditoria de deploy da seção 18.1.
- [REST API: Deployments](https://docs.github.com/en/rest/deployments/deployments) — confirma o endpoint `GET /repos/{owner}/{repo}/deployments` citado na seção 18.1.
- [Reviewing the audit log for your organization](https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/reviewing-the-audit-log-for-your-organization) — sustenta o uso do audit log para registrar mudanças em proteção de branch, secrets e membros (seção 18.1); a página não restringe explicitamente a planos Team/Enterprise, ponto que vale revalidar antes de reafirmar essa restrição no texto.
- [About security hardening with OpenID Connect](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect) — confirma a troca de secrets de longa duração por tokens OIDC de curta duração validados por claims (repo/branch/environment), base da seção 18.2 e da recomendação da seção 35.

### Métricas de engenharia

- [DORA — Guides: DORA Metrics](https://dora.dev/guides/dora-metrics-four-keys/) — fonte oficial do programa DORA para deployment frequency, lead time for changes, change failure rate e MTTR (hoje evoluído para um modelo de cinco métricas), citadas na seção 56.

**Fim do Volume 15 — Governança e Operação**
