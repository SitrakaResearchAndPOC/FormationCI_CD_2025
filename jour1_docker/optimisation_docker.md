## Optimisation de taille de l'image de dockerfile
```
git clone https://github.com/hymaia/handson-container.git
```
```
cd handson-container
```

```
FROM nginx:1.17.5

# port à exposer pour accéder à l'application
EXPOSE 80

# on installe les outils nécessaire à la construction et à l'exécution
RUN apt update && apt install -y npm nodejs nginx

# on se place dans un dossier de travail et on y copie tout le code de l'application
WORKDIR /app
COPY package* .

# On construit l'application et on la déplace dans le bon dossier pour nginx
RUN npm run build 
RUN cp -r dist/* /usr/share/nginx/html

CMD ["nginx", "-g", "daemon off;"]
```


