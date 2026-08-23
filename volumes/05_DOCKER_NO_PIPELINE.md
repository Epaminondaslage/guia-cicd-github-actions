# Volume 05 — Docker no Pipeline de CI/CD

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 05_DOCKER_NO_PIPELINE.md  
**Versão:** 0.1.0  
**Status:** Primeira versão para expansão incremental  
**Pré-requisitos:** Volumes 01 a 04

---

## 1. Objetivo

Este volume apresenta o uso de Docker como base de padronização, isolamento e empacotamento no pipeline de CI/CD.

O objetivo é evoluir de:

```text
código
 |
 v
instalação manual no servidor
```

para:

```text
código
 |
 v
CI
 |
 v
docker build
 |
 v
imagem versionada
 |
 +--> testes
 |
 +--> DEV
 |
 +--> PROD
```

A ideia central é simples:

> O mesmo software validado deve ser o software implantado.

---

# 2. Onde Docker entra no pipeline

Docker pode ser utilizado em diferentes pontos:

```text
Desenvolvimento local
        |
        v
Ambiente reproduzível

CI
        |
        v
Bancos / Redis / MQTT / serviços de teste

Build
        |
        v
Imagem da aplicação

DEV
        |
        v
Container da versão validada

PROD
        |
        v
Mesma imagem promovida
```

---

# 3. Docker não é máquina virtual

Uma VM normalmente inclui:

```text
hardware virtual
kernel
sistema operacional
aplicação
```

Um container utiliza o kernel do host e isola processos e recursos.

Conceitualmente:

```text
HOST LINUX
 |
 +-- Docker
      |
      +-- container app
      +-- container db
      +-- container redis
      +-- container mqtt
```

Containers tendem a ser mais leves que VMs completas.

---

# 4. Imagem e container

Uma **imagem** é um pacote imutável utilizado para criar containers.

```text
Dockerfile
    |
    v
docker build
    |
    v
Imagem
```

Um **container** é uma instância em execução:

```text
Imagem
  |
  v
docker run
  |
  v
Container
```

Podemos criar vários containers a partir da mesma imagem.

---

# 5. Dockerfile

Exemplo simples para Node.js:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

CMD ["npm", "start"]
```

Esse arquivo descreve como construir a imagem.

---

# 6. Build

```bash
docker build -t minha-app:dev .
```

Listar:

```bash
docker images
```

Executar:

```bash
docker run --rm -p 3000:3000 minha-app:dev
```

---

# 7. Tags de imagem

Evite depender apenas de:

```text
latest
```

Prefira versões identificáveis:

```text
app:1.4.0
app:git-a91c302
app:pr-42
```

No CI, o SHA do commit é uma excelente referência.

---

# 8. Imagem identificada pelo commit

Exemplo:

```text
suporteone:a91c302
```

Fluxo:

```text
commit a91c302
      |
      v
docker build
      |
      v
suporteone:a91c302
```

Assim é possível responder:

> Qual código está rodando?

---

# 9. Imutabilidade

Uma imagem publicada não deve ser silenciosamente substituída por conteúdo diferente usando a mesma identificação imutável.

Modelo desejado:

```text
SHA A
 |
 v
imagem A
 |
 +--> DEV
 |
 +--> PROD
```

Não:

```text
DEV -> build A

PROD -> novo build B
```

Mesmo que ambos tenham vindo teoricamente do mesmo código, reconstruções podem introduzir diferenças.

---

# 10. Build once, deploy many

Princípio:

```text
BUILD ONCE
    |
    v
artifact
    |
    +----> DEV
    |
    +----> PROD
```

No contexto Docker:

```text
docker image
```

é o artifact implantável.

---

# 11. .dockerignore

Crie:

```text
.dockerignore
```

Exemplo:

```text
.git
.github
node_modules
coverage
playwright-report
*.log
.env
.env.*
README.md
```

Objetivos:

- reduzir contexto do build;
- evitar arquivos desnecessários;
- evitar inclusão acidental de secrets;
- acelerar builds.

---

# 12. Nunca copiar secrets para a imagem

Evite:

```dockerfile
COPY .env /app/.env
```

A imagem pode preservar camadas e ser distribuída.

Secrets devem ser injetados em runtime por mecanismos apropriados.

---

# 13. Variáveis em runtime

Exemplo:

```bash
docker run \
  -e NODE_ENV=production \
  -e DATABASE_HOST=db \
  minha-app:a91c302
```

Em Compose:

```yaml
services:
  app:
    image: minha-app:a91c302
    environment:
      NODE_ENV: production
      DATABASE_HOST: db
```

Secrets reais exigem tratamento mais cuidadoso.

---

# 14. Docker Compose

Docker Compose descreve aplicações com múltiplos serviços.

Exemplo:

```yaml
services:

  app:
    build: .
    ports:
      - "3000:3000"

  db:
    image: mariadb:11

  redis:
    image: redis:alpine
```

Executar:

```bash
docker compose up -d
```

Parar:

```bash
docker compose down
```

---

# 15. Compose no desenvolvimento

Exemplo:

```text
docker compose
 |
 +-- app
 +-- mariadb
 +-- redis
 +-- mosquitto
```

Isso reduz instalações manuais no host.

---

# 16. Compose em testes

Arquivo separado:

```text
compose.test.yml
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

  mqtt:
    image: eclipse-mosquitto:2
```

---

# 17. Banco de teste descartável

Fluxo:

```text
CI
 |
 v
docker compose up
 |
 v
MariaDB temporário
 |
 v
migrations
 |
 v
testes
 |
 v
docker compose down -v
```

Isso evita usar banco DEV ou PROD para testes automatizados.

---

# 18. MQTT de teste

Para projetos com MQTT:

```text
CI
 |
 v
Mosquitto container
 |
 +-- aplicação publica
 |
 +-- teste assina
 |
 +-- assertions
```

O broker de produção não deve ser necessário para validar a lógica básica.

---

# 19. Healthcheck

Exemplo MariaDB:

```yaml
healthcheck:
  test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
  interval: 5s
  timeout: 3s
  retries: 20
```

O pipeline não deve presumir que um serviço está pronto apenas porque o processo do container iniciou.

---

# 20. Started não significa Ready

```text
container started
       |
       X
aplicação pronta
```

Um banco pode levar alguns segundos para aceitar conexões.

Use:

- healthchecks;
- scripts de espera;
- retries controlados.

Evite `sleep 30` como solução padrão.

---

# 21. Redes Docker

Compose cria normalmente uma rede para os serviços.

Exemplo:

```text
app
 |
 | hostname: db
 v
db
```

A aplicação pode acessar:

```text
db:3306
```

Não precisa conhecer o IP interno do container.

---

# 22. Não fixar IP de container

Evite:

```text
172.18.0.5
```

Prefira nomes DNS de serviço:

```text
db
redis
mqtt
```

IPs de containers podem mudar.

---

# 23. Volumes

Volumes preservam dados além da vida do container.

Exemplo:

```yaml
volumes:
  db_data:
```

Uso:

```yaml
services:
  db:
    volumes:
      - db_data:/var/lib/mysql
```

---

# 24. Volumes em CI

Para banco de testes descartável, muitas vezes queremos remover tudo ao final:

```bash
docker compose down -v
```

O `-v` remove volumes associados ao projeto Compose.

Use somente quando esses volumes realmente forem descartáveis.

---

# 25. Bind mounts

Exemplo:

```yaml
volumes:
  - ./src:/app/src
```

É útil em desenvolvimento.

Em produção, normalmente preferimos que o código esteja dentro da imagem.

---

# 26. Imagem de produção

A imagem de produção deve conter somente o necessário para executar a aplicação.

Evite incluir:

- testes;
- documentação desnecessária;
- caches;
- ferramentas de desenvolvimento;
- chaves;
- `.git`.

---

# 27. Multi-stage build

Exemplo:

```dockerfile
FROM node:22-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build


FROM node:22-alpine AS runtime

WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev

COPY --from=build /app/dist ./dist

USER node

CMD ["node", "dist/server.js"]
```

Benefícios:

- imagem final menor;
- separação build/runtime;
- menos ferramentas no runtime.

---

# 28. Usuário não-root

Evite executar aplicação como root dentro do container quando não houver necessidade.

Exemplo:

```dockerfile
USER node
```

Para imagens próprias, crie usuário dedicado quando necessário.

---

# 29. PID 1 e sinais

A aplicação deve responder corretamente a sinais de término.

Quando:

```bash
docker stop
```

o processo deve encerrar de forma limpa.

Isso é importante para:

- deploy;
- restart;
- rollback;
- manutenção.

---

# 30. Graceful shutdown

Uma aplicação Node.js deve tratar sinais quando necessário:

```text
SIGTERM
 |
 v
parar de aceitar novas requisições
 |
 v
fechar conexões
 |
 v
encerrar processo
```

Evite matar processos abruptamente sem necessidade.

---

# 31. Restart policy

Exemplo:

```yaml
restart: unless-stopped
```

Pode ser útil em DEV/PROD simples.

Mas restart automático não substitui observabilidade.

Se o processo reinicia 100 vezes, existe um problema.

---

# 32. Logs

Containers devem preferencialmente escrever logs em:

```text
stdout
stderr
```

Consultar:

```bash
docker logs nome-container
```

Compose:

```bash
docker compose logs
```

Follow:

```bash
docker compose logs -f
```

---

# 33. Não guardar logs indefinidamente

Docker pode consumir disco com logs.

Configure rotação conforme o ambiente.

Exemplo conceitual no daemon ou serviço:

```text
max-size
max-file
```

A política exata deve ser definida de acordo com o host.

---

# 34. Espaço em disco

Verifique:

```bash
docker system df
```

E:

```bash
df -h
```

Em self-hosted runners, builds frequentes acumulam:

- imagens;
- layers;
- containers;
- caches;
- volumes.

---

# 35. Limpeza controlada

Inspeção:

```bash
docker ps -a
docker images
docker volume ls
docker system df
```

Limpeza conservadora:

```bash
docker container prune -f
docker image prune -f
```

Não execute limpeza agressiva em host compartilhado sem avaliar impacto.

---

# 36. Build cache

Docker reutiliza camadas quando possível.

Por isso, ordem do Dockerfile importa.

Exemplo:

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

Se apenas `src` mudou, a camada de dependências pode ser reaproveitada.

---

# 37. Dockerfile ruim para cache

```dockerfile
COPY . .
RUN npm ci
```

Qualquer alteração no código invalida a camada anterior ao `npm ci`.

Isso pode tornar builds desnecessariamente lentos.

---

# 38. BuildKit

Docker moderno utiliza recursos avançados de build.

Com `buildx`, podemos ter:

- cache avançado;
- builds multi-plataforma;
- exportação de cache;
- melhor controle de build.

No pipeline inicial, começaremos simples e evoluiremos conforme necessidade.

---

# 39. Build no GitHub Actions

Exemplo:

```yaml
- name: Build image
  run: |
    docker build \
      -t suporteone:${{ github.sha }} \
      .
```

O runner precisa possuir Docker operacional.

---

# 40. Testar a imagem

Depois do build:

```yaml
- name: Run container
  run: |
    docker run -d \
      --name app-test \
      -p 3000:3000 \
      suporteone:${{ github.sha }}
```

Depois:

```yaml
- name: Health check
  run: curl --fail http://localhost:3000/health
```

---

# 41. Cleanup da imagem em teste

```yaml
- name: Cleanup
  if: always()
  run: |
    docker rm -f app-test 2>/dev/null || true
```

Em runner persistente, cleanup é obrigatório como disciplina operacional.

---

# 42. Nomes únicos

Duas execuções simultâneas não devem disputar:

```text
container app-test
porta 3000
volume db_data
```

Use identificadores únicos.

Exemplo conceitual:

```text
app-${RUN_ID}
```

---

# 43. GitHub run ID

O contexto do workflow fornece identificadores úteis.

Exemplo:

```yaml
docker build -t app:${{ github.sha }} .
```

Para nomes temporários, podemos usar valores específicos da execução.

O objetivo é evitar colisões.

---

# 44. Compose project name

Compose permite separar stacks usando nome de projeto.

Exemplo:

```bash
docker compose \
  -p ci-12345 \
  -f compose.test.yml \
  up -d
```

Cleanup:

```bash
docker compose \
  -p ci-12345 \
  -f compose.test.yml \
  down -v
```

Isso melhora isolamento entre jobs.

---

# 45. Portas e concorrência

Se dois jobs publicarem:

```text
3000:3000
```

no mesmo host, haverá conflito.

Alternativas:

- portas dinâmicas;
- redes internas;
- runner exclusivo por job;
- múltiplas VMs;
- não publicar portas quando não necessário.

---

# 46. Testes dentro da rede Compose

Uma arquitetura melhor pode executar o próprio teste em container.

```text
Compose network
 |
 +-- app
 +-- db
 +-- mqtt
 +-- test-runner
```

Assim, não precisamos publicar todos os serviços no host.

---

# 47. Test runner containerizado

Exemplo conceitual:

```yaml
services:

  tests:
    build:
      context: .
      target: test
    depends_on:
      - db
      - mqtt
