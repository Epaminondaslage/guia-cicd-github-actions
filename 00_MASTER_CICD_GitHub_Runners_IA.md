# Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA

**Documento:** 00_MASTER.md  
**Versão:** 0.1.0  
**Status:** Documento mestre / índice global  
**Plataforma principal:** Linux Ubuntu, Docker, GitHub, Node.js e PHP  
**Abordagem:** CI/CD, testes automatizados, self-hosted runners e desenvolvimento assistido por IA

---

## 1. Objetivo do projeto

Este guia será construído incrementalmente como uma referência prática para implantação de um pipeline profissional de desenvolvimento, testes e entrega contínua.

A arquitetura terá como foco:

- Git e GitHub como base de versionamento e colaboração.
- Pull Requests como unidade de revisão e integração.
- GitHub Actions como mecanismo de automação.
- Self-hosted runners Linux para reduzir dependência de minutos de execução hospedados pelo GitHub.
- Docker para padronização dos ambientes.
- Node.js e PHP como stacks principais dos exemplos.
- Testes unitários, integração, API e E2E.
- Ambiente de desenvolvimento antes da produção.
- Aprovação humana como gate para produção.
- Desenvolvimento orientado por especificação.
- Uso de agentes de IA para especificação, planejamento, implementação, revisão e testes.
- Segurança, observabilidade, rollback, backup e auditoria.

O documento 00_MASTER funciona como mapa de todo o projeto. Cada capítulo poderá posteriormente ser desenvolvido em seu próprio arquivo Markdown.

---

# 2. Arquitetura conceitual do fluxo

Fluxo de referência:

```text
IDEIA / NECESSIDADE
        |
        v
DISCUSSÃO E LEVANTAMENTO
        |
        v
SPEC
        |
        v
REVISÃO DA SPEC
        |
        v
PLANO DE IMPLEMENTAÇÃO
        |
        v
BRANCH
        |
        v
IMPLEMENTAÇÃO
        |
        v
TESTES LOCAIS
        |
        v
PULL REQUEST
        |
        v
GITHUB ACTIONS
        |
        +--> Lint
        +--> Unit Tests
        +--> Integration Tests
        +--> Build
        +--> Smoke Tests
        +--> E2E selecionados
        |
        v
SELF-HOSTED RUNNER
        |
        v
DEPLOY DEV
        |
        v
VALIDAÇÃO
        |
        v
GATE HUMANO
   "Publicar?"
     /     \
   NÃO     SIM
            |
            v
       PRODUÇÃO
            |
            v
       MONITORAMENTO
```

Um princípio central será separar **CI** de **CD**.

CI valida continuamente o código.

CD controla como uma versão validada chega aos ambientes de execução.

---

# 3. Estrutura global

## VOLUME 00 — Documento Mestre

### Capítulo 00.1 — Visão geral

Apresenta os objetivos, arquitetura, público-alvo e organização do projeto.

### Capítulo 00.2 — Filosofia do pipeline

Define os princípios utilizados durante todo o guia:

- automação;
- rastreabilidade;
- reprodutibilidade;
- segurança;
- simplicidade operacional;
- rollback;
- infraestrutura versionada;
- aprovação humana para operações críticas.

### Capítulo 00.3 — Convenções

Padronização de:

- nomes de branches;
- commits;
- PRs;
- workflows;
- containers;
- ambientes;
- secrets;
- releases;
- documentação.

### Capítulo 00.4 — Glossário

Glossário evolutivo contendo Git, CI/CD, DevOps, runners, Actions, artifacts, E2E, smoke test, rollback, deployment, entre outros.

---

# VOLUME 01 — Fundamentos de Git e Controle de Versão

## Capítulo 01.1 — Git

Conceitos fundamentais do Git e funcionamento de um repositório distribuído.

## Capítulo 01.2 — Repositório

Working tree, staging area e histórico.

