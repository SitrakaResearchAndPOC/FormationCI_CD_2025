# Infrastructure
```
ssh root@45.79.212.220
cluster1
Lamp$$205909$$
```
```
ssh root@192.155.92.35
cluster2
Lamp$$205909$$
```
```
ssh root@192.155.92.55
cluster3
Lamp$$205909$$
```
```
ssh root@192.155.92.76
cluster4
Lamp$$205909$$
```

```
test_python
ssh root@139.177.206.42
Lamp$$205909$$
```
```
test_python-unbuntu-22.04
ssh root@139.177.206.172
Lamp$$205909$$
```


## Commande de base docker

# Exercice 2 : 
```
docker run --help
```
```
docker run -d --name c1 nginx:latest
```
```
docker ps 
```
```
docker stop c1
```
```
docker ps 
```
```
docker ps -a
```
```
docker start c1
```
```
docker ps 
```
-f car le conteneur est actif : 
```
docker rm -f c1
```
```
docker ps -a
```
Terminal interactif 
```
docker run -ti --name c1 nginx:latest
```
ça marche pas avec ls
```
ls
```
ctrl+c
```
docker rm -f c1
```
Le conteneur prevoit l'utilisation de terminal : 
```
docker run -ti debian:latest
```
```
apt update 
```
```
apt install git
```
```
hostname 
```
c'est un problème car c'est un id et non un nom : 
```         
exit
```
```
docker run -ti --name c1 debian:latest
```
```
docker ps -a 
```
Si je recrée un conteneur avec le même nom ca marche pas Erreur conflict deamon quand il y a deux c1
```
docker run -ti --name c1 debian:latest
```
Ajouter --rm : 
--rm : suppresion après arrêt du conteneur
```
docker rm -f c1
```
```
docker run -ti --rm --name c1 debian:latest
```
```
exit
```
```
docker ps -a
```
Avec hostname : 
```
docker run -ti  --rm --hostname host1 --name c1 debian:latest
```
```
hostname
```
```
exit
```
autres elements comme dns : (mode debug)  </br> 
docker run -ti  --rm --hostname host1 --name c1 --dns 8.8.8.8 debian:latest  </br> 
ASTUCES : 
```
docker ps -a
```
```
docker run -d --name c1 nginx
```
```
docker ps -a
```
```
docker ps
```
lister les id 
```
docker ps -q 
```
IMBRICATION DES COMMANDES POUR LA SUPPRESSION DES CONTENEURS
```
docker rm -f $(docker ps -q)
```
```
docker rm -f $(docker ps -aq)
```
IMBRICATION DES COMMANDES POUR LA SUPPRESSION DES IMAGES
```
docker rmi -f $(docker images -q)
```
```
docker rmi -f $(docker images -aq)
```

```
docker ps 
```
Publication de port : 
```
docker run -d --name c1 -p 8080:80  nginx:latest 
```
Entrer sur le navigateur : 
```
localhost:8080
```
## DEV with docker
```
docker run -itd --name serveur_nginx -p 8000:80 
nginx
```
```
docker exec -it serveur_nginx bash
```
```
cat /usr/share/nginx/html/index.html
```
Les deux commandes peuvent être simplifés par : 
```
docker exec -it serveur_nginx cat /usr/share/nginx/html/index.html
```

```
docker run -itd --name serveur_nginx -p 8000:80 
-v .\index.html:/usr/share/nginx/html/index.html nginx
```

## Docker network 
## Exerice 9 : Docker network (service dns)
* Verifier qu'une adresse IP n'est pas propre à un conteneur
* Utiliser un dns pour verifier la connexion entre conteneur

