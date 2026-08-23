# Apêndice A — Comandos git: referência operacional

## Objetivo

Cheat sheet dos comandos utilizados ao longo do guia.

## Estado

| Comando | Descrição |
|---|---|
| `git status` | Mostra o estado atual do working directory e da staging area |
| `git log --oneline --graph --decorate --all` | Histórico compacto com grafo de branches |
| `git diff` | Diferenças não staged |
| `git diff --staged` | Diferenças já staged |

## Branches

| Comando | Descrição |
|---|---|
| `git branch` | Lista branches locais |
| `git switch main` | Troca para a branch `main` |
| `git switch -c feature/minha-feature` | Cria e troca para uma nova branch |
| `git branch -d feature/minha-feature` | Remove uma branch local já mesclada |

## Staging e commits

| Comando | Descrição |
|---|---|
| `git add arquivo` | Adiciona um arquivo à staging area |
| `git add .` | Adiciona todas as alterações à staging area |
| `git restore --staged arquivo` | Remove um arquivo da staging area |
| `git commit -m "feat: descrição"` | Cria um commit com mensagem |

## Remotos

| Comando | Descrição |
|---|---|
| `git remote -v` | Lista os remotos configurados |
| `git fetch origin` | Busca atualizações do remoto sem aplicar |
| `git pull` | Busca e aplica atualizações do remoto |
| `git push` | Envia commits para o remoto |
| `git push -u origin feature/minha-feature` | Envia a branch e define o upstream |

## Merge e rebase

| Comando | Descrição |
|---|---|
| `git merge feature/x` | Mescla a branch `feature/x` na branch atual |
| `git rebase main` | Reaplica os commits da branch atual sobre `main` |
| `git rebase --continue` | Continua um rebase após resolver conflitos |
| `git rebase --abort` | Cancela o rebase em andamento |

## Stash

| Comando | Descrição |
|---|---|
| `git stash` | Guarda alterações não commitadas |
| `git stash list` | Lista os stashes guardados |
| `git stash pop` | Restaura o último stash e o remove da lista |

## Tags

| Comando | Descrição |
|---|---|
| `git tag` | Lista as tags existentes |
| `git tag -a v1.0.0 -m "Release v1.0.0"` | Cria uma tag anotada |
| `git push origin v1.0.0` | Envia uma tag para o remoto |

## Desfazer

| Comando | Descrição |
|---|---|
| `git restore arquivo` | Descarta alterações locais de um arquivo |
| `git revert HASH` | Cria um commit que desfaz outro commit |
| `git reset --soft HEAD~1` | Desfaz o último commit mantendo as alterações staged |

Use `reset --hard` somente quando compreender que alterações locais podem ser destruídas.

## Diagnóstico

| Comando | Descrição |
|---|---|
| `git show HASH` | Mostra os detalhes de um commit |
| `git blame arquivo` | Mostra quem alterou cada linha de um arquivo |
| `git reflog` | Mostra o histórico de referências, incluindo commits "perdidos" |

## Bisect

| Comando | Descrição |
|---|---|
| `git bisect start` | Inicia uma busca binária por um commit problemático |
| `git bisect bad` | Marca o commit atual como ruim |
| `git bisect good HASH` | Marca um commit específico como bom |
| `git bisect reset` | Encerra a busca e retorna ao estado original |

## Limpeza

| Comando | Descrição |
|---|---|
| `git branch --merged` | Lista branches já mescladas |
| `git fetch --prune` | Remove referências de branches remotas que não existem mais |

## Worktrees

Úteis em CI/CD para manter múltiplas branches (ex.: build de release + hotfix) sem clonar o repositório várias vezes.

| Comando | Descrição |
|---|---|
| `git worktree add ../app-hotfix hotfix/x` | Cria um novo worktree para a branch `hotfix/x` |
| `git worktree add -b feature/y ../app-y origin/main` | Cria um novo worktree com uma nova branch a partir de `origin/main` |
| `git worktree list` | Lista os worktrees existentes |
| `git worktree remove ../app-hotfix` | Remove um worktree |
| `git worktree prune` | Limpa referências de worktrees removidos manualmente |

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