```

O pipeline executa:

```bash
docker compose run --rm tests
```

---

# 48. Vantagem do teste containerizado

Reduz dependência de ferramentas instaladas diretamente no runner.

Em vez de:

```text
runner precisa:
Node
PHP
browser
...
```

podemos mover parte disso para imagens.

O runner continua precisando de Docker.

---

# 49. Desvantagem

Containerizar tudo também adiciona:

- complexidade;
- tempo de build;
- debugging adicional;
- gerenciamento de cache.

Não transforme Docker em objetivo por si só.

Use onde melhora reprodutibilidade e isolamento.

---

# 50. Imagem de testes

Podemos ter um stage:

```dockerfile
FROM node:22 AS test
```

com:

- dependências dev;
- linters;
- test runners.

E outro:

```dockerfile
FROM node:22-alpine AS runtime
```

mais enxuto.

---

# 51. Exemplo multi-stage completo

```dockerfile
FROM node:22-alpine AS deps

WORKDIR /app
COPY package*.json ./
RUN npm ci


FROM deps AS test

COPY . .
RUN npm run lint
RUN npm test


FROM deps AS build

COPY . .
RUN npm run build


FROM node:22-alpine AS runtime

WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev

COPY --from=build /app/dist ./dist

ENV NODE_ENV=production

USER node

CMD ["node", "dist/server.js"]
```

O desenho real deve considerar framework, dependências nativas e processo de build do projeto.

---

# 52. Build target

Podemos construir um stage específico:

```bash
docker build --target test .
```

Ou:

```bash
docker build --target runtime -t app:sha .
```

Isso permite reutilizar o Dockerfile para finalidades diferentes.

---

# 53. Docker e PHP

Exemplo simples:

```dockerfile
FROM php:8.4-cli

WORKDIR /app

COPY . .

CMD ["php", "app.php"]
```

Aplicações web podem exigir:

- PHP-FPM;
- Nginx;
- extensões;
- Composer;
- permissões.

Esses detalhes devem ser modelados explicitamente.

---

# 54. Composer em multi-stage

Exemplo:

```dockerfile
FROM composer:2 AS vendor

WORKDIR /app
COPY composer.json composer.lock ./
RUN composer install \
    --no-dev \
    --no-interaction \
    --prefer-dist \
    --optimize-autoloader
```

Depois copiar `vendor` para a imagem runtime.

---

# 55. Aplicação Node + PHP

Se um sistema possui componentes distintos:

```text
frontend
backend-node
backend-php
worker
```

não é obrigatório colocá-los todos no mesmo container.

Prefira separar processos e responsabilidades quando fizer sentido.

---

# 56. Um processo principal por container

Regra prática:

```text
container app
container worker
container nginx
container db
```

em vez de um container gigantesco executando tudo.

Há exceções, mas a separação melhora operação.

---

# 57. Banco em produção

Banco de dados em Docker é possível, mas exige política própria para:

- volumes;
- backup;
- restore;
- upgrades;
- monitoramento;
- replicação;
- corrupção;
- disaster recovery.

Não trate banco como container descartável em produção.

---

# 58. Banco no CI

No CI, o objetivo é diferente.

O banco deve ser:

```text
rápido
isolado
reproduzível
descartável
```

Por isso containers são particularmente úteis.

---

# 59. Migrations

Pipeline de teste:

```text
db vazio
 |
 v
migrations
 |
 v
fixtures
 |
 v
testes
```

Isso valida se o banco pode ser construído a partir do código versionado.

---

# 60. Migrations em deploy

Em produção, migrations exigem estratégia.

Questões:

- são backward compatible?
- bloqueiam tabela?
- podem ser revertidas?
- aplicação antiga continua funcionando durante deploy?
- existe backup?

Não execute migration destrutiva automaticamente sem política.

---

# 61. Docker Registry

Um registry armazena imagens.

Fluxo:

```text
docker build
    |
    v
docker push
    |
    v
Registry
    |
    +--> DEV pull
    |
    +--> PROD pull
```

Exemplos de registries incluem serviços públicos, privados e registries integrados a plataformas de código.

---

# 62. Registry privado

Para aplicações internas:

```text
registry privado
```

é normalmente apropriado.

Credenciais devem ser tratadas como secrets.

---

# 63. GitHub Container Registry

Uma opção integrada ao ecossistema GitHub é o GitHub Container Registry.

O endereço e as permissões devem ser configurados conforme o repositório/organização.

O pipeline pode autenticar, construir e publicar imagens.

---

# 64. Login no registry

Conceitualmente:

```bash
echo "$TOKEN" | docker login REGISTRY \
  -u "$USER" \
  --password-stdin
```

Nunca:

```bash
docker login -p senha-em-texto
```

em scripts versionados.

---

# 65. Tagging strategy

Sugestão:

```text
app:sha-a91c302
app:1.4.0
```

Opcionalmente:

```text
app:main
```

como alias móvel.

A referência imutável deve continuar disponível.

---

# 66. latest

`latest` não significa necessariamente:

```text
mais recente
```

É apenas uma tag convencional.

Não dependa dela para auditoria.

---

# 67. Tag por branch

Para DEV:

```text
app:dev
```

pode ser um alias conveniente.

Mas o deploy deve registrar também o SHA real.

Exemplo:

```text
DEV alias -> sha-a91c302
```

---

# 68. Tag por release

Produção:

```text
app:v2.3.1
```

E também:

```text
app:sha-a91c302
```

Isso conecta versão comercial e commit.

---

# 69. Pipeline de build

```text
PR
 |
 v
testes
 |
 v
merge main
 |
 v
build image
 |
 v
scan
 |
 v
push registry
 |
 v
deploy DEV
```

---

# 70. Pipeline de promoção

Depois de DEV:

```text
imagem sha-a91c302
      |
      v
validação
      |
      v
aprovação
      |
      v
PROD usa sha-a91c302
```

Sem novo build.

---

# 71. Rollback com imagem

Se:

```text
PROD = sha-B
```

falha, podemos voltar para:

```text
PROD = sha-A
```

desde que:

- imagem A continue no registry;
- banco permaneça compatível;
- configuração seja compatível.

Rollback de aplicação não resolve automaticamente rollback de banco.

---

# 72. Compose DEV

Exemplo:

```yaml
services:

  app:
    image: registry/app:${APP_VERSION}
    restart: unless-stopped
    ports:
      - "3000:3000"
```

Arquivo `.env` do servidor:

```text
APP_VERSION=sha-a91c302
```

O pipeline pode atualizar a versão de forma controlada.

---

# 73. Não enviar .env de produção ao Git

No repositório:

```text
.env.example
```

No servidor:

```text
.env
```

O `.env` real deve permanecer fora do versionamento.

Para ambientes mais maduros, considere mecanismos específicos de secret management.

---

# 74. Configuração versus código

Imagem:

```text
código + runtime
```

Runtime:

```text
configuração do ambiente
```

Exemplo:

```text
mesma imagem
 |
 +-- DEV -> API_URL dev
 |
 +-- PROD -> API_URL prod
