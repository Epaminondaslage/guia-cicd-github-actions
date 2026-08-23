# Apêndice A — Comandos Git: Referência Operacional

## Objetivo
Cheat sheet dos comandos utilizados ao longo do guia.

## Estado
```bash
git status
git log --oneline --graph --decorate --all
git diff
git diff --staged
```

## Branches
```bash
git branch
git switch main
git switch -c feature/minha-feature
git branch -d feature/minha-feature
```

## Staging e commits
```bash
git add arquivo
git add .
git restore --staged arquivo
git commit -m "feat: descrição"
```

## Remotos
```bash
git remote -v
git fetch origin
git pull
git push
git push -u origin feature/minha-feature
```

## Merge e rebase
```bash
git merge feature/x
git rebase main
git rebase --continue
git rebase --abort
```

## Stash
```bash
git stash
git stash list
git stash pop
```

## Tags
```bash
git tag
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```

## Desfazer
```bash
git restore arquivo
git revert HASH
git reset --soft HEAD~1
```

Use `reset --hard` somente quando compreender que alterações locais podem ser destruídas.

## Diagnóstico
```bash
git show HASH
git blame arquivo
git reflog
```

## Bisect
```bash
git bisect start
git bisect bad
git bisect good HASH
git bisect reset
```

## Limpeza
```bash
git branch --merged
git fetch --prune
```

## Worktrees
Úteis em CI/CD para manter múltiplas branches (ex.: build de release + hotfix) sem clonar o repositório várias vezes.
```bash
git worktree add ../app-hotfix hotfix/x
git worktree add -b feature/y ../app-y origin/main
git worktree list
git worktree remove ../app-hotfix
git worktree prune
```

## Fluxo diário
```bash
git switch main
git pull
git switch -c feature/x
# editar
git add .
git commit -m "feat: implementa x"
git push -u origin feature/x
```

**Fim do Apêndice A**