Les conteneurs n'ont pas des addresses ip statique  </br>
La destruction puis la recréation d'un conteneur ne donne pas une même addresse  </br>
```
docker rm -f c1
```
```
docker run -d --name c1 nginx:latest
```
```
docker inspect c1
```
```
docker stop c1
```
```
docker inspect c1
```
Pas d'addresse
```
docker inspect c1 | grep IPAddress
```
```
docker rm -f c2
```
```
docker run -d --name c2 nginx:latest
```
```
docker ps -a
```
```
docker inspect c2
```
c'est lui qui va prendre l'adresse IP
```
docker start c1
```
```
docker inspect c1
```
```
docker network ls
```
```
docker network create --driver=bridge --subnet=192.168.0.0/24 sitrakanet0
```
```
docker network ls
```
```
docker network inspect sitrakanet0
```
Créeons deux conteneurs :
```
docker rm -f c1
```
```
docker run -d --name c1 --network sitrakanet0 nginx:latest
```
```
docker rm -f c2
```
```
docker run -d --name c2 --network sitrakanet0 nginx:latest
```
```
docker ps
```
```
docker exec -ti c2 bash
```
```
apt update 
```
```
apt install iputils-ping
```
```
ping c1
```
```
exit
```
reessayez avec docker0
```
docker rm -f c1 c2
```
```
docker run -d --name c1  nginx:latest
```
```
docker run -d --name c2  nginx:latest
```
```
docker exec -ti c2 bash
```
```
apt update 
```
```
apt install iputils-ping  net-tools
```
```
ping c1
```
c'est normal qu'il n'y a pas interconnexion (pas de dns car default bridge)
```
ifconfig
```
```
exit
```
avant c'est utilisation link, aujourd'hui c'est docker connect
```
docker network connect sitrakanet0 c1
```
```
docker network connect sitrakanet0 c2
```
```
docker inspect c1
```
verifie l'addresse de c1
```
docker inspect c2
```
verifie l'addresse de c2
```
docker exec -ti c2 bash
```
```
ifconfig
```
```
ping c1
```
```
exit
```
Les drivers de types host :
```
docker rm -f c3
```
```
docker run -d --name c3 --network host nginx:latest
```
```
docker ps
```
```
curl 127.0.0.1
```
le nginx reponds sans mapping???
```
docker inspect c3
```
Pas d'addresse 
```
docker exec -ti c3 bash
```
```
apt update
```
```
apt install  net-tools
```
A l'interieur
```
ifconfig
```

Sortir vers l'exterieur :
```
exit
```
```
ifconfig
```
Conclusion : les configurations interieures et exterieures sont identiques en mode host
avec le mode host : </br>
Pas d'isolation réseau </br>
pas d'ip de conteneur </br>
ports uniques </br>

## variable d'environnement 
Lister les images via docker image ls
```
docker image ls
```
Pour avoir un tty, il faut toujours le mode detaché! </br>
Pour le variable d'environnement, il est possible d'utiliser --env ou -e </br>
```
docker run -tid --name testenv --env MYVARIABLE="123456" ubuntu:latest
```
```
docker ps
```
Pour rendre dans le conteneur, l'option exec
```
docker exec -ti testenv sh
```
OU 
```
docker exec -ti testenv bash
```
utiliser la commande env
```
env
```
```
echo $MYVARIABLE
```
```
ps
```
En combinant les deux directement :
```
docker exec -ti testenv env
```
## DockerFile
## Correction exercice 2 : dans slide : Dockerfile
* Création de répértoire de travail
</br>
POUR EVITER ERREUR : exec /home/docker/script/service_start.sh: no such file or directory</br>
</br>
AJOUTER notepad++ dans variable environnement/PATH de windows </br>
UTILISER notepad++ pour editer les codes et changer en UNIX LF et non Windows CRLF (en bas de notepad++) </br>

```
mkdir nginx-ubuntu
```
```
cd nginx-ubuntu
```

* Création de Dockerfile

```
nano Dockerfile
```
```
FROM ubuntu
MAINTAINER Josue R <josue.ratovondrahona@esti.mg>

RUN apt-get update && apt-get install nginx -y
COPY default /etc/nginx/sites-enabled/default
COPY index.html /var/www/html/index.html
COPY service_start.sh /home/docker/script/service_start.sh
RUN chmod +x /home/docker/script/service_start.sh
ENTRYPOINT ["/home/docker/script/service_start.sh"]

WORKDIR /home/docker
```
Tapez ctrl+x puis y -> entrer
* création du code html

```
nano index.html
```
```
<html>
	<title>Test Dockerfile</title>
	<body>
		<center><b>Test Dockerfile</b></center>
	</body>
</html >	
```
Tapez ctrl+x puis y -> entrer
* création de fichier de configuration default de nginx

```
nano default
```
```
server {
    listen 2080 default_server;
    listen [::]:2080 default_server;

    root /var/www/html;
    index index.html index.htm;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
Tapez ctrl+x puis y -> entrer
* création de script de demarrage

```
nano service_start.sh 
```
SANS MODE DETACHE
```
#!/bin/bash
echo LANCEMENT DE SERVEUR NGINX
nginx -g 'daemon off;'
```
Tapez ctrl+x puis y -> entrer
```
docker build -t viveticimg .
```
```
docker run -it --name viveticimg1 -p 8080:2080 viveticimg
```
RAFRAICHIR locahost:8080
```
exit
```
```
cd ..
```

OU AVEC MODE DETACHE
```
#!/bin/bash
echo "LANCEMENT DU SERVEUR NGINX"

# Lancer nginx en arrière-plan
service nginx start

