# Volume 01 — Fundamentos de Git e Controle de Versão

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 01_FUNDAMENTOS_GIT.md  
**Versão:** 0.1.0  
**Status:** Primeira versão para expansão incremental

---

## 1. Objetivo

Este volume apresenta os conceitos de Git necessários para compreender o restante do pipeline de CI/CD.

Ao final, o leitor deverá compreender a relação:

```text
Arquivos
   ↓
Working Tree
   ↓
Staging Area
   ↓
Commit
   ↓
Branch
   ↓
Repositório remoto
   ↓
Pull Request
```

Git e GitHub não são a mesma coisa.

**Git** é o sistema distribuído de controle de versão.

**GitHub** é uma plataforma que hospeda repositórios Git e acrescenta colaboração, Pull Requests, Issues, Actions, releases, permissões e outros serviços.

---

# 2. Repositório Git

Um repositório é um projeto cujo histórico é controlado pelo Git.

Criar:

```bash
mkdir meu-projeto
cd meu-projeto
git init
```

Verificar:

```bash
git status
```

O diretório oculto `.git` contém os metadados e o histórico do repositório.

Nunca altere manualmente seu conteúdo sem uma razão técnica específica.

---

# 3. Working Tree

A Working Tree é a cópia dos arquivos na qual o desenvolvedor trabalha.

Exemplo:

```text
projeto/
├── package.json
├── src/
│   └── server.js
└── tests/
    └── server.test.js
```

Ao editar `server.js`, o arquivo fica modificado, mas a alteração ainda não constitui um commit.

Verifique:

```bash
git status
```

---

# 4. Staging Area

A Staging Area é a área de preparação do próximo commit.

Adicionar um arquivo:

```bash
git add src/server.js
```

Adicionar vários:

```bash
git add src/ tests/
```

Adicionar todas as mudanças pertinentes:

```bash
git add .
```

Antes do commit, confira:

```bash
git status
git diff --staged
```

Fluxo:

```text
arquivo modificado
      |
      | git add
      v
staging area
      |
      | git commit
      v
histórico Git
```

---

# 5. Commit

Um commit representa um ponto identificado no histórico.

Exemplo:

```bash
git commit -m "feat: adiciona autenticação por token"
```

Um bom commit deve representar uma mudança coerente.

Evite:

```text
"alterações"
"teste"
"coisas novas"
"corrigindo"
```

Prefira mensagens como:

```text
feat: adiciona filtro por status
fix: corrige cálculo de paginação
test: adiciona cobertura para autenticação
docs: documenta instalação local
refactor: separa serviço de notificações
```

---

# 6. Histórico

Consultar:

```bash
git log
```

Forma compacta:

```bash
git log --oneline --graph --decorate --all
```

Exemplo:

```text
* a91c302 (HEAD -> main) feat: adiciona dashboard
* c842e11 fix: corrige autenticação
* 12fa908 initial commit
```

Cada commit possui um hash que o identifica.

---

# 7. Branch

Uma branch é uma linha de desenvolvimento.

O objetivo é permitir que uma alteração seja desenvolvida sem modificar diretamente a linha principal.

Exemplo:

```text
main
 |
 A---B---C
             D---E
       feature/dashboard
```

Criar:

```bash
git switch -c feature/dashboard
```

ou:

```bash
git checkout -b feature/dashboard
```

Listar:

```bash
git branch
```

Trocar:

```bash
git switch main
```

---

# 8. Por que utilizar branches

Imagine que `main` contém a versão estável.

Uma nova interface deve ser desenvolvida.

Em vez de alterar `main` diretamente:

```text
main
  |
  +---- feature/nova-interface
```

Todo o trabalho ocorre na branch.

Somente depois de implementação, testes e revisão ela será integrada.

Isso é a base do fluxo com Pull Requests.

---

# 9. Uma branch para cada mudança

Uma prática recomendada:

```text
main
 |
 +-- feature/login
 |
 +-- feature/dashboard
 |
 +-- fix/calculo-fatura
```

Evite colocar várias mudanças independentes na mesma branch.

Uma branch pequena gera:

- PR menor;
- revisão mais simples;
- testes mais direcionados;
- rollback conceitualmente mais fácil;
- menor chance de conflito.

---

# 10. Branches e desenvolvimento paralelo

É possível trabalhar em várias funcionalidades antes de integrá-las.

Exemplo:

```text
                 +-- feature-a
                /
main -----------+-- feature-b
                \
                 +-- feature-c
```

Cada branch pode gerar uma PR independente.

Posteriormente as PRs podem ser integradas uma por uma.

Não é necessário fazer merge imediatamente após abrir uma PR.

---

# 11. Branch local e remota

Uma branch inicialmente pode existir somente na máquina local.

Publicar:

```bash
git push -u origin feature/dashboard
```

