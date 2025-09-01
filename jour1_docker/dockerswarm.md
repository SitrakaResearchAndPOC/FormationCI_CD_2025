Terminal clusterswarm1
```
docker info
```
```
docker swarm leave --force
```
```
docker swarm init
```
option : --listen-addr exist also </br>
copy the output as : docker join ... </br>
and add it on nano dockerjoin.txt

## Testing service whoami
IN ALL TERMINAL
```
docker pull emilevauge/whoami
```
Terminal clusterswarm1
```
docker service create --name web -p 80:80 emilevauge/whoami
```
```
docker service ps web
```
```
docker service rm web
```

## Testing service rolling update & rolling back & draining exquisite web

```
docker network create --driver overlay --opt encrypted exquisite
```
```
docker network ls
```
IN ALL TERMINAL
```
docker pull vdemeester/exquisite-words-java:v1
```
Terminal clusterswarm1
```
docker service create --name back --network exquisite --limit-memory 64M --reserve-memory 64M vdemeester/exquisite-words-java:v1
```
```
docker service ls
```
IN ALL TERMINAL
```
docker pull  mongo:3.3.8
```
Terminal clusterswarm1
```
docker service create --name mongo --network exquisite mongo:3.3.8
```
IN ALL TERMINAL
```
docker pull vdemeester/exquisite-web:v1
```
Terminal clusterswarm1
```
docker service create --name front --network exquisite -p 800:80 vdemeester/exquisite-web:v1
```
Tape in navigator <ip_node_manager:800>
```
docker service scale back=15
```
```
docker service ls
```
```
docker service ps back
```
Tape in navigator <ip_node_manager:800>
```
docker service scale front=2
```
```
bash print_services.sh
```
```
docker service ls
```
```
docker service ps front
```
Tape in navigator <ip_node_manager:800>
```
docker service update --update-parallelism 2 --update-delay 10s front
```
IN ALL TERMINAL
```
docker pull vdemeester/exquisite-web:v2
```
```
docker service update --image vdemeester/exquisite-web:v2 front
```
Tape in navigator <ip_node_manager:800>
```
docker service update --update-parallelism 2 --update-delay 10s back
```
IN ALL TERMINAL
```
docker pull  vdemeester/exquisite-words-java:v2 
```
Terminal clusterswarm1
```
docker service update --image vdemeester/exquisite-words-java:v2  back
```
Tape in navigator <ip_node_manager:800>
## Testing service web
```
docker service create --replicas 10 --name web  -p 80:80 nginx
```
```
docker node ps 
```
