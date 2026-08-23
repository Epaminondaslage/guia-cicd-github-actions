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

## Nova Pull Request
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
- [ ] Trigger correto.
- [ ] `permissions` mínimas.
- [ ] Runner/labels corretos.
- [ ] Timeout.
- [ ] Concurrency.
- [ ] Dependências reproduzíveis.
- [ ] Cleanup.
- [ ] Artifacts.
- [ ] Sem secrets em logs.
- [ ] Testado em branch/PR.

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
- [ ] MFA.
- [ ] Colaboradores.
- [ ] Tokens.
- [ ] SSH keys.
- [ ] Secrets.
- [ ] Dependências.
- [ ] Base images.
- [ ] Runner patches.
- [ ] Firewall/rede.
- [ ] Backup/restore.

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

**Fim do Apêndice H**
