# Optimisation de taille de l'image de dockerfile
## Préparation
```
git clone https://github.com/hymaia/handson-container.git
```
```
cd handson-container/frontend
```
## Etape 1
```
cp -rf ../step1.Dockerfile ../frontend/
```
```
FROM nginx:1.27.5

# port à exposer pour accéder à l'application
EXPOSE 80

RUN    apt update && \
    apt install -y npm nodejs && \
    apt clean && rm -rf /var/lib/apt/lists/*

# on se place dans un dossier de travail et on y copie tout le code de l'application
WORKDIR /app
COPY package* ./


# Installer les dépendances node (vite inclus)
RUN npm install
# Copier tout le code source
COPY . .


# On construit l'application et on la déplace dans le bon dossier pour nginx
RUN npm run build 
RUN cp -r dist/* /usr/share/nginx/html

CMD ["nginx", "-g", "daemon off;"]

```
```
docker build -t cactus:step1 -f step1.Dockerfile .
```
```
docker run -t -p 8081:80 cactus:step1
```
## Etape2 
```
cp -rf ../step2.Dockerfile ../frontend/
```
```
FROM nginx:1.27-alpine

# Exposer le port HTTP
EXPOSE 80

# Définir l'environnement pour éviter les prompts
ENV DEBIAN_FRONTEND=noninteractive

# Installer Node.js et npm dans l'image nginx:alpine
RUN apk add --no-cache nodejs npm

# Créer le dossier de travail
WORKDIR /app

# Copier les fichiers de dépendances
COPY package*.json ./

# Installer les dépendances
RUN npm install

# Copier le reste du code source
COPY . .

# Construire l'application (avec Vite ou autre)
RUN npm run build

# Copier les fichiers build dans le dossier HTML de Nginx
RUN cp -r dist/* /usr/share/nginx/html

# Commande de lancement par défaut
CMD ["nginx", "-g", "daemon off;"]
```
```
docker build -t cactus:step2 -f step2.Dockerfile .
```
```
docker run -t -p 8082:80 cactus:step2
```
## Etape3
```
cp -rf ../step3.Dockerfile ../frontend/
```
```
# Étape 1 : Build de l'application avec Node.js (image alpine)
FROM node:18-alpine AS builder

# Crée le dossier de travail
WORKDIR /app

# Copier les fichiers package.json / lock
COPY package*.json ./

# Installer les dépendances
RUN npm install

# Copier le reste du code source
COPY . .

# Construire l’application (Vite, React, Vue, etc.)
RUN npm run build


# Étape 2 : Image de production avec Nginx (Alpine)
FROM nginx:1.27-alpine

# Copier les fichiers compilés depuis l’étape de build vers le dossier nginx
COPY --from=builder /app/dist /usr/share/nginx/html

# Exposer le port 80
EXPOSE 80

# Commande par défaut pour lancer nginx
CMD ["nginx", "-g", "daemon off;"]
```
```
docker build -t cactus:step3 -f step3.Dockerfile .
```
```
docker run -t -p 8081:80 cactus:step3
```
# Docker compose 
```
docker-compose down --volumes --remove-orphans
```
```
docker system prune -f
```
```
docker build up -d
```

* Dockerfile du front
```
# Étape 1 : Build de l'application avec Node.js (image alpine)
FROM node:18-alpine AS builder

# Crée le dossier de travail
WORKDIR /app

# Copier les fichiers package.json / lock
COPY package*.json ./

# Installer les dépendances
RUN npm install

# Copier le reste du code source
COPY . .

# Construire l’application (Vite, React, Vue, etc.)
RUN npm run build


# Étape 2 : Image de production avec Nginx (Alpine)
FROM nginx:1.27-alpine

# Copier les fichiers compilés depuis l’étape de build vers le dossier nginx
COPY --from=builder /app/dist /usr/share/nginx/html

# Exposer le port 80
EXPOSE 80

# Commande par défaut pour lancer nginx
CMD ["nginx", "-g", "daemon off;"]
```
* Dockerfile du backend
```
FROM node:18-alpine
WORKDIR /app

COPY server.js package.json ./
RUN npm install

# EXPOSE 3000
CMD ["node", "server.js"]
```
* docker-compose.yml
```
version: "3"
services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: clickcat_frontend
    ports:
      - "3001:80"               # frontend visible sur localhost:3001
    depends_on:
      - backend

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: clickcat_backend
    ports:
      - "3000:3000"             # backend visible sur localhost:3000
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    depends_on:
      - redis

  redis:
    image: redis:7-alpine
    container_name: clickcat_redis
    ports:
      - "6379:6379"
```
* Tester API :
```
http://localhost:3000/api/use-redis
```
```
http://localhost:3000/api/click
```
```
http://localhost:3000/api/number
```
* Tester Front-End
```
http://localhost:3001
```
# Changer avec network

```
version: "3"
services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: clickcat_frontend
    ports:
      - "3001:80"
    depends_on:
      - backend
    networks:
      - clickcat-network

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: clickcat_backend
    ports:
      - "3000:3000"
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    depends_on:
      - redis
    networks:
      - clickcat-network

  redis:
    image: redis:7-alpine
    container_name: clickcat_redis
    ports:
      - "6379:6379"
    networks:
      - clickcat-network

networks:
  clickcat-network:
    driver: bridge
```
* Log de containers
```
docker-compose logs backend
```
```
docker-compose logs frontend
```
```
docker-compose logs redis
```
* Log du backend
```
docker-compose logs -f backend
```
OU
```
docker ps
```
```
docker logs -f clickcat_backend
```
* Log du service
```
docker-compose logs -f
```
* Debug front-end et back-end
```
docker exec -it clickcat_backend sh
```
* Inspecter reseau :
```
docker network inspect clickcat-network
```
* surveiller reseau
```
sudo tcpdump -i docker0
```