Agora temos:

```text
LOCAL                       GITHUB

feature/dashboard  ------>  origin/feature/dashboard
```

O `-u` estabelece o upstream.

Depois disso:

```bash
git push
```

normalmente é suficiente.

---

# 12. Remote

Verificar:

```bash
git remote -v
```

Normalmente:

```text
origin
```

Adicionar:

```bash
git remote add origin URL_DO_REPOSITORIO
```

Enviar `main`:

```bash
git push -u origin main
```

---

# 13. Clone

Para obter um repositório existente:

```bash
git clone URL_DO_REPOSITORIO
```

O clone traz o histórico e configura normalmente `origin`.

---

# 14. Fetch

`git fetch` consulta alterações do remoto sem integrá-las automaticamente à branch atual.

```bash
git fetch origin
```

É útil para atualizar a visão do estado remoto antes de decidir o que fazer.

---

# 15. Pull

`git pull` normalmente combina busca e integração das alterações remotas.

```bash
git pull
```

Conceitualmente:

```text
git fetch
    +
integração
```

Em equipes, é importante entender a estratégia configurada para pull: merge, rebase ou fast-forward.

---

# 16. Push

Publica commits locais:

```bash
git push
```

Fluxo:

```text
LOCAL                    REMOTO

A---B---C  ------------> A---B---C
```

Push não é Pull Request.

O push envia commits.

A PR propõe integrar uma branch em outra e será estudada no Volume 02.

---

# 17. Merge

Merge integra históricos.

Exemplo:

```text
main
A---B-------M
     \     /
      C---D
      feature
```

`M` representa o merge.

Exemplo local:

```bash
git switch main
git merge feature/dashboard
```

Em um fluxo baseado em GitHub, normalmente o merge de uma feature ocorre através da Pull Request.

---

# 18. Fast-forward

Quando não existem alterações concorrentes, o Git pode simplesmente avançar a referência.

Antes:

```text
main
 A---B
      \
       C---D feature
```

Depois:

```text
main
 A---B---C---D
```

Não é necessário um commit de merge adicional.

---

# 19. Merge commit

Em outros casos, o Git cria um commit que une duas linhas.

```text
A---B-------M
     \     /
      C---D
```

Essa estratégia preserva explicitamente a topologia da branch.

---

# 20. Squash merge

No GitHub, uma PR pode ser integrada condensando vários commits em um único commit.

Branch:

```text
C1
C2
C3
C4
```

Após squash:

```text
S1
```

Vantagem: histórico principal mais compacto.

É muito útil quando os commits intermediários são apenas etapas de desenvolvimento.

---

# 21. Rebase

Rebase reposiciona commits sobre outra base.

Antes:

```text
A---B---C main
     \
      D---E feature
```

Depois do rebase:

```text
A---B---C main
         \
          D'---E' feature
```

Os commits são recriados.

Por isso os hashes mudam.

---

# 22. Quando evitar rebase

Evite reescrever histórico compartilhado sem coordenação.

Regra prática:

> Rebase é excelente para organizar trabalho local; exige cuidado quando commits já são utilizados por outras pessoas.

---

# 23. Conflitos

Um conflito acontece quando o Git não consegue decidir automaticamente como combinar mudanças.

Exemplo:

Branch A:

```javascript
const timeout = 5000;
```

Branch B:

```javascript
const timeout = 10000;
```

O Git pode apresentar:

```text
<<<<<<< HEAD
const timeout = 5000;
=======
const timeout = 10000;
>>>>>>> feature
```

O desenvolvedor deve decidir o resultado correto.

Depois:

```bash
git add arquivo
git commit
```

ou continuar a operação específica em andamento.

---

# 24. Como reduzir conflitos

- branches pequenas;
- PRs pequenas;
- atualizar branches com frequência;
- evitar refatorações gigantes junto com features;
- separar responsabilidades;
- não deixar branches abertas por períodos excessivos;
- integrar mudanças independentes cedo quando apropriado.

---

# 25. HEAD

`HEAD` representa normalmente a posição atual do desenvolvedor.

Exemplo:

```text
HEAD
 |
 v
main
 |
 C
```

Após:

```bash
git switch feature/login
```

temos:

```text
HEAD
 |
 v
feature/login
```

---

# 26. Git diff

Mudanças não adicionadas:

```bash
git diff
```

Mudanças preparadas:

```bash
git diff --staged
```

Comparar branches:

```bash
git diff main..feature/dashboard
```

Antes de criar um commit ou PR, revisar o diff é uma prática importante.

---

# 27. Restaurar arquivo

Para descartar uma alteração não preparada:

```bash
git restore arquivo
```

Para retirar da staging area:

```bash
git restore --staged arquivo
```

Esses comandos devem ser usados conscientemente para não perder trabalho.

---

# 28. Git stash

