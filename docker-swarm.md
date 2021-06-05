## Join swarm

```
docker swarm join-token manager
docker swarm join-token worker
```

## Create network

```
docker network create --driver overlay --subnet 10.1.0.0/16 --attachable private
```

## Set node labels

```
docker node update --label-add nginx=true `docker node inspect self --format "{{ .ID }}"`
docker node update --label-add pgsql=true `docker node inspect self --format "{{ .ID }}"`
docker node update --label-add redis=true `docker node inspect self --format "{{ .ID }}"`

docker node update --label-add proxy=true <NODE-NAME>
```

## Set service labels

```
docker service update --label-add nginx-server-name=www.example.com <SERVICE-NAME>
```

## Update service images

```
docker service update --image softvisio/<NAME> <SERVICE-NAME>
```
