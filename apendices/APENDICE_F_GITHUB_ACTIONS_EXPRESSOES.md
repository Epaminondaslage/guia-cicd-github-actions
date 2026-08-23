# Apêndice F — Expressões e contextos do GitHub Actions

## Sintaxe

```yaml
${{ ... }}
```

## Contexto github

Exemplos frequentes:

```yaml
${{ github.sha }}
${{ github.ref }}
${{ github.actor }}
${{ github.event_name }}
${{ github.repository }}
${{ github.run_id }}
```

## Secrets

```yaml
${{ secrets.DB_PASSWORD }}
```

## Variables

```yaml
${{ vars.DEV_HOST }}
```

## Inputs

```yaml
${{ inputs.version }}
```

## Matrix

```yaml
${{ matrix.node }}
```

## Needs

```yaml
${{ needs.build.result }}
${{ needs.build.outputs.versao }}
```

## Steps e env

```yaml
${{ steps.meu-id.outputs.valor }}
${{ steps.meu-id.outcome }}
${{ env.MINHA_VAR }}
```

## Funções úteis

| Função | Exemplo |
|---|---|
| `contains` | `${{ contains(github.event.head_commit.message, '[skip ci]') }}` |
| `startsWith` | `${{ startsWith(github.ref, 'refs/tags/') }}` |
| `endsWith` | `${{ endsWith(github.ref, '/main') }}` |
| `format` | `${{ format('{0}-{1}', github.sha, github.run_number) }}` |
| `toJSON` | `${{ toJSON(github.event) }}` |
| `fromJSON` | `${{ fromJSON(steps.meu-id.outputs.json) }}` |
| `join` | `${{ join(github.event.commits.*.message, ', ') }}` |

## Conditions

```yaml
if: github.ref == 'refs/heads/main'
```

## Funções de status

| Função | Uso |
|---|---|
| `success()` | `if: success()` |
| `failure()` | `if: failure()` |
| `always()` | `if: always()` |
| `cancelled()` | `if: cancelled()` |

## Exemplo de cleanup

```yaml
- name: Cleanup
  if: always()
  run: docker compose down -v
```

## Concurrency

```yaml
concurrency:
  group: ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

## SHA como versão

```yaml
- run: docker build -t app:${{ github.sha }} .
```

## Environment

```yaml
environment:
  name: production
```

## Comparações

```yaml
if: github.event_name == 'push'
```

## Operadores

Expressões suportam operadores lógicos e comparações. Mantenha condições simples e legíveis.

## Cuidado com conteúdo não confiável

Valores vindos de Issues, PRs, branches e payloads podem conter conteúdo controlado por usuários. Evite interpolar dados não confiáveis diretamente em shell.

Prefira passar dados por `env` e tratá-los como strings.

## Debug

Ao depurar contextos, não imprima objetos inteiros que possam conter dados sensíveis.

## Regra prática

Uma expressão deve tomar decisão de orquestração; lógica de negócio complexa pertence a scripts/código testável.

## Fontes

- [Evaluate expressions in workflows and actions](https://docs.github.com/en/actions/learn-github-actions/expressions) — sintaxe `${{ }}`, operadores e funções `contains`, `startsWith`, `endsWith`, `format`, `toJSON`, `fromJSON`, `join`, além de `success()`, `failure()`, `always()`, `cancelled()`.
- [Access contextual information about workflow runs](https://docs.github.com/en/actions/learn-github-actions/contexts) — contextos `github`, `secrets`, `vars`, `inputs`, `matrix`, `needs`, `steps`, `env`.
- [Control the concurrency of workflows and jobs](https://docs.github.com/en/actions/using-jobs/using-concurrency) — `concurrency.group` e `cancel-in-progress`.

**Fim do Apêndice F**
</content>