Permite guardar temporariamente alterações sem criar commit.

```bash
git stash
```

Listar:

```bash
git stash list
```

Restaurar:

```bash
git stash pop
```

Útil para uma troca rápida de contexto, mas não deve substituir commits adequados.

---

# 29. Tags

Tags identificam pontos importantes do histórico.

Exemplo:

```bash
git tag v1.0.0
```

Tag anotada:

```bash
git tag -a v1.0.0 -m "Release 1.0.0"
```

Publicar:

```bash
git push origin v1.0.0
```

Tags são úteis para releases e deploys.

---

# 30. Versionamento semântico

Formato:

```text
MAJOR.MINOR.PATCH
```

Exemplo:

```text
2.4.1
```

Conceitualmente:

- `MAJOR`: mudança incompatível;
- `MINOR`: nova funcionalidade compatível;
- `PATCH`: correção compatível.

Exemplo:

```text
v1.0.0
v1.1.0
v1.1.1
v2.0.0
```

---

# 31. .gitignore

Arquivos que não devem ser versionados:

```gitignore
node_modules/
vendor/
.env
coverage/
dist/
*.log
```

Nunca utilize `.gitignore` como única estratégia para proteger uma credencial que já foi commitada.

Se um secret entrou no histórico, deve ser considerado comprometido e rotacionado.

---

# 32. O que não deve ser commitado

Normalmente:

- senhas;
- tokens;
- chaves privadas;
- `.env` real;
- `node_modules`;
- dependências reconstruíveis;
- dumps sensíveis;
- arquivos temporários;
- logs;
- artifacts de build desnecessários.

---

# 33. Fluxo diário básico

```bash
git switch main
git pull

git switch -c feature/minha-feature

# desenvolver

git status
git diff

git add .
git diff --staged

git commit -m "feat: implementa minha feature"

git push -u origin feature/minha-feature
```

Depois, no GitHub:

```text
Abrir Pull Request
```

---

# 34. Fluxo completo conceitual

```text
main
 |
 | criar branch
 v
feature/x
 |
 | desenvolver
 v
arquivos modificados
 |
 | git add
 v
staging
 |
 | git commit
 v
commits
 |
 | git push
 v
GitHub
 |
 | abrir PR
 v
Pull Request
 |
 | testes + revisão
 v
merge
 |
 v
main
```

---

# 35. Várias PRs antes do merge

É perfeitamente possível:

```text
main
 |
 +-- branch-A --> PR #10
 |
 +-- branch-B --> PR #11
 |
 +-- branch-C --> PR #12
```

Continuar desenvolvendo e só depois integrar.

Porém, as PRs devem ser realmente independentes.

Se `B` depende de `A`, a ordem de integração e a base das branches precisam ser planejadas.

O Volume 02 detalhará esse cenário.

---

# 36. Commits atômicos

Um commit atômico representa uma unidade lógica.

Ruim:

```text
feat: login, dashboard, banco, css e correção de relatório
```

Melhor:

```text
feat: adiciona endpoint de login
test: cobre autenticação inválida
feat: adiciona sessão ao frontend
fix: corrige expiração do token
```

Isso melhora:

- revisão;
- bisect;
- rollback;
- investigação;
- rastreabilidade.

---

# 37. Git bisect

Quando uma regressão apareceu em algum ponto do histórico, `git bisect` pode localizar o commit responsável através de busca binária.

Início:

```bash
git bisect start
```

Marcar versão ruim:

```bash
git bisect bad
```

Marcar commit conhecido como bom:

```bash
git bisect good HASH
```

O Git navegará pelo histórico para reduzir a busca.

Finalizar:

```bash
git bisect reset
```

Com testes automatizados, esse recurso pode ser ainda mais poderoso.

---

# 38. Git revert

Para desfazer uma mudança já compartilhada, frequentemente é preferível criar um novo commit inverso:

```bash
git revert HASH
```

Isso preserva o histórico.

Conceito:

```text
A---B---C---R
```

`R` desfaz o efeito de `C`, sem apagar `C` da história.

---

# 39. Git reset

`git reset` altera referências e pode modificar staging/working tree dependendo do modo.

Exemplos:

```bash
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1
```

`--hard` pode destruir alterações locais.

Não utilize sem compreender exatamente o efeito.

Para histórico compartilhado, `git revert` costuma ser mais seguro.

---

# 40. Branch principal

Neste guia utilizaremos:

```text
main
```

como branch principal.

A branch principal deverá representar código integrado e sujeito às regras de qualidade definidas pelo projeto.

No GitHub, ela poderá posteriormente ser protegida contra:

- push direto;
- merge sem testes;
- merge sem revisão;
- exclusão acidental.

---

# 41. Convenção de nomes de branches

Sugestão:

```text
feature/nome
fix/nome
refactor/nome
docs/nome
test/nome
chore/nome
```

