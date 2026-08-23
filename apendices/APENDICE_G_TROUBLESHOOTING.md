# Apêndice G — Troubleshooting de CI/CD

## 1. Workflow não inicia
Verifique:

```text
evento correto?
arquivo YAML na branch?
workflow habilitado?
filtro de branch/path?
```

## 2. Job fica em fila
Self-hosted:

```text
runner offline
runner ocupado
labels incompatíveis
runner group sem acesso
```

## 3. Runner offline
```bash
cd ~/actions-runner
sudo ./svc.sh status
ls -lt _diag | head
```

## 4. Docker permission denied
```bash
id
groups
docker version
```

Confirme grupo `docker` e nova sessão.

## 5. Docker daemon indisponível
```bash
systemctl status docker
journalctl -u docker
```

## 6. Disco cheio
```bash
df -h
df -i
docker system df
```

## 7. Memória insuficiente
```bash
free -h
docker stats
journalctl -k | grep -i oom
```

## 8. Funciona local, falha CI
Compare:

```text
runtime version
env vars
lockfile
timezone
filesystem
permissions
network
database
```

## 9. npm ci falha
Verifique:

```text
package-lock sincronizado?
Node correto?
registry acessível?
dependency script?
```

## 10. Composer falha
Verifique:

```text
composer.lock
PHP version
extensions
credentials privadas
```

## 11. Banco não conecta
Dentro de Compose, use nome do serviço:

```text
DB_HOST=db
```

não `localhost`, se banco está em outro container.

## 12. Banco ainda não está pronto
Use healthcheck/retry. Evite `sleep` arbitrário.

## 13. Porta ocupada
```bash
ss -lntp
docker ps
```

Evite portas fixas em jobs concorrentes.

## 14. E2E timeout
Investigue:

```text
selector
network
application logs
trace
screenshot
race
resource saturation
```

## 15. E2E flaky
Não apenas aumente retry. Reproduza e corrija causa.

## 16. Playwright browser ausente
Confirme instalação compatível:

```bash
npx playwright install --with-deps
```

## 17. Container reinicia
```bash
docker ps -a
docker logs CONTAINER
docker inspect CONTAINER
```

## 18. Health falha após deploy
Confirme:

```text
versão
config
DB
port
proxy
logs
migration
```

## 19. Deploy não puxa imagem
Verifique:

```bash
docker login REGISTRY
docker pull IMAGE:TAG
```

Sem imprimir token.

## 20. SSH deploy falha
Teste:

```bash
ssh -v usuario@host
```

Verifique chave, usuário, firewall e host key.

## 21. Migration falha
Pare deploy e preserve versão atual sempre que possível. Não improvise alterações manuais sem registro.

## 22. Rollback falha
Verifique compatibilidade do banco com versão anterior.

## 23. Git conflito
```bash
git status
git diff
```

Resolva conscientemente e teste.

## 24. Action de terceiro falha
Verifique release/changelog da Action antes de simplesmente alterar versão.

## 25. Workflow mudou de comportamento
Revise diff em `.github/workflows/` e permissões.

## 26. Secret vazio
Confirme:

```text
nome correto?
environment correto?
job tem acesso?
fork PR?
```

Não imprima valor.

## 27. Logs desapareceram no cleanup
Colete logs antes de `docker compose down -v`.

## 28. Queue crescente
Meça runner utilization. Talvez seja necessário adicionar runner.

## 29. CI ficou lento
Meça cada etapa; não otimize por palpite.

## 30. Diagnóstico padrão
Sempre registre:

```text
repo
PR
commit SHA
workflow
run
job
runner
step
erro
```

Isso torna investigação reproduzível.

**Fim do Apêndice G**
