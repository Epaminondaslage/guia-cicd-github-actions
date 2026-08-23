# Apêndice B — GitHub CLI (`gh`): referência operacional

## Objetivo

Resumo dos comandos mais úteis da CLI oficial do GitHub.

## Autenticação

```bash
gh auth login
gh auth status
```

## Repositório

```bash
gh repo view
gh repo clone OWNER/REPO
gh repo list
```

## Pull requests

```bash
gh pr list
gh pr view 42
gh pr checks 42
gh pr diff 42
gh pr create
gh pr checkout 42
gh pr review 42 --approve
gh pr merge 42 --squash
```

## Issues

```bash
gh issue list
gh issue view 81
gh issue create
gh issue close 81
```

## Workflows e actions

```bash
gh workflow list
gh workflow view NOME
gh workflow run NOME.yml -f input1=valor
gh workflow enable NOME
gh workflow disable NOME
gh run list --workflow NOME.yml
gh run view RUN_ID --log
gh run view RUN_ID --log-failed
gh run watch RUN_ID
gh run rerun RUN_ID
gh run rerun RUN_ID --failed
gh run cancel RUN_ID
gh run download RUN_ID
```

## Secrets e variables

```bash
gh secret list
gh secret set NOME
gh secret set NOME --body "valor"
gh secret set NOME --env production
gh secret set NOME --org ORG --visibility all
gh secret delete NOME

gh variable list
gh variable set NOME --body "valor"
gh variable delete NOME
```

Nunca passe secrets em texto puro na linha de comando quando puder usar entrada via stdin/prompt (`gh secret set NOME < arquivo` ou digitando interativamente). `gh secret set` nunca expõe o valor de volta, apenas confirma a criação/atualização.

## Releases

```bash
gh release list
gh release view v1.0.0
gh release create v1.0.0
```

## API

```bash
gh api repos/OWNER/REPO
```

Use `gh api` com cuidado, especialmente em endpoints de escrita.

## Fluxo prático

```bash
git push -u origin feature/x
gh pr create --fill
gh pr checks --watch
```

## Fontes

- [GitHub CLI Manual](https://cli.github.com/manual/) — índice oficial dos comandos `gh` (repo, pr, issue, workflow, run, secret, variable, release, api).
- [gh secret set](https://cli.github.com/manual/gh_secret_set) — confirma entrada via stdin (`gh secret set MYSECRET < arquivo`) em vez de valor em texto puro na linha de comando.
- [gh workflow run](https://cli.github.com/manual/gh_workflow_run) — sintaxe de `gh workflow run NOME.yml -f input1=valor`.
- [gh run rerun](https://cli.github.com/manual/gh_run_rerun) — `gh run rerun RUN_ID` e `--failed`.

**Fim do Apêndice B**
