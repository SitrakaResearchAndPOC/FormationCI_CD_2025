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

## Docker stack : 
```
version: "3.8"

networks:
  exquisite:
    driver: overlay
    driver_opts:
      encrypted: ""

services:
  mongo:
    image: mongo:3.3.8
    networks:
      - exquisite
#    deploy:
#      placement:
#        constraints: [node.role == worker]  # Optionnel, pour isoler mongo

  back:
    image: vdemeester/exquisite-words-java:v1  # Dernière version de votre image backend
    networks:
      - exquisite
    environment:
      - MONGO_URI=mongodb://mongo:27017/exquisite  # Connexion à MongoDB
    depends_on:
      - mongo  # Le backend dépend de MongoDB
    deploy:
      replicas: 10
      resources:
        limits:
          memory: 64M
        reservations:
          memory: 64M
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  front:
    image: vdemeester/exquisite-web:v1  # Dernière version de votre image frontend
    ports:
      - "800:80"
    networks:
      - exquisite
    depends_on:
      - back  # Le frontend dépend du backend
    deploy:
      replicas: 10
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure   
```
```
docker-compose down --volumes --remove-orphans
```
```
docker system prune -f
```
```
docker stack deploy -c docker-compose.yml exquisite_stack
```

verifier etat du stack :
```
docker stack ps exquisite-app
```
Vérifier les logs :
```
docker service logs exquisite-app_<service-name>
```
Suprrimer stack : 
```
docker stack rm exquisite-app
```

* separation en mutiple fichiers

```
docker swarm init
```

```
docker network create --driver overlay --opt encrypted exquisite
```
Le reseau devrait crée via docker network ou via yml mais au moins avec un service

```
# mongo.yml
version: "3.8"

services:
  mongo:
    image: mongo:3.3.8
    networks:
      - exquisite
#    deploy:
#      placement:
#        constraints: [node.role == worker]  # Optionnel, pour isoler MongoDB sur des nœuds spécifiques
```

```
docker stack deploy --detach=true -c mongo.yml exquisite-mongo
```


```
# back.yml
version: "3.8"

services:
  back:
    image: vdemeester/exquisite-words-java:v1
    networks:
      - exquisite
    environment:
      - MONGO_URI=mongodb://mongo:27017/exquisite  # Connexion à MongoDB
    depends_on:
      - mongo  # Le backend dépend de MongoDB
    deploy:
      replicas: 10
      resources:
        limits:
          memory: 64M
        reservations:
          memory: 64M
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

networks:
  exquisite:
    external: true
```

```
docker stack deploy --detach=true -c back.yml exquisite-back
```


```
# front.yml
version: "3.8"

services:
  front:
    image: vdemeester/exquisite-web:v1
    ports:
      - "800:80"
    networks:
      - exquisite
    depends_on:
      - back  # Le frontend dépend du backend
    deploy:
      replicas: 10
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

networks:
  exquisite:
    external: true
```
```
docker stack deploy --detach=true -c front.yml exquisite-front
```

```
docker stack ps exquisite-mongo
```
```
docker stack ps exquisite-back
```
```
docker stack ps exquisite-front
```

* Amélioration avec volumes :



```
version: "3.8"

networks:
  exquisite:
    driver: overlay
    driver_opts:
      encrypted: ""

services:
  mongo:
    image: mongo:3.3.8
    networks:
      - exquisite
    volumes:
      - mongo_data:/data/db
    deploy:
      restart_policy:
        condition: on-failure

  back:
    image: vdemeester/exquisite-words-java:v1
    networks:
      - exquisite
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 64M
        reservations:
          memory: 64M
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
    environment:
      - MONGO_HOST=mongo
      - MONGO_PORT=27017
    depends_on:
      - mongo

  front:
    image: vdemeester/exquisite-web:v1
    ports:
      - "800:80"
    networks:
      - exquisite
    deploy:
      replicas: 2
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
    depends_on:
      - back

volumes:
  mongo_data:
``` 
``` 
docker stack deploy -c docker-compose.yml exquisite_stack
``` 

## Testing service web
```
docker service create --replicas 10 --name web  -p 80:80 nginx
```
```
docker node ps 
```
