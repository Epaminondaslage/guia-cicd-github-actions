# Volume 04 — Self-Hosted Runners com Ubuntu e Docker

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 04_SELF_HOSTED_RUNNERS.md  
**Versão:** 0.1.0  
**Status:** Primeira versão para expansão incremental  
**Ambiente de referência:** Ubuntu Server + Docker + GitHub Actions  
**Objetivo:** Substituir total ou parcialmente runners hospedados pelo GitHub por infraestrutura própria, reduzindo consumo de minutos e aumentando o controle sobre o ambiente de testes e deploy.

---

## 1. Objetivo deste capítulo

Este capítulo descreve como implantar um **GitHub Actions Self-Hosted Runner** em um servidor Linux próprio.

O servidor será preparado para executar:

- pipelines de CI;
- testes Node.js;
- testes PHP;
- testes de integração;
- testes E2E;
- Playwright;
- Docker e Docker Compose;
- builds;
- criação de artifacts;
- deploy em ambiente DEV;
- preparação de deploy para produção com aprovação humana.

O modelo adotado é:

```text
GitHub
  |
  | HTTPS / porta 443
  |
  v
Servidor Linux
Ubuntu Server
  |
  +-- GitHub Actions Runner
  |
  +-- Docker Engine
  |
  +-- Docker Compose
  |
  +-- Node.js
  |
  +-- PHP / Composer
  |
  +-- Playwright
  |
  +-- ferramentas auxiliares
```

---

# 2. O que é um Self-Hosted Runner

Um GitHub Actions Runner é o agente responsável por executar os `jobs` definidos nos workflows.

Em um workflow tradicional:

```yaml
runs-on: ubuntu-latest
```

o GitHub cria uma máquina temporária na infraestrutura dele.

Com um runner próprio:

```yaml
runs-on: self-hosted
```

a execução ocorre em uma máquina administrada por você.

Também podemos utilizar labels:

```yaml
runs-on:
  - self-hosted
  - linux
  - x64
  - docker
  - e2e
```

Isso permite escolher qual máquina executará determinado job.

---

# 3. Por que utilizar runner próprio

Principais vantagens:

- redução do consumo de minutos hospedados pelo GitHub;
- maior controle sobre CPU e RAM;
- armazenamento local de caches;
- possibilidade de máquinas mais potentes para E2E;
- acesso controlado à rede interna;
- integração direta com servidores DEV;
- possibilidade de Docker local;
- ferramentas persistentes entre execuções;
- controle sobre versões de Node.js, PHP, browsers e dependências;
- possibilidade de runners especializados.

Exemplo:

```text
runner-ci
   |
   +-- lint
   +-- unit
   +-- integration

runner-e2e
   |
   +-- Chromium
   +-- Firefox
   +-- Playwright

runner-deploy
   |
   +-- acesso ao DEV
   +-- acesso controlado ao PROD
```

---

# 4. Limitações e responsabilidades

O runner próprio transfere parte da responsabilidade operacional do GitHub para você.

Você passa a ser responsável por:

- sistema operacional;
- atualizações;
- segurança;
- disponibilidade;
- armazenamento;
- limpeza de arquivos temporários;
- Docker;
- monitoramento;
- backups de configuração;
- isolamento;
- capacidade da máquina;
- proteção contra workflows maliciosos.

O runner deve ser tratado como parte da infraestrutura de produção do processo de desenvolvimento.

---

# 5. Regra de segurança fundamental

Self-hosted runners executam código enviado pelo workflow.

Portanto:

> Um runner nunca deve ser considerado seguro apenas porque está dentro da sua rede.

O código executado pode:

- ler arquivos acessíveis ao usuário do runner;
- executar comandos;
- utilizar Docker;
- acessar recursos permitidos pela rede;
- tentar capturar credenciais expostas ao job.

Para repositórios sob controle do próprio desenvolvedor, o risco pode ser administrado.

