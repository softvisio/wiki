# Docker swarm

### Join swarm

```shell
docker swarm join-token manager
docker swarm join-token worker
```

### Create network

```shell
docker network create --driver overlay --subnet 10.1.0.0/16 --attachable private
```

### Set node labels

```shell
docker node update --label-add nginx=true `docker node inspect self --format "{{ .ID }}"`
docker node update --label-add pgsql=true `docker node inspect self --format "{{ .ID }}"`
docker node update --label-add redis=true `docker node inspect self --format "{{ .ID }}"`

docker node update --label-add proxy=true <NODE-NAME>
```

### Set service labels

```shell
docker service update --label-add nginx-server-name=www.example.com <SERVICE-NAME>
```

### Update service images

```shell
docker service update --image softvisio/<NAME> <SERVICE-NAME>
```

### Continer log by name

```shell
docker logs $(d container ps --filter name= < NAME > --quiet) -f
```