## Capítulo 01.3 — Commit

Como estruturar commits pequenos, rastreáveis e semanticamente claros.

## Capítulo 01.4 — Branches

Uso de branches para desenvolvimento isolado.

Exemplo:

```text
main
 |
 +--- feature/login
 |
 +--- fix/dashboard
 |
 +--- feature/relatorios
```

## Capítulo 01.5 — Merge

Estratégias de integração entre branches.

## Capítulo 01.6 — Rebase

Quando utilizar rebase e seus riscos.

## Capítulo 01.7 — Conflitos

Diagnóstico e resolução segura de conflitos.

## Capítulo 01.8 — Tags

Versionamento de releases.

---

# VOLUME 02 — GitHub e Pull Requests

## Capítulo 02.1 — Estrutura de um repositório GitHub

Organização profissional de um projeto.

## Capítulo 02.2 — Pull Request

Ciclo completo de uma PR.

```text
Branch
  |
  v
Commits
  |
  v
PR
  |
  +--> revisão
  +--> testes
  +--> comentários
  +--> correções
  |
  v
Merge
```

## Capítulo 02.3 — PRs independentes

Como desenvolver várias funcionalidades simultaneamente.

## Capítulo 02.4 — PRs dependentes

Tratamento de alterações encadeadas.

## Capítulo 02.5 — Code Review

Processo técnico de revisão.

## Capítulo 02.6 — Branch Protection

Proteção do branch principal.

## Capítulo 02.7 — CODEOWNERS

Definição de responsáveis por partes do código.

## Capítulo 02.8 — Issues e PRs

Rastreabilidade entre necessidade, implementação e entrega.

## Capítulo 02.9 — Templates

Templates para Issues e Pull Requests.

---

# VOLUME 03 — GitHub Actions

## Capítulo 03.1 — Conceitos

Introdução aos workflows.

## Capítulo 03.2 — Estrutura YAML

Anatomia de:

```text
.github/
└── workflows/
    ├── ci.yml
    ├── e2e.yml
    └── deploy.yml
```

## Capítulo 03.3 — Events

Execução em:

- push;
- pull_request;
- workflow_dispatch;
- schedule;
- release.

## Capítulo 03.4 — Jobs

Separação lógica das tarefas.

## Capítulo 03.5 — Steps

Execução dos comandos.

## Capítulo 03.6 — Actions reutilizáveis

Uso de componentes existentes.

## Capítulo 03.7 — Secrets

Gerenciamento seguro de credenciais. Inclui OIDC (OpenID Connect) como alternativa preferencial a secrets estáticos de longa duração para autenticação com provedores cloud (AWS, Azure, GCP) — o workflow recebe um token de curta duração assinado pelo GitHub, sem chave permanente armazenada no repositório.

## Capítulo 03.8 — Variables

Configurações não secretas.

## Capítulo 03.9 — Artifacts

Persistência de resultados de builds e testes.

## Capítulo 03.10 — Cache

Cache de npm, Composer e dependências.

## Capítulo 03.11 — Matrices

Testes com múltiplas versões.

## Capítulo 03.12 — Workflows e composite actions reutilizáveis

Redução de duplicação via `workflow_call` (workflows reutilizáveis) e composite actions locais.

## Capítulo 03.13 — Permissões do `GITHUB_TOKEN`

Uso do bloco `permissions:` no nível de workflow e de job para aplicar o princípio do menor privilégio — o padrão recomendado é `permissions: {}` no topo do arquivo e elevar apenas o necessário (`contents: read`, `pull-requests: write` etc.) em cada job.

## Capítulo 03.14 — Versionamento de actions e sintaxe atual

Fixação de actions em versões major suportadas (ex.: `actions/checkout@v4`, `actions/setup-node@v4`, `actions/upload-artifact@v4`) e substituição de comandos legados (`::set-output::`, `::save-state::`, `::set-env::`) pelos arquivos de ambiente `$GITHUB_OUTPUT`, `$GITHUB_STATE` e `$GITHUB_ENV`, obrigatórios desde a desativação dos comandos antigos.