```

Isso facilita promoção do mesmo artifact.

---

# 75. Health endpoint

Aplicação web deveria expor algo como:

```text
/health
```

Resposta básica:

```json
{
  "status": "ok"
}
```

Uma versão mais útil pode validar dependências críticas com cuidado para não gerar cascatas indevidas.

---

# 76. Readiness e liveness

Conceitualmente:

**Liveness**

```text
o processo está saudável?
```

**Readiness**

```text
está pronto para receber tráfego?
```

Mesmo fora de Kubernetes, essa separação é útil.

---

# 77. Healthcheck Docker

Exemplo:

```dockerfile
HEALTHCHECK \
  --interval=30s \
  --timeout=3s \
  --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1
```

O comando disponível depende da imagem.

Não instale ferramentas extras apenas por conveniência sem avaliar tamanho e segurança.

---

# 78. Dependências no healthcheck

Evite healthcheck excessivamente complexo.

Se cada verificação consulta dez serviços externos, uma falha externa pode fazer todos os containers parecerem mortos.

Defina semântica clara.

---

# 79. Docker Compose profiles

Profiles podem separar serviços opcionais.

Exemplo conceitual:

```text
profile test
profile observability
```

Isso permite ativar componentes somente quando necessários.

---

# 80. Override files

Podemos ter:

```text
compose.yml
compose.dev.yml
compose.test.yml
compose.prod.yml
```

Mas muitos arquivos sobrepostos podem ficar difíceis de entender.

Prefira clareza à abstração excessiva.

---

# 81. Docker no self-hosted runner

Arquitetura:

```text
Ubuntu
 |
 +-- GitHub Runner
 |
 +-- Docker daemon
      |
      +-- containers de teste
      +-- builds
```

O usuário do runner precisa de acesso ao Docker.

Como visto no Volume 04, isso equivale a privilégio elevado no host.

---

# 82. Segurança do socket Docker

Acesso a:

```text
/var/run/docker.sock
```

permite controle amplo do daemon.

Não exponha esse socket a containers não confiáveis.

---

# 83. Docker-in-Docker

Docker-in-Docker (`dind`) é uma estratégia possível, mas adiciona complexidade.

Para o primeiro ambiente self-hosted, utilizaremos o daemon Docker do host de forma controlada.

Mais tarde podemos estudar isolamento maior.

---

# 84. Runner efêmero

Uma evolução de segurança é criar runners descartáveis.

```text
job
 |
 v
runner novo
 |
 v
execução
 |
 v
runner destruído
```

Isso reduz resíduos entre execuções.

Exige automação adicional.

---

# 85. Docker e runner efêmero

Uma arquitetura avançada pode provisionar runners em:

- VMs;
- containers;
- autoscaling;
- infraestrutura de nuvem.

Não será necessária na primeira implantação.

---

# 86. Supply chain

Imagem Docker também faz parte da cadeia de suprimentos.

Riscos:

```text
imagem base comprometida
dependência vulnerável
Action comprometida
registry comprometido
secret exposto
```

Segurança precisa cobrir todo o pipeline.

---

# 87. Imagens oficiais e confiáveis

Prefira imagens:

- oficiais;
- mantidas;
- com origem conhecida;
- atualizadas;
- compatíveis com a política de segurança.

Evite imagens aleatórias apenas porque parecem convenientes.

---

# 88. Fixar versões

Em vez de:

```dockerfile
FROM node:latest
```

prefira algo controlado:

```dockerfile
FROM node:22-alpine
```

Para reprodutibilidade ainda maior, imagens podem ser fixadas por digest.

---

# 89. Digest

Uma imagem pode ser referenciada por digest.

Conceitualmente:

```text
node@sha256:...
```

Isso identifica conteúdo específico.

É mais rígido que uma tag móvel.

---

# 90. Atualizações de imagem base

Fixar versões não significa nunca atualizar.

Fluxo:

```text
nova base
 |
 v
PR automática/manual
 |
 v
CI
 |
 v
E2E
 |
 v
merge
```

Assim a atualização continua controlada.

---

# 91. Vulnerability scanning

Imagens podem ser analisadas por scanners de vulnerabilidade.

Fluxo:

```text
build
 |
 v
scan
 |
 +-- política aceita -> publicar
 |
 +-- risco crítico -> bloquear
```

Ferramentas específicas serão tratadas no volume de segurança.

---

# 92. SBOM

SBOM significa Software Bill of Materials.

É um inventário dos componentes presentes no software/imagem.

Ajuda em:

- auditoria;
- vulnerabilidades;
- compliance;
- incident response.

---

# 93. Assinatura de imagens

Ambientes mais maduros podem assinar artifacts/imagens e verificar assinatura antes do deploy.

Isso fortalece a cadeia:

```text
CI confiável
 |
 v
imagem assinada
 |
 v
registry
 |
 v
deploy verifica
```

---

# 94. Dockerfile como código

Dockerfile deve passar por:

- code review;
- testes;
- versionamento;
- política de segurança.

Uma alteração em:

```dockerfile
FROM ...
```

pode ser tão relevante quanto alteração no código da aplicação.

---

# 95. Docker Compose como código

O mesmo vale para:

```text
compose.yml
```

Mudanças de:

- portas;
- volumes;
- networks;
- privileges;
- capabilities;
- mounts;

podem afetar segurança e disponibilidade.

---

# 96. Privileged

Evite:

```yaml
privileged: true
```

sem necessidade técnica forte.

Esse modo remove importantes barreiras de isolamento.

---

# 97. Capabilities

Linux capabilities permitem granularidade maior do que simplesmente executar privilegiado.

Princípio:

```text
conceder apenas o necessário
```

---

# 98. Read-only filesystem

Algumas aplicações podem rodar com filesystem somente leitura, usando volumes/tmpfs apenas onde precisam escrever.

Isso reduz superfície de ataque.

É uma otimização de hardening posterior.

---

# 99. Limites de recursos

Em Compose podemos considerar limites de recursos conforme modo de execução e suporte disponível.

Objetivo:

```text
container defeituoso
não deve consumir toda a máquina
```

No self-hosted runner, isso é particularmente importante.

---

# 100. OOM

Se o host ficar sem memória:

```text
E2E
+ banco
+ build
+ browser
```

podem causar OOM.

Monitore:

```bash
free -h
docker stats
```

Não atribua automaticamente falhas de browser ao código da aplicação.

---

# 101. docker stats

```bash
docker stats
```

Mostra:

- CPU;
- memória;
- rede;
- I/O.

É útil para diagnosticar testes pesados.

---

# 102. Compose ps

```bash
docker compose ps
```

Ajuda a verificar:

- estado;
- health;
- portas.

---

# 103. Compose logs

```bash
docker compose logs --tail=200
```

Em falha de CI, preserve logs antes do cleanup.

---

# 104. Coletar logs antes de destruir

Ordem:

```text
teste falhou
 |
 v
coletar logs
 |
 v
upload artifact
 |
 v
cleanup
```

Não:

```text
teste falhou
 |
 v
down -v
 |
 v
