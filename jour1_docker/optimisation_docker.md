## Optimisation de taille de l'image de dockerfile
```
git clone https://github.com/hymaia/handson-container.git
```
```
cd handson-container
```

```
FROM nginx

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
docker run -t -p 8080:80 cactus:step1
```

