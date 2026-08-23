# Apêndice C — Docker CLI: Referência Operacional

## Informações
```bash
docker version
docker info
docker system df
```

## Containers
```bash
docker ps
docker ps -a
docker run --rm hello-world
docker stop CONTAINER
docker start CONTAINER
docker restart CONTAINER
docker rm CONTAINER
docker rm -f CONTAINER
```

## Logs e diagnóstico
```bash
docker logs CONTAINER
docker logs -f CONTAINER
docker inspect CONTAINER
docker stats
docker exec -it CONTAINER sh
```

## Imagens
```bash
docker images
docker pull IMAGE
docker build -t app:tag .
docker tag app:tag registry/app:tag
docker push registry/app:tag
docker rmi IMAGE
```

## Build multi-stage e Buildx
```bash
docker build -f Dockerfile --target build -t app:build .
docker buildx create --use --name ci-builder
docker buildx inspect --bootstrap
docker buildx build --platform linux/amd64,linux/arm64 -t registry/app:tag --push .
docker buildx build --cache-from type=registry,ref=registry/app:cache --cache-to type=registry,ref=registry/app:cache,mode=max -t registry/app:tag --push .
```

Multi-stage (`FROM ... AS build` seguido de `FROM ... AS final` copiando com `COPY --from=build`) reduz o tamanho final da imagem e é o padrão recomendado para builds em CI.

## Volumes
```bash
docker volume ls
docker volume inspect VOLUME
docker volume rm VOLUME
```

## Networks
```bash
docker network ls
docker network inspect NETWORK
```

## Compose
```bash
docker compose config
docker compose up -d
docker compose ps
docker compose logs
docker compose logs -f
docker compose pull
docker compose build
docker compose restart
docker compose up -d --build
docker compose exec SERVICO sh
docker compose down
docker compose down -v
```

## Projeto Compose isolado
```bash
docker compose -p ci-123 -f compose.test.yml up -d
docker compose -p ci-123 -f compose.test.yml down -v
```

## Limpeza conservadora
```bash
docker container prune -f
docker image prune -f
docker builder prune -f
docker system df -v
```

Não execute `docker system prune -a --volumes` nem `docker builder prune -a` em host compartilhado (ex.: runner self-hosted com múltiplos jobs concorrentes) sem entender o impacto: remove cache de build e imagens não referenciadas por qualquer container, mesmo de outros pipelines.

## Registry
```bash
docker login REGISTRY
docker logout REGISTRY
```

Prefira `--password-stdin` em automações.

**Fim do Apêndice C**