"cadê o erro?"
```

---

# 105. Script de diagnóstico

Exemplo:

```bash
#!/usr/bin/env bash

docker compose ps
docker compose logs --no-color --tail=300
docker system df
```

Pode ser chamado com:

```yaml
if: failure()
```

---

# 106. Testes Playwright com Docker

Duas estratégias:

```text
A) browser instalado no runner
```

ou:

```text
B) ambiente/browser containerizado
```

A escolha depende de:

- performance;
- manutenção;
- isolamento;
- compatibilidade.

---

# 107. Browser no runner

Vantagem:

- menor overhead de container em alguns cenários;
- integração simples com setup existente.

Desvantagem:

- runner precisa manter browsers/dependências.

---

# 108. Browser containerizado

Vantagem:

- ambiente reproduzível.

Desvantagem:

- networking e permissões podem ficar mais complexos;
- imagens podem ser grandes.

---

# 109. Estratégia inicial

Para o pipeline atual:

```text
Runner Ubuntu
 |
 +-- Docker para serviços
 |
 +-- Playwright no runner
```

é uma abordagem simples.

Depois podemos medir e decidir se vale containerizar o E2E.

---

# 110. Workflow de integração com Compose

```yaml
name: Integration

on:
  pull_request:

jobs:

  integration:
    runs-on:
      - self-hosted
      - linux
      - docker

    timeout-minutes: 20

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - run: npm ci

      - name: Subir ambiente
        run: |
          docker compose \
            -p ci-${{ github.run_id }} \
            -f compose.test.yml \
            up -d

      - name: Testar
        run: npm run test:integration

      - name: Logs em falha
        if: failure()
        run: |
          docker compose \
            -p ci-${{ github.run_id }} \
            -f compose.test.yml \
            logs --no-color

      - name: Cleanup
        if: always()
        run: |
          docker compose \
            -p ci-${{ github.run_id }} \
            -f compose.test.yml \
            down -v
```

---

# 111. Build da imagem após testes

Podemos estruturar:

```text
quality
 |
 v
integration
 |
 v
build-image
```

YAML conceitual:

```yaml
build-image:
  needs:
    - quality
    - integration
```

Assim não construímos imagem final se testes básicos já falharam.

---

# 112. Testar a própria imagem final

Uma estratégia mais forte:

```text
build imagem
 |
 v
subir imagem
 |
 v
smoke test
 |
 v
publicar
```

Isso verifica o artifact real.

---

# 113. Não testar somente o source tree

Se CI testa:

```text
npm start local
```

mas produção roda:

```text
Docker image
```

podem existir diferenças.

Idealmente, pelo menos uma etapa deve validar a imagem final.

---

# 114. Pipeline recomendado

```text
PR
 |
 +-- lint
 +-- unit
 +-- integration
 +-- build Docker
 +-- smoke da imagem
 +-- E2E pertinente
 |
 v
merge
```

Após merge:

```text
build/publish artifact imutável
 |
 v
DEV
```

Dependendo da política, a imagem da PR pode ser descartada e a imagem de `main` construída uma vez para promoção.

---

# 115. Build na PR versus main

Na PR:

```text
build para validar Dockerfile
```

Após merge:

```text
build oficial
tag SHA
push registry
```

Isso mantém artifact oficial ligado ao commit integrado em `main`.

---

# 116. Workflow de publicação

Conceitualmente:

```yaml
on:
  push:
    branches:
      - main
```

Etapas:

```text
checkout
login registry
build
tag SHA
push
deploy DEV
```

---

# 117. Não publicar imagem quebrada

Antes do push oficial:

```text
testes obrigatórios
```

devem estar satisfeitos.

Também é possível testar a imagem construída antes de publicá-la.

---

# 118. Metadata da imagem

OCI/Docker labels podem registrar:

- source repository;
- commit;
- version;
- build date.

Isso melhora rastreabilidade.

---

# 119. Exemplo de labels

```dockerfile
LABEL org.opencontainers.image.source="REPOSITORIO"
LABEL org.opencontainers.image.revision="COMMIT"
```

Valores dinâmicos podem ser passados no build.

---

# 120. Build args

Exemplo:

```dockerfile
ARG GIT_SHA
LABEL org.opencontainers.image.revision=$GIT_SHA
```

Build:

```bash
docker build \
  --build-arg GIT_SHA="$GITHUB_SHA" \
  -t app:"$GITHUB_SHA" \
  .
```

Não use build args para secrets.

---

# 121. Build secrets

BuildKit oferece mecanismos específicos para secrets durante build.

Eles são preferíveis a `ARG` quando uma etapa realmente necessita credencial.

Mesmo assim, avalie se a dependência privada pode ser obtida de forma mais segura.

---

# 122. Secret em ARG é risco

Evite:

```dockerfile
ARG TOKEN
RUN comando --token=$TOKEN
```

Tokens podem vazar em histórico, logs ou metadados dependendo do processo.

Use mecanismos apropriados de build secret.

---

# 123. Docker e monorepo

Em monorepo:

```text
apps/
  api/
  frontend/
  worker/
```

podemos ter:

```text
Dockerfile.api
Dockerfile.frontend
Dockerfile.worker
```

ou Dockerfiles em cada diretório.

O pipeline pode construir apenas componentes afetados.

---

# 124. Build seletivo

Se somente:

```text
apps/frontend/
```

mudou, talvez não seja necessário reconstruir worker.

Essa otimização deve vir depois de termos pipeline correto e métricas.

---

# 125. Dependências compartilhadas

Monorepos complicam detecção de impacto:

```text
packages/common
```

pode afetar várias imagens.

Não faça seleção de build apenas por caminho sem entender dependências.

---

# 126. Docker Compose para DEV

Um DEV simples pode usar:

```text
compose.dev.yml
```

com:

- aplicação;
- banco DEV;
- Redis;
- MQTT;
- proxy.

Produção pode usar arquitetura diferente.

Não é obrigatório que DEV e PROD sejam idênticos, mas o artifact da aplicação deve ser compatível.

---

# 127. Reverse proxy

Nginx ou outro proxy pode ficar à frente:

```text
Internet/LAN
    |
    v
Nginx
    |
    v
app container
```

Responsabilidades possíveis:

- TLS;
- roteamento;
- headers;
- compressão;
- limites.

---

# 128. TLS

Não coloque certificados privados dentro da imagem da aplicação.

Certificados devem ser gerenciados pelo ambiente/proxy/secret store.

---

# 129. Deploy Compose

Exemplo conceitual:

```bash
export APP_VERSION=sha-a91c302

