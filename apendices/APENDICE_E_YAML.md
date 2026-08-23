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
