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