docker compose pull app
docker compose up -d app
```

Depois:

```bash
curl --fail http://localhost/health
```

---

# 130. Pull antes do up

Se a imagem está no registry:

```bash
docker compose pull
```

garante que o host obtenha a versão referenciada.

Depois:

```bash
docker compose up -d
```

---

# 131. Não usar docker compose down em todo deploy

Em produção, executar:

```bash
docker compose down
```

pode interromper todos os serviços.

Frequentemente é melhor atualizar apenas o serviço necessário.

A estratégia depende da arquitetura.

---

# 132. Zero downtime

Docker Compose simples não garante automaticamente zero downtime.

Para sistemas que exigem alta disponibilidade, serão necessárias estratégias adicionais:

- múltiplas instâncias;
- load balancer;
- blue/green;
- rolling deployment;
- orquestradores.

---

# 133. Blue/Green

Conceito:

```text
BLUE = versão atual
GREEN = nova versão
```

Fluxo:

```text
subir GREEN
 |
 v
testar
 |
 v
mudar tráfego
 |
 v
GREEN vira ativa
```

Rollback pode retornar tráfego para BLUE.

---

# 134. Canary

Conceito:

```text
nova versão recebe pequena parte do tráfego
```

Se saudável:

```text
aumentar gradualmente
```

É mais complexo e normalmente exige infraestrutura adicional.

---

# 135. Para o projeto inicial

Não precisamos começar com Kubernetes, blue/green ou canary.

Primeiro:

```text
imagem versionada
Compose
healthcheck
rollback documentado
gate humano
```

Isso já representa avanço significativo.

---

# 136. Rollback Compose

Se o servidor usa:

```text
APP_VERSION=sha-B
```

e falha:

```text
APP_VERSION=sha-A
docker compose pull
docker compose up -d
```

Depois:

```text
healthcheck
```

O processo deve ser automatizado e auditável posteriormente.

---

# 137. Banco e rollback

Problema:

```text
app B aplica migration destrutiva
 |
 v
rollback para app A
 |
 X
app A não entende novo banco
```

Por isso deploy de banco precisa de planejamento.

---

# 138. Expand/Contract

Estratégia de migrations compatíveis:

```text
1. EXPAND
adicionar estrutura nova sem quebrar antiga

2. MIGRATE
aplicação passa a usar estrutura nova

3. CONTRACT
remover estrutura antiga depois
```

Isso facilita deploy e rollback.

---

# 139. Backup antes de migration crítica

Para mudanças de alto risco:

```text
backup verificado
 |
 v
migration
```

Mas backup que nunca foi restaurado em teste não é garantia suficiente.

---

# 140. Docker não substitui backup

Volume Docker não é backup.

```text
volume
```

é armazenamento.

Backup exige cópia independente e procedimento de restore.

---

# 141. Observabilidade

Após deploy:

```text
health
logs
CPU
RAM
erros
latência
```

devem ser observados.

Deploy bem-sucedido no CLI não significa aplicação saudável.

---

# 142. Container status

```bash
docker ps
```

é apenas um sinal.

Um container `Up` pode conter uma aplicação funcionalmente quebrada.

Por isso:

```text
healthcheck + teste funcional
```

é melhor.

---

# 143. Smoke pós-deploy

Exemplo:

```text
GET /health
login técnico
consulta crítica
```

O smoke deve ser curto e seguro.

Não execute ações destrutivas em produção.

---

# 144. E2E pós-deploy

Se houver E2E em produção, utilize cenários explicitamente seguros.

Muitas suítes E2E devem continuar rodando em ambiente controlado, não diretamente contra dados reais.

---

# 145. Configuração DEV e PROD

Exemplo:

```text
DEV
DATABASE_HOST=db-dev
MQTT_HOST=mqtt-dev

PROD
DATABASE_HOST=db-prod
MQTT_HOST=mqtt-prod
```

A imagem permanece a mesma.

---

# 146. Docker e GitHub Environments

Secrets:

```text
development
production
```

podem fornecer configurações distintas ao processo de deploy.

O artifact não precisa conhecer os secrets no momento do build.

---

# 147. Princípio

```text
BUILD
não conhece secret de PROD
```

```text
DEPLOY PROD
recebe somente o necessário
```

Isso reduz exposição.

---

# 148. CI runner versus deploy runner

Arquitetura recomendada:

```text
runner-ci
 |
 +-- Docker build
 +-- tests
 +-- sem acesso PROD

runner-deploy
 |
 +-- deploy
 +-- acesso restrito
```

A separação será aprofundada no volume de segurança/deploy.

---

# 149. Docker daemon compartilhado

Se runner CI e aplicações permanentes usam o mesmo daemon:

```text
docker prune
```

pode afetar serviços.

Por isso, runner deve preferencialmente possuir host/VM dedicado.

---

# 150. Regra operacional

Nunca execute comando Docker destrutivo apenas porque apareceu em um tutorial.

Antes:

```text
docker ps
docker images
docker volume ls
docker system df
```

Entenda o host.

---

# 151. Docker Compose config

Antes de aplicar:

```bash
docker compose config
```

Isso mostra a configuração resolvida e ajuda a detectar erros.

Cuidado: valores sensíveis podem aparecer dependendo de como a configuração é fornecida.

Não publique essa saída indiscriminadamente.

---

# 152. Validação antes do deploy

Pipeline pode verificar:

```bash
docker compose config --quiet
```

quando suportado pelo ambiente/versão.

Isso detecta problemas de sintaxe/configuração antes da alteração operacional.

---

# 153. Versionar Compose

Arquivos de Compose devem estar no Git:

```text
infra/
  compose.yml
```

Exceto dados sensíveis.

Assim mudanças de infraestrutura passam por PR.

---

# 154. Infraestrutura revisável

Mudança:

```text
porta 3000 -> 80
```

ou:

```text
novo volume
```

deve aparecer no diff da PR.

Isso melhora auditoria.

---

# 155. Diretórios sugeridos

```text
.
├── .github/
│   └── workflows/
├── docker/
│   ├── app/
│   └── test/
├── scripts/
│   ├── ci/
│   └── deploy/
├── compose.yml
├── compose.test.yml
├── Dockerfile
└── .dockerignore
```

Não é obrigatório; adapte ao projeto.

---

# 156. Exemplo de scripts

```text
scripts/ci/build-image.sh
scripts/ci/run-integration.sh
scripts/ci/collect-logs.sh

scripts/deploy/deploy-dev.sh
scripts/deploy/deploy-prod.sh
scripts/deploy/rollback.sh
scripts/deploy/health-check.sh
```

O YAML fica responsável pela orquestração.

---

# 157. Script de build

```bash
#!/usr/bin/env bash

set -euo pipefail

IMAGE="$1"
VERSION="$2"

docker build \
  --build-arg GIT_SHA="$VERSION" \
  -t "${IMAGE}:${VERSION}" \
  .
```

Uso:

```bash
./scripts/ci/build-image.sh suporteone a91c302
```

---

# 158. Script de health check

```bash
#!/usr/bin/env bash

set -euo pipefail

URL="$1"

for i in $(seq 1 30); do
    if curl --fail --silent "$URL" >/dev/null; then
        echo "Health check OK"
        exit 0
    fi

    sleep 2
