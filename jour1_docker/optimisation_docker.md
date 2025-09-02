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