---

# VOLUME 04 — Self-Hosted Runners

Este será um dos volumes centrais do projeto.

## Capítulo 04.1 — O que é um runner

Diferença entre GitHub-hosted e self-hosted.

## Capítulo 04.2 — Arquitetura recomendada

```text
GitHub
   |
   | HTTPS
   v
Self-Hosted Runner
Ubuntu Linux
   |
   +--> Docker
   +--> Node.js
   +--> PHP
   +--> Composer
   +--> npm
   +--> Playwright
   +--> ferramentas de teste
```

## Capítulo 04.3 — Requisitos de hardware

Dimensionamento de CPU, RAM, disco e rede.

## Capítulo 04.4 — Ubuntu Server

Preparação do sistema operacional.

## Capítulo 04.5 — Criação de usuário dedicado

Execução do runner sem privilégios desnecessários.

## Capítulo 04.6 — Instalação oficial do GitHub Actions Runner

Download, registro e configuração.

## Capítulo 04.7 — Runner como serviço systemd

Inicialização automática.

## Capítulo 04.8 — Labels

Exemplo:

```yaml
runs-on:
  - self-hosted
  - linux
  - docker
```

## Capítulo 04.9 — Múltiplos runners

Escalabilidade.

## Capítulo 04.10 — Segurança

Isolamento, permissões e riscos de execução de código. Ponto crítico: self-hosted runners **não devem** ser habilitados para workflows disparados por `pull_request` em repositórios públicos sem revisão — um PR malicioso de um fork pode executar código arbitrário no runner. Cuidados obrigatórios:

- restringir execução de workflows em PRs de forks (aprovação manual do mantenedor antes de rodar);
- nunca usar `pull_request_target` combinado com checkout do código do fork sem sanitização — esse evento roda com o contexto (e secrets) do branch base, ampliando o risco;
- runners self-hosted em repositórios públicos, quando inevitáveis, devem ser efêmeros (uma execução por VM/container descartável) para evitar persistência de artefatos maliciosos entre jobs;
- isolar o runner em rede própria, sem acesso direto a outros sistemas internos.

## Capítulo 04.11 — Atualizações

Manutenção do runner, incluindo acompanhamento do fim de suporte de versões de Node.js usadas pelo runner e pelas actions (ex.: Node 16 já descontinuado; runners atuais exigem Node 20+).

## Capítulo 04.12 — Monitoramento

Verificação de disponibilidade.

## Capítulo 04.13 — Backup e recuperação

Reconstrução rápida de runners.

---

# VOLUME 05 — Docker no Pipeline

## Capítulo 05.1 — Fundamentos

Containers versus máquinas virtuais.

## Capítulo 05.2 — Docker Engine

Instalação no Ubuntu.

## Capítulo 05.3 — Dockerfile

Criação de imagens reproduzíveis.

## Capítulo 05.4 — Docker Compose

Ambientes multi-container.

## Capítulo 05.5 — Node.js em Docker

Boas práticas.

## Capítulo 05.6 — PHP em Docker

PHP-FPM, Apache/Nginx e Composer.

## Capítulo 05.7 — MySQL/MariaDB

Banco de testes.

## Capítulo 05.8 — MQTT

Uso de Mosquitto em pipelines de automação/IoT.

## Capítulo 05.9 — Redes Docker

Isolamento dos serviços.

## Capítulo 05.10 — Volumes

Persistência e dados temporários.

## Capítulo 05.11 — Docker Compose para testes

Ambiente descartável por execução.

---

# VOLUME 06 — Estratégia Profissional de Testes

## Capítulo 06.1 — Pirâmide de testes

```text
          /\
         /E2E\
        /----\
       / API  \
      /--------\
     /Integração\
    /------------\
   /   Unitários  \
  /________________\
```