Para repositórios públicos que aceitam PRs de terceiros, runners persistentes exigem cuidados adicionais e normalmente não devem executar automaticamente código não confiável.

---

# 6. Arquitetura recomendada para o laboratório

Para começar, será utilizada uma única máquina:

```text
                    INTERNET
                       |
                       v
                  github.com
                       |
                 HTTPS :443
                       |
                       v
        +-----------------------------+
        | Ubuntu Server               |
        |                             |
        | github-runner               |
        |                             |
        | Docker Engine               |
        | Docker Compose              |
        |                             |
        | Node.js                     |
        | PHP / Composer              |
        | Playwright                  |
        +-----------------------------+
                       |
                       |
              Rede de desenvolvimento
                       |
                 +-----+------+
                 |            |
                 v            v
               DEV         Serviços
                            auxiliares
```

No início, o runner poderá ser utilizado para CI e E2E.

Posteriormente, o guia separará:

```text
runner-ci
runner-e2e
runner-deploy
```

---

# 7. Hardware recomendado

O aplicativo do runner em si é leve.

Quem define o consumo real é o pipeline.

## 7.1 Laboratório mínimo

```text
CPU:   2 vCPU
RAM:   4 GB
SSD:   40 GB
SO:    Ubuntu Server
Rede:  conexão estável
```

Adequado para:

- lint;
- testes unitários;
- Node.js pequeno;
- PHP;
- builds simples.

## 7.2 Ambiente recomendado

```text
CPU:   4 a 8 cores
RAM:   8 a 16 GB
SSD:   100 GB ou mais
```

Adequado para:

- Docker;
- banco temporário;
- Playwright;
- múltiplos containers;
- testes E2E.

## 7.3 E2E mais pesado

```text
CPU:   8+ cores
RAM:   16+ GB
SSD:   NVMe
```

Especialmente útil quando forem executados browsers em paralelo.

---

# 8. Sistema operacional recomendado

Neste guia será utilizado:

```text
Ubuntu Server LTS
```

O uso de uma versão LTS simplifica:

- manutenção;
- documentação;
- disponibilidade de pacotes;
- suporte do Docker;
- suporte do GitHub Actions Runner.

---

# 9. Preparação inicial do Ubuntu

Atualize o sistema:

```bash
sudo apt update
sudo apt upgrade -y
```

Instale ferramentas básicas:

```bash
sudo apt install -y \
    curl \
    wget \
    git \
    ca-certificates \
    gnupg \
    unzip \
    jq \
    rsync \
    htop
```

Confirme:

```bash
git --version
curl --version
```

---

# 10. Criar usuário dedicado ao runner

Evite executar o runner como `root`.

Crie:

```bash
sudo adduser github-runner
```

Opcionalmente:

```bash
sudo usermod -aG sudo github-runner
```

Para um ambiente mais restritivo, não adicione o usuário ao grupo `sudo` e libere somente comandos específicos quando necessário.

Alterne para o usuário:

```bash
sudo su - github-runner
```

---

# 11. Estrutura de diretórios

Sugestão:

```text
/home/github-runner/
|
+-- actions-runner/
|
+-- cache/
|
+-- scripts/
|
+-- logs/
```

Criar:

```bash
mkdir -p ~/actions-runner
mkdir -p ~/cache
mkdir -p ~/scripts
mkdir -p ~/logs
```

---

# 12. Instalar Docker Engine

Para pipelines com containers e service containers, o Docker será uma peça importante.

A instalação recomendada deve utilizar o repositório oficial do Docker.

## 12.1 Dependências

```bash
sudo apt update
sudo apt install -y ca-certificates curl
```

## 12.2 Diretório de chaves

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

## 12.3 Chave do Docker

```bash
sudo curl -fsSL \
  https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc
```

## 12.4 Adicionar repositório