Exemplos:

```text
feature/dashboard-clientes
fix/login-timeout
refactor/mqtt-service
docs/runner-installation
test/e2e-authentication
```

O objetivo é comunicar intenção.

---

# 42. Convenção de commits

Este guia utilizará uma convenção inspirada em Conventional Commits:

```text
feat:
fix:
docs:
test:
refactor:
chore:
ci:
build:
perf:
```

Exemplos:

```text
ci: adiciona workflow de testes
test: adiciona E2E de login
fix: corrige reconexão MQTT
docs: documenta self-hosted runner
```

---

# 43. Git no contexto de CI/CD

O Git não serve apenas para armazenar código.

O histórico se torna um gatilho operacional.

Exemplo:

```text
git push
   |
   v
GitHub
   |
   v
GitHub Actions
   |
   v
Runner
   |
   +-- lint
   +-- tests
   +-- build
```

Uma tag pode iniciar uma release.

Uma PR pode iniciar testes.

Um merge pode iniciar deploy DEV.

Uma aprovação pode liberar produção.

---

# 44. Git e desenvolvimento assistido por IA

O uso de IA aumenta a importância de branches e commits pequenos.

Fluxo recomendado:

```text
SPEC
 |
 v
Plano
 |
 v
Branch
 |
 v
IA implementa uma etapa
 |
 v
Teste
 |
 v
Commit
 |
 v
Próxima etapa
```

Isso cria pontos de controle.

Se a implementação da IA seguir uma direção incorreta, fica mais fácil:

- identificar;
- comparar;
- reverter;
- corrigir;
- revisar.

---

# 45. Branch por SPEC

Uma estratégia útil:

```text
SPEC-042
   |
   v
feature/042-novo-dashboard
   |
   v
PR #57
```

A PR referencia a SPEC.

Uma alteração posterior:

```text
SPEC-063
Refina PR #57
   |
   v
feature/063-ajuste-dashboard
   |
   v
PR #81
```

Isso cria rastreabilidade histórica.

---

# 46. Checklist antes do commit

- [ ] O código compila/executa?
- [ ] O diff foi revisado?
- [ ] Não existem secrets?
- [ ] Não existem arquivos temporários?
- [ ] A alteração pertence ao escopo da branch?
- [ ] Os testes pertinentes passam?
- [ ] A mensagem do commit descreve a intenção?

---

# 47. Checklist antes do push

- [ ] Branch correta?
- [ ] Commits corretos?
- [ ] Nenhum secret?
- [ ] Nenhum arquivo grande acidental?
- [ ] Testes locais básicos executados?
- [ ] Histórico compreensível?

---

# 48. Checklist antes da Pull Request

- [ ] A branch possui um objetivo claro?
- [ ] A mudança está completa o suficiente para revisão?
- [ ] A SPEC foi atendida?
- [ ] Critérios de aceitação foram verificados?
- [ ] Testes foram criados/atualizados?
- [ ] Documentação necessária foi atualizada?
- [ ] A branch está atualizada em relação à base?

---

# 49. Comandos essenciais

```bash
git status
git add
git commit
git log
git diff
git branch
git switch
git fetch
git pull
git push
git merge
git rebase
git restore
git stash
git tag
git revert
```

Não é necessário memorizar todas as opções.

É mais importante compreender o modelo de dados e saber qual operação deseja realizar.

---

# 50. Resumo mental

```text
REPOSITÓRIO
   |
   +-- commits
   |
   +-- branches
   |
   +-- tags
```

Durante o desenvolvimento:

```text
Working Tree
     |
     | git add
     v
Staging Area
     |
     | git commit
     v
Commit
```

Colaboração:

```text
Branch
   |
   | push
   v
GitHub
   |
   v
Pull Request
```

Automação:

```text
Pull Request
   |
   v
GitHub Actions
   |
   v
Runner
   |
   v
Testes
```

---

# 51. Laboratório proposto

Criar um repositório de treinamento e executar:

1. criar repositório;
2. criar `README.md`;
3. fazer primeiro commit;
4. criar branch `feature/exemplo`;
5. alterar README;
6. fazer dois commits;
7. publicar branch;
8. observar o histórico;
9. comparar `main` e a feature;
10. abrir posteriormente uma PR durante o Volume 02.

---

# 52. Próximo volume

**Volume 02 — GitHub e Pull Requests**

O próximo volume aprofundará:

- GitHub;
- repositórios remotos;
- Pull Requests;
- PRs simultâneas;
- PRs dependentes;
- revisão;
- merge;
- squash;
- branch protection;
- Issues;
- templates;
- CODEOWNERS;
- rastreabilidade entre SPEC, branch, PR e release.

---

**Fim do Volume 01 — Fundamentos de Git e Controle de Versão**