## Capítulo 06.2 — Testes unitários

Testes rápidos e isolados.

## Capítulo 06.3 — Testes de integração

Comunicação entre componentes.

## Capítulo 06.4 — Testes de API

Validação de endpoints.

## Capítulo 06.5 — Testes E2E

Fluxo completo do usuário.

## Capítulo 06.6 — Playwright

Automação moderna de navegadores.

## Capítulo 06.7 — Cypress

Alternativa para testes de frontend.

## Capítulo 06.8 — PHPUnit

Testes para aplicações PHP.

## Capítulo 06.9 — Jest/Vitest

Testes JavaScript/Node.js.

## Capítulo 06.10 — Smoke Tests

Conjunto mínimo para verificar funcionamento básico.

## Capítulo 06.11 — Regression Tests

Proteção contra regressões.

## Capítulo 06.12 — Testes seletivos

Executar somente testes afetados pela alteração.

## Capítulo 06.13 — Paralelização

Redução do tempo total.

## Capítulo 06.14 — Testes noturnos

Execução de suítes extensas fora do ciclo da PR.

---

# VOLUME 07 — Pipeline CI

## Capítulo 07.1 — Pipeline mínimo

```text
Checkout
   |
Install
   |
Lint
   |
Unit
   |
Integration
   |
Build
```

## Capítulo 07.2 — Pipeline para Node.js

Exemplo completo.

## Capítulo 07.3 — Pipeline para PHP

Composer, PHPUnit e análise estática.

## Capítulo 07.4 — Pipeline híbrido

Projetos Node.js + PHP.

## Capítulo 07.5 — Banco de dados temporário

MySQL/MariaDB no CI.

## Capítulo 07.6 — MQTT no CI

Testes de aplicações que utilizam broker.

## Capítulo 07.7 — Quality Gates

Critérios objetivos para aprovação.

---

# VOLUME 08 — Desenvolvimento Orientado por Especificação e IA

Este volume documentará o fluxo de desenvolvimento assistido por agentes.

## Capítulo 08.1 — Da ideia à especificação

Transformação de uma necessidade em requisitos claros.

## Capítulo 08.2 — Entrevista de requisitos pela IA

Perguntas antes da implementação.

## Capítulo 08.3 — SPEC

Estrutura recomendada:

```text
Objetivo
Escopo
Requisitos
Restrições
Critérios de aceitação
Casos de teste
Fora de escopo
```

## Capítulo 08.4 — Revisão da SPEC

Validação humana.

## Capítulo 08.5 — Plano de implementação

Transformação da especificação em tarefas técnicas.

## Capítulo 08.6 — Implementação assistida

Uso de agentes para produção do código.

## Capítulo 08.7 — Revisão automática

Análise do código antes da PR.

## Capítulo 08.8 — IA e testes

Geração e execução de testes.

## Capítulo 08.9 — IA e E2E

Validação dos critérios de aceitação.

## Capítulo 08.10 — Alteração posterior de funcionalidades

Nova SPEC referenciando PRs anteriores.

Exemplo:

```text
PR original: #42
        |
        v
Nova necessidade
        |
        v
SPEC de refinamento
        |
        v
Nova PR #57
```

## Capítulo 08.11 — Contexto e documentação

Como impedir que agentes alterem partes fora do escopo.

## Capítulo 08.12 — Desenvolvimento paralelo com agentes

Uso seguro de múltiplas branches e PRs.

---

# VOLUME 09 — Continuous Deployment

## Capítulo 09.1 — Conceitos de CD

Diferença entre entrega e implantação contínua.

## Capítulo 09.2 — Ambiente DEV

Deploy automático após validação.

## Capítulo 09.3 — Validação em DEV

Testes pós-deploy.

## Capítulo 09.4 — Gate humano

Modelo adotado:

```text
CI passou
   |
Deploy DEV
   |
Validação
   |
   v
Aprovar produção?
   |
 +----+----+
 |         |
Não       Sim
           |
           v
         PROD
```

## Capítulo 09.5 — GitHub Environments

Proteção de ambientes.

## Capítulo 09.6 — Deploy via SSH

Procedimento seguro.

## Capítulo 09.7 — Deploy com Docker

Atualização de containers. Pode ser feito diretamente via SSH/Docker Compose ou delegado a um orquestrador de deploy (ex.: Coolify), que recebe o webhook do GitHub Actions e cuida do build, health check e substituição do container — reduzindo a lógica de deploy mantida dentro do próprio workflow.

## Capítulo 09.8 — Blue/Green Deployment

Estratégia futura para sistemas críticos.

## Capítulo 09.9 — Rollback

Retorno rápido à versão anterior.

---

# VOLUME 10 — Segurança do Pipeline

## Capítulo 10.1 — Threat Model

Identificação das superfícies de ataque.

## Capítulo 10.2 — Secrets

Proteção de senhas, tokens e chaves.

## Capítulo 10.3 — SSH

Chaves dedicadas ao CI/CD.

## Capítulo 10.4 — Princípio do menor privilégio

Permissões mínimas, incluindo o bloco `permissions:` do `GITHUB_TOKEN` (workflow e por job) e uso de OIDC em vez de secrets estáticos para autenticação com serviços externos.

## Capítulo 10.5 — Segurança de self-hosted runners

Riscos específicos.

## Capítulo 10.6 — Dependências

Análise de vulnerabilidades.

## Capítulo 10.7 — Dependabot

Atualizações automáticas.

## Capítulo 10.8 — Secret scanning

Detecção de credenciais expostas.

## Capítulo 10.9 — Code scanning

Análise estática.

## Capítulo 10.10 — Supply Chain Security

Proteção da cadeia de software.

---

# VOLUME 11 — Observabilidade

## Capítulo 11.1 — Logs

Centralização e retenção.

## Capítulo 11.2 — Métricas

CPU, memória, containers e aplicações.

## Capítulo 11.3 — Prometheus

Coleta de métricas.

## Capítulo 11.4 — Grafana

Dashboards.

## Capítulo 11.5 — Loki

Centralização de logs.

## Capítulo 11.6 — Alertas

Detecção de falhas após deploy.

## Capítulo 11.7 — Health Checks

Verificação automática de serviços.

---

# VOLUME 12 — Infraestrutura do Servidor CI/CD

## Capítulo 12.1 — Servidor físico ou VM

Critérios de escolha.

## Capítulo 12.2 — Dimensionamento

CPU, RAM, SSD e rede.

## Capítulo 12.3 — Linux Ubuntu Server

Configuração base.

## Capítulo 12.4 — Firewall

UFW/nftables.

## Capítulo 12.5 — SSH

Hardening.

## Capítulo 12.6 — Docker

Runtime principal.

## Capítulo 12.7 — Reverse Proxy

Nginx ou Traefik.

## Capítulo 12.8 — DNS e TLS

HTTPS e certificados.

## Capítulo 12.9 — Backup

Estratégia de recuperação.

## Capítulo 12.10 — UPS e disponibilidade

Proteção da infraestrutura local.

---

# VOLUME 13 — Arquiteturas de Referência

## Capítulo 13.1 — Projeto Node.js simples

CI/CD completo.

## Capítulo 13.2 — Node.js + MySQL

Aplicação com banco.

## Capítulo 13.3 — PHP + MariaDB

Pipeline para aplicações PHP existentes.

## Capítulo 13.4 — Node.js + MQTT

Aplicações de automação.

## Capítulo 13.5 — PHP + MQTT

Integração com sistemas legados.

## Capítulo 13.6 — Aplicação frontend + backend

Testes completos.

## Capítulo 13.7 — Microsserviços Docker

