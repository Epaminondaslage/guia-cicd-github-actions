# Apêndice H — Checklists Consolidados

## Nova SPEC
- [ ] Problema definido.
- [ ] Objetivo definido.
- [ ] Escopo.
- [ ] Fora de escopo.
- [ ] Requisitos.
- [ ] Critérios de aceitação.
- [ ] Casos de teste.
- [ ] Riscos.
- [ ] Referências a Issues/PRs anteriores.

## Nova branch
- [ ] Partiu da base correta.
- [ ] Nome claro.
- [ ] Escopo único.
- [ ] Working tree limpo antes de iniciar.

## Antes do commit
- [ ] Diff revisado.
- [ ] Sem secrets.
- [ ] Testes pertinentes.
- [ ] Arquivos temporários removidos.
- [ ] Mensagem de commit clara.

## Antes do push
- [ ] Branch correta.
- [ ] Commits corretos.
- [ ] Lockfiles consistentes.
- [ ] Testes locais rápidos.
- [ ] Nenhum artifact acidental.

## Nova pull request
- [ ] Título claro.
- [ ] SPEC citada.
- [ ] Objetivo.
- [ ] Alterações.
- [ ] Como testar.
- [ ] Critérios.
- [ ] Riscos.
- [ ] PR anterior referenciada quando for refinamento.

## Merge
- [ ] Required checks PASS.
- [ ] Review concluído.
- [ ] Conversas resolvidas.
- [ ] Sem conflitos.
- [ ] Merge strategy correta.
- [ ] Branch pode ser removida.

## Novo workflow
- [ ] Trigger correto (`on:` restrito ao evento/branch/path necessário).
- [ ] `permissions:` mínimas declaradas no topo do workflow ou do job.
- [ ] Runner/labels corretos (`runs-on:` casa com labels do runner self-hosted).
- [ ] `timeout-minutes` definido no job.
- [ ] `concurrency` com `cancel-in-progress` quando aplicável.
- [ ] Dependências reproduzíveis (lockfile commitado, `npm ci` em vez de `npm install`).
- [ ] Cleanup garantido com `if: always()`.
- [ ] Artifacts relevantes publicados (`actions/upload-artifact`).
- [ ] Sem secrets em logs (nenhum `echo`/`run` que imprima valor de secret).
- [ ] Testado em branch/PR antes de mesclar na branch principal.

## Novo self-hosted runner
- [ ] Ubuntu atualizado.
- [ ] Usuário dedicado.
- [ ] Docker instalado.
- [ ] Runner registrado.
- [ ] Labels.
- [ ] systemd.
- [ ] Monitoramento de disco.
- [ ] SSH seguro.
- [ ] Firewall.
- [ ] Rebuild documentado.
- [ ] Não compartilha PROD sem justificativa.

## CI
- [ ] Lint.
- [ ] Unit.
- [ ] Integration.
- [ ] Build.
- [ ] Smoke E2E.
- [ ] Required checks.
- [ ] Dados isolados.
- [ ] Sem PROD secrets.
- [ ] Flaky controlado.

## Dockerfile
- [ ] Base confiável.
- [ ] Versão controlada.
- [ ] `.dockerignore`.
- [ ] Multi-stage quando útil.
- [ ] Cache adequado.
- [ ] Usuário não-root.
- [ ] Sem secrets.
- [ ] Artifact identificável por SHA.

## Compose de teste
- [ ] Banco descartável.
- [ ] Healthchecks.
- [ ] Projeto único por execução.
- [ ] Sem portas fixas desnecessárias.
- [ ] Logs antes do cleanup.
- [ ] `down -v` em cleanup.

## Deploy DEV
- [ ] Artifact oficial.
- [ ] Versão explícita.
- [ ] Deploy serializado.
- [ ] Config validada.
- [ ] Health.
- [ ] Smoke.
- [ ] E2E relacionado.
- [ ] Validação visual.

## Aprovação PROD
- [ ] DEV validado.
- [ ] Versão correta.
- [ ] CI/E2E PASS.
- [ ] Migrations conhecidas.
- [ ] Rollback conhecido.
- [ ] Janela adequada.
- [ ] Sem incidente ativo incompatível.

## Deploy PROD
- [ ] Gate humano.
- [ ] Mesma imagem validada.
- [ ] Preflight.
- [ ] Backup quando necessário.
- [ ] Health.
- [ ] Smoke seguro.
- [ ] `/version`.
- [ ] Monitoramento.
- [ ] Registro de deploy.

## Rollback
- [ ] Versão anterior identificada.
- [ ] Artifact disponível.
- [ ] Banco compatível.
- [ ] Reaplicar versão.
- [ ] Health.
- [ ] Smoke.
- [ ] Registrar incidente.
- [ ] Preservar logs/evidências.

## Incidente
- [ ] Impacto identificado.
- [ ] Mudanças concorrentes bloqueadas.
- [ ] Timeline iniciada.
- [ ] Rollback avaliado.
- [ ] Serviço restaurado.
- [ ] Evidências preservadas.
- [ ] Post-mortem.
- [ ] Action items.

## Segurança periódica
- [ ] MFA obrigatório para todos os colaboradores.
- [ ] Colaboradores e permissões revisados (remover acesso de quem saiu/mudou de função).
- [ ] Tokens (PAT, deploy keys) com validade e escopo mínimo, sem tokens "clássicos" sem expiração.
- [ ] SSH keys rotacionadas/auditadas (`authorized_keys` sem chaves órfãs).
- [ ] Secrets sem uso removidos; nenhum secret plaintext em log/artifact.
- [ ] Dependências com `dependabot`/`npm audit`/`pip-audit` em dia.
- [ ] Base images com tag fixa (não `latest`) e sem CVEs críticas conhecidas (`docker scout`/`trivy`).
- [ ] Runner patches (SO e Docker Engine) aplicados.
- [ ] `permissions:` do `GITHUB_TOKEN` mínimas por workflow (evitar `write-all`).
- [ ] Actions de terceiros fixadas por SHA (não por tag mutável) quando crítico.
- [ ] Firewall/rede do runner revisados.
- [ ] Backup/restore testado (não só existente).

## Observabilidade
- [ ] Health.
- [ ] Version.
- [ ] Logs estruturados.
- [ ] CPU/RAM.
- [ ] Disk.
- [ ] Errors.
- [ ] Latency.
- [ ] Deployment markers.
- [ ] Alertas acionáveis.
- [ ] Runbooks.

## Revisão trimestral
- [ ] Duração do CI.
- [ ] Queue time.
- [ ] Flaky tests.
- [ ] Capacidade dos runners.
- [ ] Custos.
- [ ] Incidentes.
- [ ] Segurança.
- [ ] Dependências.
- [ ] Backups.
- [ ] Documentação.

## Fontes

- [Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions) — pin de actions por SHA e `permissions:` mínimas do `GITHUB_TOKEN`, base dos itens de "Novo workflow" e "Segurança periódica".
- [Workflow syntax for GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions) — `permissions`, `timeout-minutes`, `concurrency` e `runs-on` citados no checklist "Novo workflow".
- [Caching dependencies to speed up workflows](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows) — uso de lockfile e `npm ci` para dependências reproduzíveis.

**Fim do Apêndice H**
