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

# 2. Política de branches

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

# 3. Push direto

Recomendação:

```text
main protegida
```

Alterações entram por PR.

---

# 4. Branch curta

Branches devem viver pelo menor tempo razoável.

Branches longas acumulam divergência.

---

# 5. Política de PR

Toda mudança relevante deve:

- possuir objetivo claro;
- explicar contexto;
- passar CI;
- atender critérios;
- manter escopo;
- possuir estratégia de teste.

---

# 6. Tamanho de PR

Não definir limite rígido universal.

Avaliar:

```text
complexidade cognitiva
risco
capacidade de revisão
```

---

# 7. Draft

Use Draft quando implementação ainda está em andamento.

---

# 8. Review

Review deve verificar:

- requisito;
- arquitetura;
- segurança;
- testes;
- legibilidade;
- impacto operacional.

---

# 9. Required checks

Defina como obrigatórios apenas checks confiáveis.

Exemplo:

```text
lint
unit
integration
smoke E2E
```

---

# 10. Definition of Ready

Antes de desenvolver:

- problema compreendido;
- SPEC suficiente;
- critérios definidos;
- dependências conhecidas.

---

# 11. Definition of Done

Antes de concluir:

- código;
- testes;
- documentação;
- CI PASS;
- E2E;
- review;
- DEV validado quando aplicável.

---

# 12. Política de merge

Escolha estratégia padrão.

Sugestão inicial:

```text
Squash and merge
```

para features.

Documente exceções.

---

# 13. Commit convention

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

# 14. Releases

Use versão explícita.

Exemplo:

```text
v2.4.1
```

---

# 15. Semantic Versioning

```text
MAJOR.MINOR.PATCH
```

Use quando compatível com o produto.

---

# 16. Changelog

Registre mudanças relevantes ao usuário/operação.

Não listar todo commit técnico necessariamente.

---

# 17. Release notes

Inclua:

- features;
- fixes;
- migrations;
- riscos;
- rollback.

---

# 18. Política de deploy

```text
main -> DEV
DEV validated -> approval
approval -> PROD
```

---

# 19. Janela de deploy

Mesmo com automação, algumas mudanças podem exigir janela adequada.

Exemplo:

```text
migration grande
mudança de rede
```

---

# 20. Freeze

Em períodos críticos, pode existir freeze temporário.

Exceções devem ser explícitas.

---

# 21. Emergency change

Fluxo de emergência deve continuar rastreável.

Não significa:

```text
editar PROD sem registro
```

---

# 22. Hotfix

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

# 23. Não pular testes sem justificativa

Emergência pode reduzir conjunto, mas decisão precisa ser registrada.

---

# 24. Rollback first

Em incidente grave após deploy:

```text
se rollback seguro for mais rápido
restaure primeiro
investigue depois
```

---

# 25. Incidente

Defina severidade:

```text
SEV1
SEV2
SEV3
```

conforme impacto.

---

# 26. Incident commander

Em equipes maiores, uma pessoa coordena.

Evita múltiplas mudanças simultâneas.

---

# 27. Timeline

Registre:

```text
14:32 deploy
14:35 erro subiu
14:38 rollback
14:42 normalizado
```

---

# 28. Post-mortem

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

# 29. Blameless

O objetivo é corrigir sistema/processo, não procurar culpado.

Ainda existe responsabilidade técnica.

---

# 30. Action items

Devem possuir:

- responsável;
- prioridade;
- prazo;
- Issue.

---

# 31. Runbooks

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

# 32. Change management

Mudanças de infraestrutura também passam por PR quando possível.

---

# 33. Ownership

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

# 34. Bus factor

Documentação reduz dependência de uma única pessoa.

---

# 35. Access review

Periodicamente reveja:

- GitHub collaborators;
- SSH keys;
- tokens;
- production access;
- registry.

---

# 36. Offboarding

Ao remover alguém:

```text
revogar access
tokens
SSH
secrets compartilhados
```

---

# 37. Secret rotation policy

Defina periodicidade conforme criticidade.

Rotacione imediatamente em exposição.

---

# 38. Dependency management

Tenha rotina para PRs de atualização.

Não deixar centenas acumularem.

---

# 39. Technical debt

Registre como Issues.

Não misture dívida técnica arbitrariamente em qualquer feature.

---

# 40. Architecture governance

Mudança relevante:

```text
ADR
```

---

# 41. ADR lifecycle

Estados possíveis:

```text
proposed
accepted
superseded
deprecated
```

---

# 42. Documentation review

Docs importantes devem mudar junto com sistema.

---

# 43. README

Deve responder:

```text
o que é?
como executar?
como testar?
onde está documentação?
```

---

# 44. CONTRIBUTING

Documente:

- branches;
- commits;
- PR;
- testes;
- code style.

---

# 45. SECURITY.md

Pode documentar como reportar vulnerabilidades.

---

# 46. CODEOWNERS

Use em áreas sensíveis se equipe crescer.

---

# 47. Templates

Padronize:

- Issue bug;
- feature;
- PR;
- incident;
- ADR.

---

# 48. AI governance

Defina o que agentes podem fazer autonomamente.

---

# 49. IA pode

Exemplo:

- criar branch;
- editar código;
- rodar testes;
- abrir PR.

---

# 50. IA não pode sem gate

Exemplo:

- merge crítico;
- apagar dados;
- deploy PROD;
- rotacionar credencial;
- alterar firewall.

---

# 51. Generated code review

Código gerado por IA recebe mesmo padrão de revisão.

Não existe exceção porque "foi a IA".

---

# 52. Prompt logging

Não registre prompts contendo secrets.

---

# 53. AI provenance

Quando útil, PR pode indicar que implementação foi assistida por IA.

Mais importante é manter SPEC, diff e testes.

---

# 54. Policy as code

Algumas políticas podem ser automatizadas:

```text
branch protection
required checks
lint
security scans
```

---

# 55. Manual policy

Outras dependem de julgamento:

```text
UX
risco de negócio
janela de deploy
```

---

# 56. KPI de engenharia

Possíveis:

- lead time;
- deployment frequency;
- change failure rate;
- MTTR.

São métricas DORA conhecidas.

---

# 57. Não gamificar métricas

Métrica vira inútil quando vira meta manipulável.

Use para aprender.

---

# 58. Change failure rate

Percentual de deploys que causam:

- rollback;
- incidente;
- hotfix.

---

# 59. Deployment frequency

Frequência por si só não é qualidade.

Contextualize.

---

# 60. Lead time for changes

Tempo da mudança até produção.

---

# 61. MTTR

Tempo de recuperação.

---

# 62. Quarterly review

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

# 63. Capacity review

Verifique se hardware continua adequado.

---

# 64. Cost review

Compare:

```text
self-hosted cost
hosted minutes
maintenance
```

---

# 65. Backup review

Confirme:

- backups executando;
- restores testados;
- retenção.

---

# 66. Disaster recovery

Documente perda de:

```text
runner
DEV
PROD
database
registry
```

---

# 67. RTO

Recovery Time Objective:

```text
tempo máximo desejado de recuperação
```

---

# 68. RPO

Recovery Point Objective:

```text
quantidade aceitável de perda de dados
```

---

# 69. CI RTO

Se runner falhar, quanto tempo podemos ficar sem novos builds?

Isso orienta redundância.

---

# 70. PROD RTO/RPO

Muito mais importantes que CI.

---

# 71. Governance checklist

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

# 72. Próximo volume

**Volume 16 — Laboratório Completo: do Repositório à Produção**

---

**Fim do Volume 15 — Governança e Operação**