Pipeline multi-container.

---

# VOLUME 14 — Otimização e Escalabilidade

## Capítulo 14.1 — Redução do tempo de CI

Medição e otimização.

## Capítulo 14.2 — Cache avançado

npm, Composer, Docker e Playwright.

## Capítulo 14.3 — Testes afetados

Seleção baseada nas mudanças da PR.

## Capítulo 14.4 — Paralelização

Distribuição de testes.

## Capítulo 14.5 — Múltiplos runners

Pool de execução.

## Capítulo 14.6 — Runner dedicado para E2E

Separação das cargas pesadas.

## Capítulo 14.7 — Runner efêmero

Execução isolada e descartável.

---

# VOLUME 15 — Governança e Operação

## Capítulo 15.1 — Política de branches

Regras formais.

## Capítulo 15.2 — Política de PR

Critérios para merge.

## Capítulo 15.3 — Definition of Done

Quando uma implementação está realmente concluída.

## Capítulo 15.4 — Releases

Controle de versões.

## Capítulo 15.5 — Changelog

Histórico de alterações.

## Capítulo 15.6 — Incidentes

Procedimentos quando um deploy falha.

## Capítulo 15.7 — Post-mortem

Aprendizado após incidentes.

---

# VOLUME 16 — Laboratório Completo

Este volume consolidará todo o conhecimento.

## Capítulo 16.1 — Preparação do servidor

Ubuntu limpo.

## Capítulo 16.2 — Docker

Instalação e configuração.

## Capítulo 16.3 — Runner

Registro no GitHub.

## Capítulo 16.4 — Repositório de laboratório

Projeto exemplo.

## Capítulo 16.5 — CI

Lint, testes e build.

## Capítulo 16.6 — E2E

Playwright.

## Capítulo 16.7 — Deploy DEV

Implantação automática.

## Capítulo 16.8 — Aprovação

Gate manual.

## Capítulo 16.9 — Deploy PROD

Implantação controlada.

## Capítulo 16.10 — Rollback

Simulação de falha.

## Capítulo 16.11 — Monitoramento

Logs, métricas e health checks.

---

# APÊNDICES

## Apêndice A — Comandos Git

Cheat sheet operacional.

## Apêndice B — GitHub CLI

Uso do `gh`.

## Apêndice C — Docker CLI

Comandos essenciais.

## Apêndice D — Linux

Comandos administrativos utilizados no laboratório.

## Apêndice E — YAML

Referência rápida.

## Apêndice F — Expressões GitHub Actions

Sintaxe de conditions e contexts.

## Apêndice G — Troubleshooting

Falhas recorrentes e diagnóstico.

## Apêndice H — Checklists

Checklists para:

- nova PR;
- novo workflow;
- novo runner;
- deploy DEV;
- deploy PROD;
- rollback;
- incidente.

---

# 4. Stack preferencial

Sempre que tecnicamente adequado, o projeto dará preferência a ferramentas de código aberto.

| Área | Ferramenta principal |
|---|---|
| Sistema operacional | Ubuntu Server |
| Versionamento | Git |
| Repositório | GitHub |
| CI/CD | GitHub Actions |
| Runner | GitHub Actions Runner self-hosted |
| Containers | Docker Engine |
| Orquestração local | Docker Compose |
| Backend | Node.js / PHP |
| Banco | MySQL / MariaDB |
| Testes JS | Vitest/Jest |
| Testes PHP | PHPUnit |
| E2E | Playwright |
| MQTT | Eclipse Mosquitto |
| Proxy | Nginx / Traefik |
| Orquestração de deploy | Coolify (opcional) |
| Autenticação cloud | OIDC (preferencial a secrets estáticos) |
| Métricas | Prometheus |
| Dashboards | Grafana |
| Logs | Loki |

Alguns componentes da plataforma GitHub são serviços proprietários, mas o guia priorizará software livre para a infraestrutura controlada pelo usuário.

