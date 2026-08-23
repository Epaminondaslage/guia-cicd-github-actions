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
```

Não execute `docker system prune -a --volumes` em host compartilhado sem entender o impacto.

## Registry
```bash
docker login REGISTRY
docker logout REGISTRY
```

Prefira `--password-stdin` em automações.

**Fim do Apêndice C**
