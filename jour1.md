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


## Git simple python add actions 

```
git clone https://github.com/iam-veeramalla/GitHub-Actions-Zero-to-Hero.git
```
```
cd GitHub-Actions-Zero-to-Hero/
```
```
echo "# FormationCI_CD_2025_gitactions_python" >> README.md
```

 
## Create github
…or create a new repository on the command line
```
echo "# FormationCI_CD_2025_gitactions_python" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/SitrakaResearchAndPOC/FormationCI_CD_2025_gitactions_python.git
git push -u origin main
```
…or push an existing repository from the command line
```
git remote add origin https://github.com/SitrakaResearchAndPOC/FormationCI_CD_2025_gitactions_python.git
git branch -M main
git push -u origin main
```
