# Apêndice E — YAML: Referência Rápida para CI/CD

## Mapeamentos
```yaml
name: CI
enabled: true
```

## Listas
```yaml
branches:
  - main
  - develop
```

## Estrutura aninhada
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
```

## Strings
```yaml
value: texto
quoted: "texto"
single: 'texto'
```

## Bloco multilinha
```yaml
run: |
  npm ci
  npm test
```

## Booleanos
```yaml
enabled: true
```

## Comentários
```yaml
# comentário
```

## GitHub Actions
```yaml
name: CI

on:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test
```

## Cuidados de indentação

YAML depende de espaços.

Ruim:
```yaml
jobs:
 test:
    runs-on: ubuntu-latest
```

Prefira indentação consistente.

Não utilize TAB.

## Arrays inline
```yaml
runs-on: [self-hosted, linux, ci]
```

## env, jobs, needs e matrix
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      NODE_ENV: production
    steps:
      - uses: actions/checkout@v4
      - run: npm ci

  test:
    needs: build
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node: [18, 20, 22]
      fail-fast: false
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
```

## secrets no workflow
```yaml
steps:
  - name: Deploy
    env:
      TOKEN: ${{ secrets.DEPLOY_TOKEN }}
    run: ./deploy.sh
```

## Outputs entre steps e jobs
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      versao: ${{ steps.gerar.outputs.versao }}
    steps:
      - id: gerar
        run: echo "versao=1.2.3" >> "$GITHUB_OUTPUT"

  publish:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Versão ${{ needs.build.outputs.versao }}"
```

`$GITHUB_OUTPUT` substitui o antigo `::set-output` (removido/desativado pelo GitHub); sempre redirecione para o arquivo indicado pela variável de ambiente, nunca faça hardcode do caminho.

## Objetos inline
Possíveis em YAML, mas estruturas expandidas costumam ser mais legíveis em workflows.

## Valores que parecem números
Quando um valor precisa permanecer textual, use aspas.

## Anchors
YAML possui anchors/aliases, mas o suporte e a conveniência dentro de ferramentas específicas devem ser avaliados. Para GitHub Actions, reusable workflows e composite actions costumam ser alternativas mais explícitas para reutilização.

## Validação
Considere:
```text
yamllint
actionlint
```

## Regra prática
Se YAML ficou difícil de entender, mova lógica complexa para scripts versionados.

**Fim do Apêndice E**