# Ouvrir un shell interactif pour garder le conteneur actif
/bin/bash
```
Tapez ctrl+x puis y -> entrer
```
docker build -t viveticimg .
```
```
docker run -itd --name viveticimg1 -p 8080:2080 viveticimg
```
RAFRAICHIR locahost:8080
```
docker exec -it  viveticimg1 bash
```
```
exit
```
```
cd ..
```

```
docker-compose 
```
```
docker-compose
```
si non existant : 
```
apt update
```
```
apt install docker-compose
```
LE BUT SURTOUT C'EST PAS DE COPIE COLLER LE CODE DE DOCKER-COMPOSE MAIS L'ECRIRE VOUS MEME

## Exercices Docker compose : 
```
mkdir prog1
```
```
cd prog1
```
```
nano docker-compose.yml
```
```
version: '3'

services:
  myfirstservice:
    image: alpine
    restart: always
    container_name: myalpine
    entrypoint: ps aux
```
Tapez ctrl+x puis yes et entrée
```
docker-compose up -d
```
```
docker images
```
```
docker ps
```
```
docker ps -a
```
```
reboot
```
```
docker ps
```
```
docker-compose ps
```
```
docker-compose exec myfirstservice ps
```
```
reboot
```
```
docker ps
```
```
docker-compose down
```
```
cd ..
```
```
mkdir prog2
```
```
cd prog2
```

Tester en accédant un terminal : 
```
nano docker-compose.yml
```
```
version: '3.8'

services:
  myfirstservice:
    image: alpine
    container_name: myalpine
    entrypoint: sh -c "while true; do sleep 30; done"
```
Tapez ctrl+x puis yes et entrée
```
docker-compose up -d
```
```
docker-compose exec myfirstservice sh
```
```
exit
```



# Exercices 5 :
* Préparation des chemins
```
mkdir web_php
```
```
cd web_php
```
* Code
```
nano index.php
```
```
<p> Bonjour </p>
```
Enregistrer en tapant ctrl+x puis yes puis entrée
* docker-compose.yml
``` 
nano docker-compose.yml
```
```
version: '3'

services:
  php:
    image: php:8.3-fpm
    volumes:
      - .:/var/www/html

  web:
    image: nginx:latest
    volumes:
      - .:/var/www/html
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "8888:80"
```
Enregistrer en tapant ctrl+x puis yes puis entrée
* nginx.conf
```
nano nginx.conf
```
```
events {}

http {

    server {

        listen 80;

        root /var/www/html;
        index index.php;

        location ~ \.php$ {

            fastcgi_pass php:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;

        }

    }

}
```
Enregistrer en tapant ctrl+x puis yes puis entrée
* Vérification des donnés
```
cat index.php
```
```
cat docker-compose.yml
```
```
cat nginx.conf
```
  
* création et lancement de conteneur
```
docker-compose up -d
```
```
docker-compose start
```
```
docker-compose ps
```
```
docker-compose stop
```

```
docker-compose down
```
Tester en tapant localhost:8888 ou curl localhost:8888  
```
curl localhost:8888
```
</br>
Ajouter des erreurs sur nginx puis lancer docker-compose </br>
Ajouter des erreurs sur le code puis lancer docker-compose </br>
Modifier la version de php en 7.4-fpm puis lancer docker-compose </br>

# Exercices 6 : web_php_mysql_nginx
* Préparation des chemins
```
mkdir web_php_mysql_nginx
```
```
cd web_php_mysql_nginx
```
```
mkdir code
```
* NGINX SEULEMENT
* docker-compose.yml
``` 
nano docker-compose.yml
```
```
version : '3'

services:
  nginx:
    image: nginx:1.22-alpine
    ports:
      - "8080:80"
    volumes:
      - ./code:/code
```
Enregistrer en tapant ctrl+x puis yes puis entrée

* tester les dossiers et fichiers
```
ls
```
```
cat docker-compose.yml
```
* Création des services
```
docker-compose up -d
```
RAFRAICHIR -> localhost:8080
```
curl localhost:8080
```
```
docker ps
```
```
docker-compose down
```

* NGINGX + PHP
``` 
nano docker-compose.yml
```
```
version : '3'

services:
  nginx:
    image: nginx:1.22-alpine
    ports:
      - "8080:80"
    volumes:
      - ./code:/code
      # Ajouter dans volumes les configurations
      - ./nginx.conf:/etc/nginx/nginx.conf
 # service php aussi
  php:
    image: php:8.3-fpm-alpine
    volumes:
      - ./code:/code
```
* nginx.conf
```
nano nginx.conf
```
```
events{}

http {

    server {

        listen 80;
        server_name localhost;

        root /code;  # Définition du dossier racine du site
        index index.php;

        location ~ \.php$ {

            fastcgi_pass php:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;

        }

    }

}
```
Enregistrer en tapant ctrl+x puis yes puis entrée
* le code
```
nano code/index.php
```
```
<p> Bonjour sur mon site web </p>
```
Enregistrer en tapant ctrl+x puis yes puis entrée

* tester les dossiers et fichiers
```
ls
```
```
cat code/index.php
```
```
cat nginx.conf
```

```
cat docker-compose.yml
```

* Création des services
```
docker-compose down
```
```
docker-compose up -d
```
RAFRAICHIR -> localhost:8080
```
docker ps
```
```
curl  localhost:8080
```
* NGINX + PHP + MYSQL + PHPMYADMIN
```
nano docker-compose.yml
```
```
version : '3'

