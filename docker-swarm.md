# Docker swarm

### Join swarm

```sh
docker swarm join-token manager
docker swarm join-token worker
```

### Network

Create network:

```sh
docker network create --driver overlay --attachable main
```

Use host network:

```yaml
services:
  service:
    networks: [host]

networks:
  host: { name: host, external: true }
```

### Set node labels

```sh
docker node update --label-add nginx=true `docker node inspect self --format "{{ .ID }}"`
docker node update --label-add pgsql=true `docker node inspect self --format "{{ .ID }}"`
docker node update --label-add redis=true `docker node inspect self --format "{{ .ID }}"`

docker node update --label-add proxy=true <NODE-NAME>
```

### Set service labels

```sh
docker service update --label-add nginx-server-name=www.example.com <SERVICE-NAME>
```

### Update service images

```sh
docker service update --image softvisio/<NAME> <SERVICE-NAME>
```

### Continer log by name

```sh
docker logs $(docker container ps --filter name=$NAME --quiet) -f
```