done

echo "Health check falhou"
exit 1
```

O número de tentativas deve refletir o comportamento real da aplicação.

---

# 159. Script de deploy conceitual

```bash
#!/usr/bin/env bash

set -euo pipefail

VERSION="$1"

export APP_VERSION="$VERSION"

docker compose pull app
docker compose up -d app

./scripts/deploy/health-check.sh \
  http://localhost/health
```

Em produção real, acrescente logs, rollback e locking.

---

# 160. Lock de deploy

Dois deploys simultâneos podem gerar condição de corrida.

Precisamos impedir:

```text
deploy A
    +
deploy B
```

no mesmo ambiente.

GitHub Actions `concurrency` pode ajudar.

---

# 161. Concurrency de DEV

Conceitualmente:

```yaml
concurrency:
  group: deploy-dev
  cancel-in-progress: false
```

Assim, deploys do mesmo ambiente não devem executar simultaneamente.

A política exata depende do fluxo desejado.

---

# 162. Concurrency de PROD

Produção deve ser ainda mais restrita.

```text
um deploy por vez
```

e com gate humano.

---

# 163. Artifact provenance

Uma evolução de segurança é registrar evidências de como o artifact foi construído.

Objetivo:

```text
imagem
 |
 +-- qual repo?
 +-- qual commit?
 +-- qual workflow?
 +-- qual builder?
```

Isso fortalece auditoria.

---

# 164. Retenção de imagens

Não remova imediatamente todas as imagens antigas do registry.

Precisamos manter versões suficientes para rollback.

Defina política:

```text
últimas N versões
+
releases
+
versões em produção
```

---

# 165. Garbage collection

Registry e hosts acumulam dados.

Políticas de limpeza devem preservar:

- versão atual;
- versão anterior;
- releases;
- versões em investigação.

Automação sem inventário pode apagar o rollback necessário.

---

# 166. Backup de registry

Se o registry for autogerenciado, ele também precisa de:

- backup;
- restore;
- monitoramento;
- controle de acesso.

Serviço gerenciado transfere parte dessa responsabilidade ao provedor.

---

# 167. Docker e arquitetura AMD64/ARM64

Se servidores possuem arquiteturas diferentes:

```text
amd64
arm64
```

podemos precisar de imagens multi-platform.

Buildx permite isso.

Não adicione multiarch se todos os hosts usam a mesma arquitetura.

---

# 168. Multi-platform

Exemplo conceitual:

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  ...
```

Build multiarch pode aumentar tempo de CI.

---

# 169. Compatibilidade nativa

Dependências com binários nativos podem se comportar diferente entre arquiteturas.

Testar no target real é importante.

---

# 170. Timezone

Containers frequentemente utilizam UTC por padrão.

A aplicação deve tratar timezone conscientemente.

Não resolva bugs de data apenas alterando timezone do container sem entender a regra de negócio.

---

# 171. Locale

Locale também pode diferir do host.

Testes devem evitar depender implicitamente de configurações locais.

---

# 172. UID/GID

Bind mounts podem gerar problemas de permissão entre host e container.

Isso é comum em desenvolvimento e runners.

Diagnóstico:

```bash
ls -ln
id
```

Não use `chmod 777` como solução automática.

---

# 173. Permissões de volume

Defina proprietário e permissões adequadas.

Em Dockerfile:

```dockerfile
COPY --chown=node:node . .
```

quando apropriado.

---

# 174. Rootless Docker

Docker possui modos e arquiteturas que reduzem necessidade de daemon root tradicional.

Pode ser estudado como hardening avançado.

Não é requisito para o primeiro runner.

---

# 175. Podman

Podman é uma alternativa open source para containers e possui modelo daemonless/rootless interessante.

Entretanto, este guia utilizará Docker inicialmente por compatibilidade direta com o ecossistema e o ambiente já planejado.

Podman poderá ser avaliado posteriormente.

---

# 176. Produtos open source do stack

Base recomendada:

| Componente | Uso |
|---|---|
| Docker Engine | Runtime/build de containers |
| Docker Compose | Stacks locais/DEV/test |
| BuildKit/Buildx | Builds avançados |
| MariaDB | Banco de teste |
| Redis | Cache/filas de teste |
| Eclipse Mosquitto | MQTT de teste |
| Nginx | Reverse proxy |
| Node.js | Runtime |
| PHP | Runtime |
| Playwright | E2E |
| Git | Versionamento |

---

# 177. Quando não usar Docker

Docker pode ser desnecessário quando:

- aplicação é extremamente simples;
- host já possui processo padronizado e imutável;
- restrições do ambiente impedem containers;
- custo operacional supera benefício.

O objetivo é resolver problemas, não adicionar tecnologia.

---

# 178. Quando Docker é especialmente útil

- múltiplos serviços;
- dependências complexas;
- CI;
- bancos descartáveis;
- MQTT/Redis;
- aplicações Node/PHP;
- ambientes DEV;
- artifacts versionados;
- deploy reproduzível.

---

# 179. Anti-pattern: servidor artesanal

Problema:

```text
servidor DEV
 |
 +-- Node instalado manualmente
 +-- PHP alterado manualmente
 +-- pacote esquecido
 +-- configuração não versionada
```

Resultado:

```text
ninguém sabe reproduzir
```

Docker reduz esse tipo de drift.

---

# 180. Configuration drift

Drift ocorre quando ambientes mudam ao longo do tempo sem rastreabilidade.

Exemplo:

```text
DEV recebeu ajuste manual
PROD não
```

Com imagem versionada:

```text
artifact conhecido
```

reduzimos parte desse problema.

---

# 181. Docker não elimina drift completamente

Ainda existem:

- host;
- daemon;
- volumes;
- secrets;
- Compose;
- firewall;
- proxy;
- kernel.

Por isso infraestrutura precisa de documentação e, futuramente, automação.

---

# 182. Pipeline alvo deste guia

```text
Developer
   |
   v
Branch
   |
   v
PR
   |
   v
GitHub Actions
   |
   v
Self-hosted Runner
   |
   +-- lint
   +-- unit
   +-- integration via Docker
   +-- build image
   +-- smoke image
   +-- E2E
   |
   v
Merge
   |
   v
Build oficial
   |
   v
Registry
   |
   v
DEV
   |
   v
Health + validação
   |
   v
Aprovação
   |
   v
PROD
```

---

# 183. Primeira implementação recomendada

Etapa 1:

```text
Docker somente para dependências de teste
```

Etapa 2:

```text
Dockerfile da aplicação
```

Etapa 3:

```text
build no CI
```

Etapa 4:

```text
smoke da imagem
```

Etapa 5:

```text
registry
```

Etapa 6:

```text
deploy DEV por imagem
```

Etapa 7:

```text
promoção da mesma imagem para PROD
```

---

# 184. Checklist Dockerfile

