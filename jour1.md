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