```bash
sudo tee /etc/apt/sources.list.d/docker.sources > /dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Atualize:

```bash
sudo apt update
```

## 12.5 Instalar componentes

```bash
sudo apt install -y \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    docker-buildx-plugin \
    docker-compose-plugin
```

Verifique:

```bash
sudo systemctl status docker
```

Teste:

```bash
sudo docker run --rm hello-world
```

---

# 13. Permitir uso do Docker pelo runner

Adicione:

```bash
sudo usermod -aG docker github-runner
```

Depois faça logout/login da sessão.

Teste como `github-runner`:

```bash
docker version
docker compose version
```

E:

```bash
docker run --rm hello-world
```

## Atenção

Ser membro do grupo `docker` equivale, na prática, a possuir privilégios elevados no host.

Portanto:

- somente usuários confiáveis devem estar no grupo;
- não execute PRs não confiáveis;
- considere runners descartáveis futuramente.

---

# 14. Instalar Node.js

Existem duas estratégias.

## Estratégia A — Actions configuram Node por job

Preferencial para CI:

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 22
```

Vantagens:

- workflow documenta a versão;
- fácil alternar versões;
- reduz dependência da instalação global.

## Estratégia B — Node global no runner

Útil para ferramentas administrativas.

Verifique:

```bash
node --version
npm --version
```

Para os pipelines do guia, a versão deverá sempre estar declarada no workflow.

---

# 15. PHP e Composer

Para projetos PHP:

```bash
sudo apt install -y \
    php-cli \
    php-curl \
    php-mbstring \
    php-xml \
    php-zip
```

Confirme:

```bash
php -v
```

O Composer poderá ser instalado de forma controlada no host ou preparado pelo próprio workflow.

Em pipelines profissionais, versões devem ser explicitadas.

---

# 16. Playwright

Para testes E2E Node.js, este guia utilizará preferencialmente Playwright.

Normalmente a instalação deve fazer parte do projeto:

```bash
npm install
```

e depois:

```bash
npx playwright install --with-deps
```

Em runner persistente, browsers podem ocupar bastante disco.

Monitorar:

```bash
df -h
```

e:

```bash
du -sh ~/.cache/*
```

---

# 17. Registrar o runner no GitHub

O GitHub gera comandos específicos e um token temporário.

Não copie comandos de terceiros contendo tokens.

No repositório desejado:

```text
Repository
   |
Settings
   |
Actions
   |
Runners
   |
New self-hosted runner
```

Escolha:

```text
Linux
x64
```

O GitHub exibirá algo semelhante a:

```bash
mkdir actions-runner
cd actions-runner
```

Depois exibirá o download correspondente à versão atual do runner.

Execute **os comandos fornecidos naquele momento pelo próprio GitHub**.

---

# 18. Por que não fixar aqui a URL do pacote

O GitHub Actions Runner recebe novas versões regularmente.

Por isso, o procedimento correto é utilizar sempre os comandos apresentados em:

```text
Settings
→ Actions
→ Runners
→ New self-hosted runner
```

Isso evita instalar uma versão antiga apenas porque um tutorial ficou desatualizado.

---

# 19. Configurar o runner

Após descompactar o pacote:

```bash
cd ~/actions-runner
```

O GitHub fornecerá um comando no formato:

```bash
./config.sh --url REPOSITORIO --token TOKEN_TEMPORARIO
```

Execute o comando exato fornecido pela interface.

O assistente perguntará:

```text
Runner group
Runner name
Work folder
Labels
```

Sugestão:

```text
Runner name:
ci-runner-01

Work folder:
_work
```

---

# 20. Labels recomendadas

Por padrão, o GitHub fornece labels como:

```text
self-hosted
linux
x64
```

Podemos acrescentar:

```text
docker
node
php
e2e
dev
```

Exemplo final:

```yaml
runs-on:
  - self-hosted
  - linux
  - x64
  - docker
```

Um runner dedicado para E2E:

```yaml
runs-on:
  - self-hosted
  - linux
  - e2e
```