---

# 5. Estratégia de execução dos testes

Um objetivo importante será evitar que o crescimento do sistema torne cada PR excessivamente lenta.

Pipeline recomendado:

```text
PR
 |
 +--> Lint
 +--> Unitários
 +--> Integração
 +--> Build
 +--> Smoke E2E
 |
 v
MERGE / DEV
 |
 +--> Testes de integração
 +--> E2E relevantes
 |
 v
APROVAÇÃO HUMANA
 |
 v
PRODUÇÃO
 |
 +--> Health Check
 +--> Smoke pós-deploy
```

Suítes completas e demoradas poderão ser executadas:

- sob demanda;
- antes de releases importantes;
- durante a madrugada;
- em runner dedicado.

---

# 6. Princípios arquiteturais

O projeto seguirá os seguintes princípios.

1. **Nada crítico deve depender apenas da memória do desenvolvedor.**
2. **Toda mudança significativa deve ser rastreável a uma PR.**
3. **Toda PR deve explicar por que a mudança existe.**
4. **Testes rápidos devem ocorrer cedo.**
5. **Testes caros devem ser executados de forma inteligente.**
6. **Produção exige um gate explícito.**
7. **Credenciais nunca devem existir no código-fonte.**
8. **O ambiente deve ser reproduzível.**
9. **Todo deploy deve possuir estratégia de rollback.**
10. **A IA auxilia a engenharia; critérios técnicos continuam explícitos e auditáveis.**
11. **Specs devem registrar o comportamento desejado antes de grandes alterações.**
12. **Refinamentos posteriores devem referenciar a implementação/PR original quando aplicável.**
13. **Workflows devem declarar `permissions:` mínimas explicitamente, nunca depender do padrão implícito do repositório.**
14. **Self-hosted runners nunca executam código de PRs de forks sem revisão humana prévia.**

---

# 7. Organização futura dos arquivos

Estrutura sugerida:

```text
docs/
├── 00_MASTER.md
│
├── 01-fundamentos/
│   ├── 01-git.md
│   ├── 02-repositorios.md
│   ├── 03-commits.md
│   └── 04-branches.md
│
├── 02-github/
├── 03-github-actions/
├── 04-self-hosted-runners/
├── 05-docker/
├── 06-testes/
├── 07-ci/
├── 08-ia-spec-driven/
├── 09-deployment/
├── 10-seguranca/
├── 11-observabilidade/
├── 12-infraestrutura/
├── 13-arquiteturas/
├── 14-otimizacao/
├── 15-governanca/
├── 16-laboratorio/
│
└── appendices/
```

---

# 8. Próximas etapas

A partir deste documento mestre, os capítulos poderão ser implementados independentemente.

Prioridade inicial recomendada:

```text
00_MASTER
    |
    v
04 — SELF-HOSTED RUNNER
    |
    v
03 — GITHUB ACTIONS
    |
    v
06 — TESTES
    |
    v
07 — PIPELINE CI
    |
    v
09 — DEPLOY
    |
    v
08 — IA / SPEC-DRIVEN DEVELOPMENT
```

Essa ordem permite construir primeiro uma infraestrutura funcional e, posteriormente, aprofundar a teoria e as otimizações.

---

# 9. Controle de evolução do guia

O documento será tratado como um projeto versionado.

Sugestão:

```text
v0.x  -> estrutura e primeiros laboratórios
v1.0  -> pipeline funcional completo
v1.x  -> segurança, observabilidade e otimizações
v2.0  -> arquitetura avançada e múltiplos runners
```

Cada expansão poderá alterar este índice mestre.

---

## Estado atual

**00_MASTER — criado**

Próximo documento sugerido:

**Volume 04 — Self-Hosted Runners: instalação profissional de um runner Linux Ubuntu com Docker e integração ao GitHub Actions.**
