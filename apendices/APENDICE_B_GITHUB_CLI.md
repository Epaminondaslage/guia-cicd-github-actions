# Apêndice B — GitHub CLI (`gh`): Referência Operacional

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

## Pull Requests
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

## Workflows e Actions
```bash
gh workflow list
gh workflow view
gh workflow run NOME
gh run list
gh run view RUN_ID
gh run watch RUN_ID
gh run rerun RUN_ID
```

## Secrets
```bash
gh secret list
gh secret set NOME
```

Nunca passe secrets em histórico de shell quando puder usar entrada segura.

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
gh pr create
gh pr checks
```

**Fim do Apêndice B**