---

# 21. Testar manualmente

Antes de instalar como serviço:

```bash
cd ~/actions-runner
./run.sh
```

O terminal deverá indicar que está conectado e aguardando jobs.

No GitHub, em:

```text
Settings
→ Actions
→ Runners
```

o estado deverá aparecer como:

```text
Idle
```

Quando estiver executando:

```text
Active
```

---

# 22. Instalar como serviço

No Linux, o runner pode ser executado pelo `systemd`.

Dentro de `actions-runner`:

```bash
sudo ./svc.sh install
```

Depois:

```bash
sudo ./svc.sh start
```

Consultar:

```bash
sudo ./svc.sh status
```

---

# 23. Comandos de operação

## Iniciar

```bash
sudo ./svc.sh start
```

## Parar

```bash
sudo ./svc.sh stop
```

## Status

```bash
sudo ./svc.sh status
```

---

# 24. Confirmar no GitHub

Acesse:

```text
Repository
→ Settings
→ Actions
→ Runners
```

Confirme:

```text
ci-runner-01
Idle
```

Se estiver offline, verificar:

```bash
sudo ./svc.sh status
```

e conectividade:

```bash
curl -I https://github.com
```

---

# 25. Primeiro workflow

Crie:

```text
.github/workflows/test-self-hosted.yml
```

Conteúdo:

```yaml
name: Test Self Hosted Runner

on:
  workflow_dispatch:

jobs:
  test-runner:
    runs-on:
      - self-hosted
      - linux
      - x64

    steps:
      - name: Mostrar host
        run: hostname

      - name: Mostrar usuário
        run: whoami

      - name: Informações do sistema
        run: uname -a

      - name: Verificar Docker
        run: docker version

      - name: Verificar Docker Compose
        run: docker compose version

      - name: Espaço em disco
        run: df -h
```

Faça commit.

No GitHub:

```text
Actions
→ Test Self Hosted Runner
→ Run workflow
```

---

# 26. Workflow Node.js

Exemplo:

```yaml
name: CI Node

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  test:
    runs-on:
      - self-hosted
      - linux
      - x64
      - docker

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Node
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - name: Instalar dependências
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Testes
        run: npm test
```

---

# 27. Workflow E2E com Playwright

Exemplo:

```yaml
name: E2E

on:
  pull_request:

jobs:
  e2e:
    runs-on:
      - self-hosted
      - linux
      - e2e

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - run: npm ci

      - name: Instalar browsers
        run: npx playwright install --with-deps

      - name: Executar E2E
        run: npx playwright test
```

Posteriormente este workflow será otimizado para evitar reinstalar browsers desnecessariamente.

---

# 28. Banco de dados temporário com Docker

Para testes de integração:

```yaml
services:
  mysql:
    image: mysql:8
    env:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: app_test
    ports:
      - 3306:3306
```

Entretanto, em runners persistentes é importante evitar colisão de portas quando houver múltiplos jobs simultâneos.

Uma alternativa mais controlada será utilizar Docker Compose com nomes de projeto únicos.

---

# 29. Docker Compose para teste

Arquivo:

```text
docker-compose.test.yml
```

Exemplo:

```yaml
services:

  db:
    image: mariadb:11
    environment:
      MARIADB_ROOT_PASSWORD: test
      MARIADB_DATABASE: app_test

  redis:
    image: redis:alpine
```

Workflow:

```yaml
- name: Subir serviços
  run: docker compose -f docker-compose.test.yml up -d

- name: Executar testes
  run: npm test

- name: Encerrar serviços
  if: always()
  run: docker compose -f docker-compose.test.yml down -v
```

O `if: always()` é importante para tentar limpar os containers mesmo quando os testes falham.

---

# 30. Não deixar lixo no runner

Diferentemente de runners efêmeros do GitHub, o self-hosted persiste.

Consequentemente:

```text
execução 1
   |
   +-- arquivos
   +-- caches
   +-- containers
   +-- volumes
   |
execução 2
   |
   +-- pode encontrar resíduos
```

Por isso, o workflow precisa possuir etapas explícitas de limpeza.

---

# 31. Limpeza Docker

Inspeção:

```bash
docker ps -a
docker images
docker volume ls
```

Não utilizar indiscriminadamente comandos destrutivos em hosts que executem outros serviços.

Se o servidor for **exclusivo do runner**, uma política periódica pode ser implementada.

Exemplo conservador:

```bash
docker container prune -f
docker image prune -f
```

Evite:

```bash
docker system prune -a --volumes
```

sem entender exatamente o impacto.

---

# 32. Diretório de trabalho

O GitHub Runner utiliza normalmente:

```text
actions-runner/_work/
```

Exemplo:

```text
_work/
|
+-- projeto/
    |
    +-- projeto/
```

O conteúdo pode persistir entre jobs.

Portanto:

- não confie em resíduos da execução anterior;
- use `checkout`;
- evite armazenar secrets em arquivos permanentes;
- faça limpeza adequada.

---

# 33. Runner e secrets

Nunca grave secrets diretamente em:

```text
.env
workflow YAML
scripts versionados
README
```

Utilize:

```text
GitHub Secrets
```

No workflow:

```yaml
env:
  DATABASE_PASSWORD: ${{ secrets.DATABASE_PASSWORD }}
```

Melhor ainda: passe o secret somente para o step que realmente necessita dele.

---

# 34. Separar CI de deploy

Não é recomendado que todo job tenha acesso ao servidor de produção.

Arquitetura futura:

```text
runner-ci
   |
   +-- sem acesso ao PROD

runner-e2e
   |
   +-- sem acesso ao PROD

runner-deploy
   |
   +-- acesso restrito ao PROD
```

Isso reduz o impacto de uma possível falha de segurança.

---

# 35. Modelo DEV → PROD

Fluxo:

```text
PR
 |
 v
CI
 |
 v
E2E
 |
 v
Merge
 |
 v
Deploy DEV
 |
 v
Validação
 |
 v
Aprovação humana
 |
 v
Deploy PROD
```

O deploy de produção deverá ser protegido por:

```text
GitHub Environment: production
```

e aprovação humana.

---

# 36. Exemplo de workflow com gate

Estrutura conceitual:

```yaml
jobs:

  test:
    runs-on: self-hosted

  deploy-dev:
    needs: test
    runs-on: self-hosted

  deploy-prod:
    needs: deploy-dev
    environment:
      name: production
    runs-on: self-hosted
```

O Environment `production` poderá exigir aprovação antes do job iniciar.

---

# 37. Estratégia para diminuir tempo de E2E

Não execute necessariamente toda a suíte E2E a cada pequeno commit.

Separação recomendada:

```text
PR
 |
 +-- lint
 +-- unit
 +-- integração
 +-- smoke E2E

DEV
 |
 +-- E2E relacionados

Antes de PROD
 |
 +-- regressão relevante

Noturno
 |
 +-- suíte E2E completa
```

---

# 38. Smoke E2E

Exemplo:

```text
login
dashboard
criar registro
consultar registro
logout
```

Esses fluxos devem ser rápidos.

O objetivo não é validar cada detalhe da aplicação, mas responder:

> A aplicação essencial continua funcionando?

---

# 39. Paralelização

Uma máquina com múltiplos cores poderá executar testes paralelamente.

Entretanto, um único runner normalmente processa um job por vez.

Para execução simultânea de jobs, podem ser instalados múltiplos runners.

Exemplo:

```text
Servidor físico
|
+-- runner-01
+-- runner-02
+-- runner-03
```

Cada runner terá diretório próprio.

---

# 40. Runner dedicado para E2E

Uma evolução interessante:

```text
runner-ci-01
labels:
self-hosted, linux, ci

runner-e2e-01
labels:
self-hosted, linux, e2e
```

