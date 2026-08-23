# Apêndice C — Docker CLI: referência operacional

## Informações

| Comando | Descrição |
|---|---|
| `docker version` | Versão do cliente e do daemon |
| `docker info` | Informações gerais do ambiente Docker |
| `docker system df` | Uso de disco por imagens, containers, volumes e cache |

## Containers

| Comando | Descrição |
|---|---|
| `docker ps` | Lista containers em execução |
| `docker ps -a` | Lista todos os containers, incluindo parados |
| `docker run --rm hello-world` | Executa um container e remove ao finalizar |
| `docker stop CONTAINER` | Para um container em execução |
| `docker start CONTAINER` | Inicia um container parado |
| `docker restart CONTAINER` | Reinicia um container |
| `docker rm CONTAINER` | Remove um container parado |
| `docker rm -f CONTAINER` | Força a remoção de um container, mesmo em execução |

## Logs e diagnóstico

| Comando | Descrição |
|---|---|
| `docker logs CONTAINER` | Exibe os logs de um container |
| `docker logs -f CONTAINER` | Acompanha os logs em tempo real |
| `docker inspect CONTAINER` | Detalhes completos de um container ou imagem |
| `docker stats` | Uso de recursos dos containers em execução |
| `docker exec -it CONTAINER sh` | Abre um shell interativo dentro do container |

## Imagens

| Comando | Descrição |
|---|---|
| `docker images` | Lista imagens locais |
| `docker pull IMAGE` | Baixa uma imagem de um registry |
| `docker build -t app:tag .` | Constrói uma imagem a partir do Dockerfile |
| `docker tag app:tag registry/app:tag` | Cria uma nova tag apontando para a mesma imagem |
| `docker push registry/app:tag` | Envia uma imagem para o registry |
| `docker rmi IMAGE` | Remove uma imagem local |

## Build multi-stage e Buildx

| Comando | Descrição |
|---|---|
| `docker build -f Dockerfile --target build -t app:build .` | Constrói apenas até o estágio nomeado `build` |
| `docker buildx create --use --name ci-builder` | Cria e ativa um builder do Buildx |
| `docker buildx inspect --bootstrap` | Inspeciona e inicializa o builder ativo |
| `docker buildx build --platform linux/amd64,linux/arm64 -t registry/app:tag --push .` | Build multi-arquitetura com push direto ao registry |
| `docker buildx build --cache-from type=registry,ref=registry/app:cache --cache-to type=registry,ref=registry/app:cache,mode=max -t registry/app:tag --push .` | Build com cache remoto de leitura e escrita |

Multi-stage (`FROM ... AS build` seguido de `FROM ... AS final` copiando com `COPY --from=build`) reduz o tamanho final da imagem e é o padrão recomendado para builds em CI.

## Volumes

| Comando | Descrição |
|---|---|
| `docker volume ls` | Lista volumes |
| `docker volume inspect VOLUME` | Detalhes de um volume |
| `docker volume rm VOLUME` | Remove um volume |

## Networks

| Comando | Descrição |
|---|---|
| `docker network ls` | Lista redes |
| `docker network inspect NETWORK` | Detalhes de uma rede |

## Compose

| Comando | Descrição |
|---|---|
| `docker compose config` | Valida e exibe a configuração resolvida |
| `docker compose up -d` | Sobe os serviços em segundo plano |
| `docker compose ps` | Lista os serviços do projeto |
| `docker compose logs` | Exibe os logs dos serviços |
| `docker compose logs -f` | Acompanha os logs dos serviços em tempo real |
| `docker compose pull` | Baixa as imagens dos serviços |
| `docker compose build` | Constrói as imagens dos serviços |
| `docker compose restart` | Reinicia os serviços |
| `docker compose up -d --build` | Reconstrói as imagens e sobe os serviços |
| `docker compose exec SERVICO sh` | Abre um shell interativo em um serviço |
| `docker compose down` | Para e remove os serviços |
| `docker compose down -v` | Para e remove os serviços, incluindo volumes |

## Projeto Compose isolado

| Comando | Descrição |
|---|---|
| `docker compose -p ci-123 -f compose.test.yml up -d` | Sobe um projeto isolado com nome e arquivo específicos |
| `docker compose -p ci-123 -f compose.test.yml down -v` | Derruba o projeto isolado, incluindo volumes |

## Limpeza conservadora

| Comando | Descrição |
|---|---|
| `docker container prune -f` | Remove containers parados |
| `docker image prune -f` | Remove imagens não referenciadas por nenhum container |
| `docker builder prune -f` | Remove cache de build não utilizado |
| `docker system df -v` | Uso de disco detalhado por item |

Não execute `docker system prune -a --volumes` nem `docker builder prune -a` em host compartilhado (ex.: runner self-hosted com múltiplos jobs concorrentes) sem entender o impacto: remove cache de build e imagens não referenciadas por qualquer container, mesmo de outros pipelines.

## Registry

| Comando | Descrição |
|---|---|
| `docker login REGISTRY` | Autentica no registry |
| `docker logout REGISTRY` | Encerra a sessão autenticada no registry |

Prefira `--password-stdin` em automações.

**Fim do Apêndice C**