services:
  nginx:
    image: nginx:1.22-alpine
    ports:
      - "8080:80"
    volumes:
      - ./code:/code
      # Ajouter dans volumes les configurations
      - ./nginx.conf:/etc/nginx/nginx.conf
 # service php aussi
  php:
    image: php:8.3-fpm-alpine
    volumes:
      - ./code:/code
  #Ajouter services mysql et phpmyadmin
  mysql:
    image: mysql:8
    environment:
      # 🚨 Changer si vous utilisez cette configuration en production
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: appdb
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    volumes:
      - dbdata:/var/lib/mysql
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    environment:
      PMA_HOST: mysql
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "8081:80"

volumes:
  dbdata:

```
Enregistrer en tapant ctrl+x puis yes puis entrée
```
docker-compose down
```
```
docker-compose up -d
```
RAFRAICHIR: localhost:8081 (via interface graphique pour phpmyadmin)

* CONNEXION DB
```
nano code/index.php
```
```
<?php

echo "<p>Bonjour !</p>";

$servername = "mysql";
$username = "user";
$password = "password";
$dbname = "appdb";

try {

    $connexion = new PDO("mysql:host=$servername;dbname=$dbname", $username, $password);

} catch (PDOException $e) {

    echo "Error: " . $e->getMessage();

}
```
Enregistrer en tapant ctrl+x puis yes puis entrée
```
cat code/index.php
```
RAFRAICHIR localhost:8080
```
curl localhost:8080
```
Apparition d'erreur sur le code connexion pdo Error: "could not find driverroot"
* RESOLUTION D'ERREUR DE CONNEXION : DOCKER FILE
```
nano docker-compose.yml
```
```
version : '3'

services:
  nginx:
    image: nginx:1.22-alpine
    ports:
      - "8080:80"
    volumes:
      - ./code:/code
      # Ajouter dans volumes les configurations
      - ./nginx.conf:/etc/nginx/nginx.conf
 # service php aussi
  php:
    # MODIFICATION : pas tag image --> docker file	
    # COMMENTER IMAGE REMPLACER PAR BUILD l'image car on va utiliser Dockerfile ayant FROM  php:8.3-fpm-alpine
    # image: php:8.3-fpm-alpine
    # Il faut utiliser build pour installer via Dockerfile 
    build : .
    volumes:
      - ./code:/code
  #Ajouter services mysql et phpmyadmin
  mysql:
    image: mysql:8
    environment:
      # 🚨 Changer si vous utilisez cette configuration en production
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: appdb
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    volumes:
      - dbdata:/var/lib/mysql
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    environment:
      PMA_HOST: mysql
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "8081:80"

volumes:
  dbdata:
```
Enregistrer en tapant ctrl+x puis yes puis entrée
```
nano Dockerfile
```
```
FROM php:8.3-fpm-alpine
RUN docker-php-ext-install mysqli pdo pdo_mysql
```
Enregistrer en tapant ctrl+x puis yes puis entrée

* Vérification des donnés
```
cat docker-compose.yml
```
```
cat Dockerfile
```
* création et lancement de conteneur
```
docker-compose down
```
```
docker-compose up -d
```
RAFRAICHIR localhost:8080
```
curl localhost:8080
```
PAS D'ERREUR 
* DEVELOPPEMENT
```
nano code/index.php
```
```
<?php

echo "<p>Bonjour !</p>";

$servername = "mysql";
$username = "user";
$password = "password";
$dbname = "appdb";

try {

    $connexion = new PDO("mysql:host=$servername;dbname=$dbname", $username, $password);
    
    $sql = "CREATE TABLE IF NOT EXISTS example_table (
        id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
        fistname VARCHAR(30) NOT NULL,
        lastname VARCHAR(30) NOT NULL
    )";

    $connexion->exec($sql);

    $sql = "INSERT INTO example_table(fistname, lastname)
            VALUE ('John', 'DAC')";

    $connexion->exec($sql);


} catch (PDOException $e) {

    echo "Error: " . $e->getMessage();

}
```
Enregistrer en tapant ctrl+x puis yes puis entrée
Rafraîchir via
```
curl localhsost:8080
```
RAFRAICHIR BD : localhost:8081
```
cd .
```
# INSTALLATION DES 3 MACHINES CLUSTER & HOSTNAME & IN SAME NETWORK 
## Launching manager
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