Workflow:

```yaml
unit:
  runs-on:
    - self-hosted
    - ci

e2e:
  runs-on:
    - self-hosted
    - e2e
```

---

# 41. Monitoramento básico

Verifique periodicamente:

```bash
uptime
free -h
df -h
docker ps
```

Processos:

```bash
ps aux | grep Runner
```

Serviço:

```bash
sudo ./svc.sh status
```

---

# 42. Logs

Logs do runner:

```text
actions-runner/_diag/
```

Listar:

```bash
ls -lh ~/actions-runner/_diag/
```

Em caso de falha, esta é uma das primeiras fontes para diagnóstico.

---

# 43. Espaço em disco

E2E e Docker consomem disco rapidamente.

Verifique:

```bash
df -h
```

Docker:

```bash
docker system df
```

Diretórios grandes:

```bash
du -h --max-depth=1 ~/ | sort -h
```

---

# 44. Atualização do runner

O aplicativo oficial possui mecanismo de atualização automática.

Ainda assim, deve existir monitoramento para evitar que o runner permaneça offline após uma atualização ou falha.

Rotina:

```text
1. verificar GitHub
2. verificar status
3. verificar _diag
4. verificar espaço em disco
5. verificar Docker
```

---

# 45. Backup

Não é necessário fazer backup do diretório inteiro `_work`.

O runner deve ser reconstruível.

Backup recomendado:

```text
scripts administrativos
documentação
configurações
arquivos systemd adicionais
inventário de labels
Docker Compose administrativo
procedimentos
```

Não armazenar em backup:

```text
tokens temporários
credentials em texto puro
workspace de builds
```

---

# 46. Estratégia de reconstrução

Uma boa infraestrutura deve permitir:

```text
servidor perdido
      |
      v
instalar Ubuntu
      |
      v
instalar Docker
      |
      v
criar usuário
      |
      v
instalar runner
      |
      v
registrar
      |
      v
operacional
```

O objetivo futuro é automatizar grande parte desse procedimento.

---

# 47. Endurecimento básico do servidor

Recomendações:

- SSH por chave;
- desabilitar login root remoto;
- firewall;
- atualizações regulares;
- runner com usuário dedicado;
- sem serviços desnecessários;
- sem banco de produção no mesmo host;
- sem aplicações de produção no mesmo Docker daemon;
- logs monitorados;
- backups das configurações;
- acesso de rede mínimo necessário.

---

# 48. Firewall

O runner necessita comunicação de saída com o GitHub.

Normalmente não é necessário abrir uma porta de entrada na internet apenas para o runner.

Fluxo:

```text
Runner
  |
  | conexão de saída
  v
GitHub
```

Isso é uma vantagem de segurança importante.

---

# 49. Não misturar produção e runner

Evite:

```text
MESMO HOST
|
+-- aplicação PROD
+-- banco PROD
+-- GitHub runner
```

Preferir:

```text
HOST 1
CI Runner

HOST 2
DEV

HOST 3
PROD
```

Mesmo que inicialmente sejam VMs no mesmo servidor físico.

---

# 50. Máquina física, VM ou container

## VM

É a opção recomendada inicialmente.

Vantagens:

- isolamento;
- snapshots;
- fácil reconstrução;
- limites de CPU/RAM;
- manutenção simples.

## Máquina física

Boa para E2E pesado.

## Container

É possível executar runners em containers, mas adiciona complexidade, principalmente quando os jobs também precisam executar Docker.

Será tratado em capítulo avançado.

---

# 51. Produtos e componentes preferenciais

A stack deste capítulo utiliza:

| Componente | Função | Observação |
|---|---|---|
| Ubuntu Server | Sistema operacional | Preferência por versão LTS |
| Git | Versionamento | Código aberto |
| GitHub Actions Runner | Executor | Código-fonte disponível |
| Docker Engine | Containers | Componentes open source |
| Docker Compose | Orquestração local | Plugin do Docker |
| Node.js | Runtime | Código aberto |
| PHP | Runtime | Código aberto |
| Composer | Dependências PHP | Código aberto |
| Playwright | Testes E2E | Código aberto |
| MariaDB | Banco de testes | Código aberto |
| Eclipse Mosquitto | MQTT de teste | Código aberto |
| Nginx | Proxy | Código aberto |
| Prometheus | Métricas | Código aberto |
| Grafana | Dashboards | Código aberto |

O GitHub, como serviço SaaS, não é open source, mas a infraestrutura local será construída majoritariamente com ferramentas abertas.

---

# 52. Estratégia recomendada para começar

Não migrar tudo de uma vez.

## Etapa 1

Runner executa:

```text
lint
unit
integration
```

## Etapa 2

Adicionar:

```text
smoke E2E
```

## Etapa 3

Adicionar:

```text
deploy DEV
```

## Etapa 4

Adicionar:

```text
gate de produção
```

## Etapa 5

Criar runner dedicado:

```text
runner-e2e
```

---

# 53. Migração de workflow existente

Se hoje existe:

```yaml
runs-on: ubuntu-latest
```

a primeira mudança pode ser:

```yaml
runs-on:
  - self-hosted
  - linux
  - x64
```

Mas isso não garante compatibilidade.

É necessário verificar:

- ferramentas instaladas;
- dependências do sistema;
- permissões;
- Docker;
- browsers;
- variáveis;
- limpeza;
- secrets;
- paths absolutos;
- concorrência.

---

# 54. Diferença crítica: runner efêmero x persistente

GitHub-hosted:

```text
job
 |
 v
VM nova
 |
 v
execução
 |
 v
VM destruída
```

Self-hosted tradicional:

```text
job 1
 |
 v
mesma máquina
 |
 v
job 2
 |
 v
mesma máquina
 |
 v
job 3
```

Esse é um dos conceitos mais importantes deste capítulo.

---

# 55. Implicações da persistência

Cuidados:

- caches podem ajudar;
- lixo pode acumular;
- containers podem permanecer;
- alterações globais podem afetar jobs futuros;
- secrets nunca devem ser deixados em disco;
- testes devem ser reprodutíveis.

---

# 56. Teste de saúde do runner

Crie:

```bash
~/scripts/runner-health.sh
```

Exemplo:

```bash
#!/usr/bin/env bash

set -e

echo "=== HOST ==="
hostname

echo
echo "=== UPTIME ==="
uptime

echo
echo "=== MEMORY ==="
free -h

echo
echo "=== DISK ==="
df -h /

echo
echo "=== DOCKER ==="
docker version --format '{{.Server.Version}}'

echo
echo "=== CONTAINERS ==="
docker ps --format 'table {{.Names}}\t{{.Status}}'
```

Permissão:

```bash
chmod +x ~/scripts/runner-health.sh
```

---

# 57. Cron de saúde

Opcionalmente:

```bash
crontab -e
```

Exemplo:

```cron
*/15 * * * * /home/github-runner/scripts/runner-health.sh >> /home/github-runner/logs/health.log 2>&1
```

Posteriormente este controle poderá migrar para Prometheus.

---

# 58. Troubleshooting

## Runner aparece Offline

Verificar:

```bash
sudo ./svc.sh status
```

Depois:

```bash
ls -lt _diag | head
```

## Docker Permission denied

Verificar:

```bash
groups
```

Deve incluir:

```text
docker
```

Se não:

```bash
sudo usermod -aG docker github-runner
```

Depois fazer nova sessão.

## Disco cheio

```bash
df -h
docker system df
```

## Workflow fica aguardando

Possíveis causas:

- runner offline;
- labels incompatíveis;
- runner ocupado;
- workflow solicitando label inexistente.

---