- [ ] Imagem base confiável.
- [ ] Versão controlada.
- [ ] `.dockerignore`.
- [ ] Dependências com lockfile.
- [ ] Cache de layers considerado.
- [ ] Multi-stage quando útil.
- [ ] Runtime sem ferramentas desnecessárias.
- [ ] Aplicação não roda como root sem necessidade.
- [ ] Nenhum secret incorporado.
- [ ] Logs em stdout/stderr.
- [ ] Processo encerra corretamente.
- [ ] Imagem pode ser identificada pelo commit.

---

# 185. Checklist Compose de teste

- [ ] Serviços isolados.
- [ ] Banco descartável.
- [ ] Sem conexão com PROD.
- [ ] Healthchecks.
- [ ] Nomes/projetos únicos.
- [ ] Sem portas fixas desnecessárias.
- [ ] Logs coletados em falha.
- [ ] Cleanup com `down -v`.
- [ ] Secrets de teste separados.
- [ ] Tempo limite definido.

---

# 186. Checklist CI

- [ ] Docker disponível no runner.
- [ ] Runner possui espaço em disco.
- [ ] Build utiliza tag imutável.
- [ ] Imagem final é testada.
- [ ] Cleanup não afeta outros serviços.
- [ ] Execuções concorrentes não colidem.
- [ ] Artifacts/logs são preservados.
- [ ] Scan poderá ser acrescentado.
- [ ] Imagem oficial só é publicada após gates.

---

# 187. Checklist deploy

- [ ] Imagem está no registry.
- [ ] Versão é conhecida.
- [ ] DEV usa versão explícita.
- [ ] Health check após deploy.
- [ ] PROD exige autorização.
- [ ] PROD usa a mesma imagem validada.
- [ ] Existe versão anterior disponível.
- [ ] Rollback documentado.
- [ ] Migrations são compatíveis.
- [ ] Secrets não estão na imagem.

---

# 188. Troubleshooting: build lento

Verifique:

```text
contexto enorme?
.dockerignore ausente?
COPY . antes do npm ci?
cache perdido?
imagem base grande?
dependências nativas?
download repetido?
```

Use métricas antes de alterar arquitetura.

---

# 189. Troubleshooting: container reiniciando

```bash
docker ps
docker logs CONTAINER
docker inspect CONTAINER
```

Verifique:

- exit code;
- configuração;
- conexão com banco;
- permissões;
- porta;
- memória.

---

# 190. Troubleshooting: banco não conecta

Verifique:

```text
hostname correto?
porta interna correta?
mesma network?
health?
credenciais?
migration?
```

Dentro do Compose, normalmente use nome do serviço, não `localhost`.

---

# 191. Localhost dentro do container

Dentro do container:

```text
localhost
```

significa o próprio container.

Se o banco está no serviço `db`:

```text
DB_HOST=db
```

não:

```text
DB_HOST=localhost
```

---

# 192. Troubleshooting: porta ocupada

```bash
ss -lntp
docker ps
```

Em CI, prefira evitar portas fixas compartilhadas.

---

# 193. Troubleshooting: permission denied

Verifique:

```bash
id
ls -ln
docker inspect
```

Não aplique `777` sem diagnóstico.

---

# 194. Troubleshooting: disco cheio

```bash
df -h
docker system df
du -sh /var/lib/docker/*
```

A última consulta pode exigir privilégios e deve ser usada apenas para diagnóstico.

---

# 195. Troubleshooting: imagem funciona local e não no servidor

Compare:

- arquitetura CPU;
- env vars;
- volumes;
- secrets;
- portas;
- DNS;
- firewall;
- filesystem;
- versão Docker;
- recursos.

A imagem reduz diferenças, mas não elimina diferenças do ambiente.

---

# 196. Laboratório 1

Criar uma aplicação simples Node.js:

```text
GET /health
```

Criar Dockerfile.

Build:

```bash
docker build -t app:lab .
```

Executar:

```bash
docker run --rm -p 3000:3000 app:lab
```

Testar:

```bash
curl http://localhost:3000/health
```

---

# 197. Laboratório 2

Adicionar:

```text
MariaDB
```

via Compose.

A aplicação deve conectar usando:

```text
DB_HOST=db
```

---

# 198. Laboratório 3

Adicionar Mosquitto:

```text
mqtt
```

e testar publicação/assinatura em ambiente isolado.

---

# 199. Laboratório 4

Criar:

```text
compose.test.yml
```

Executar no GitHub Actions self-hosted runner.

---

# 200. Laboratório 5

Buildar imagem com:

```text
github.sha
```

Subir a imagem e executar smoke test.

---

# 201. Laboratório 6

Configurar um registry e publicar:

```text
app:sha-...
```

Somente após os testes.

---

# 202. Laboratório 7

Implantar a imagem em DEV.

Validar:

```text
health
versão
logs
```

---

# 203. Laboratório 8

Simular rollback:

```text
versão B
 |
 falha
 |
 voltar A
```

Documentar o tempo e os comandos necessários.

---

# 204. Relação com Volume 06

Docker fornece infraestrutura para uma estratégia de testes melhor.

No próximo volume poderemos organizar:

```text
Unit
Integration
Contract
E2E Smoke
E2E Full
Visual
Performance
Security
```

e decidir:

```text
quando cada teste executa
```

---

# 205. Relação com Volume 09

O deploy futuro utilizará conceitos deste capítulo:

- artifact imutável;
- registry;
- versão por SHA;
- DEV;
- PROD;
- healthcheck;
- rollback;
- promoção.

---

# 206. Resumo mental

```text
Dockerfile
   |
   v
Image
   |
   v
Container
```

No CI:

```text
Code
 |
 v
Tests
 |
 v
Build
 |
 v
Image
 |
 v
Smoke
```

No CD:

```text
Image
 |
 v
Registry
 |
 +--> DEV
 |
 +--> PROD
```

Regra principal:

```text
MESMA IMAGEM
DEV -> PROD
```

---

# 207. Arquitetura acumulada dos volumes 01–05

```text
Git
 |
 v
Branch
 |
 v
PR
 |
 v
GitHub Actions
 |
 v
Self-Hosted Runner
 |
 v
Docker
 |
 +-- serviços de teste
 +-- build
 +-- imagem versionada
 |
 v
DEV
 |
 v
Aprovação
 |
 v
PROD
```

---

# 208. Próximo volume

**Volume 06 — Estratégia Profissional de Testes**

Conteúdo previsto:

- pirâmide de testes;
- unitários;
- integração;
- contratos;
- E2E;
- smoke;
- regressão completa;
- Playwright;
- testes visuais;
- flaky tests;
- paralelização;
- sharding;
- seleção de testes;
- dados de teste;
- isolamento;
- testes noturnos;
- quality gates;
- redução do tempo total do pipeline.

---

**Fim do Volume 05 — Docker no Pipeline de CI/CD**
