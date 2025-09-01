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