# 59. Exemplo de diagnóstico de labels

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - e2e
```

Runner:

```text
self-hosted
linux
x64
docker
```

Resultado:

```text
job aguardará indefinidamente
```

porque não existe a label:

```text
e2e
```

---

# 60. Checklist de instalação

- [ ] Ubuntu instalado.
- [ ] Sistema atualizado.
- [ ] Usuário `github-runner` criado.
- [ ] Git instalado.
- [ ] Docker Engine instalado.
- [ ] Docker Compose funcionando.
- [ ] Usuário do runner com acesso ao Docker.
- [ ] Runner baixado pelo comando atual fornecido pelo GitHub.
- [ ] Runner registrado.
- [ ] Labels definidas.
- [ ] Runner executando manualmente.
- [ ] Serviço systemd instalado.
- [ ] Runner aparece como `Idle`.
- [ ] Workflow de teste executado.
- [ ] Docker testado dentro do workflow.
- [ ] Política de secrets definida.
- [ ] Política de limpeza definida.
- [ ] Monitoramento básico configurado.

---

# 61. Checklist de segurança

- [ ] Runner não executa como root.
- [ ] Acesso SSH usa chave.
- [ ] Root remoto desativado.
- [ ] Firewall configurado.
- [ ] Runner não compartilha host com banco de produção.
- [ ] Runner não compartilha host com aplicação PROD.
- [ ] Secrets somente no GitHub Secrets/Environments.
- [ ] Produção possui gate humano.
- [ ] PRs não confiáveis não executam livremente no runner.
- [ ] Docker é utilizado somente por usuários autorizados.
- [ ] Logs são revisáveis.
- [ ] Processo de reconstrução está documentado.

---

# 62. Arquitetura alvo para evolução

Primeira fase:

```text
GitHub
  |
  v
runner-01
  |
  +-- CI
  +-- E2E
  +-- DEV
```

Segunda fase:

```text
                 GitHub
                   |
       +-----------+-----------+
       |                       |
       v                       v
   runner-ci               runner-e2e
       |                       |
       v                       v
 unit/integration          browsers
       |
       +-----------+
                   |
                   v
                  DEV
                   |
                   v
              aprovação
                   |
                   v
             runner-deploy
                   |
                   v
                  PROD
```

---

# 63. Objetivo final

Ao concluir este volume, a infraestrutura deverá permitir:

```text
git push
   |
   v
GitHub
   |
   v
Self-hosted runner
   |
   +-- instalar dependências
   +-- lint
   +-- testes unitários
   +-- integração
   +-- build
   +-- smoke E2E
   |
   v
DEV
   |
   v
validação
   |
   v
aprovação humana
   |
   v
PROD
```

Sem depender dos minutos de execução dos runners hospedados pelo GitHub para os jobs migrados.

---

# 64. Próximo capítulo sugerido

O próximo documento deverá ser:

```text
03_GITHUB_ACTIONS.md
```

Conteúdo:

- anatomia completa de um workflow;
- triggers;
- jobs;
- steps;
- dependencies;
- conditions;
- cache;
- artifacts;
- secrets;
- environments;
- concurrency;
- matrices;
- reusable workflows;
- workflows específicos para self-hosted runners.

Depois:

```text
06_ESTRATEGIA_DE_TESTES.md
```

com foco em reduzir o tempo de E2E à medida que o sistema cresce.

---

# 65. Notas de atualização

Este documento evita fixar a versão do binário do GitHub Actions Runner. A instalação deve utilizar a versão apresentada no momento pela interface oficial do GitHub.

A documentação oficial atual do GitHub informa que runners Linux suportam Ubuntu 20.04 ou posterior e que workflows com container actions ou service containers precisam de Linux com Docker instalado.

O acesso do runner ao GitHub é iniciado pela própria máquina, usando comunicação HTTPS de saída.

---

**Fim do Volume 04 — Self-Hosted Runners com Ubuntu e Docker**
