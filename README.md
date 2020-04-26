This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).
## Project Setup
Install Create React App globally:

```
vagrant@sivakumarvunnam:~/Dockerizing-React-App$ npm install -g create-react-app@3.4.1

```
Generate a new app:

```
vagrant@sivakumarvunnam:~/Dockerizing-React-App$ npm init react-app Dockerizing-React-App --use-npm
vagrant@sivakumarvunnam:~/Dockerizing-React-App$ cd Dockerizing-React-App

```

## Production
Let’s create a Dockerfile for use in production called Dockerfile:
```
# build environment
FROM node:13.12.0-alpine as build
MAINTAINER sivakumarvunnam1@gmail.com
WORKDIR /app
ENV PATH /app/node_modules/.bin:$PATH
COPY package.json ./
COPY package-lock.json ./
RUN npm ci --silent
RUN npm install react-scripts@3.4.1 -g --silent
COPY . ./
RUN npm run build

# production environment
FROM nginx:stable-alpine
COPY --from=build /app/build /usr/share/nginx/html
# new
COPY nginx/nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

```
Here, we take advantage of the multistage build pattern to create a temporary image used for building the artifact – the production-ready React static files – that is then copied over to the production image. The temporary build image is discarded along with the original files and folders associated with the image. This produces a lean, production-ready image.

Create the following folder along with a nginx.conf file:

```
└── nginx
    └── nginx.conf
```
nginx.conf:

```
server {

  listen 80;

  location / {
    root   /usr/share/nginx/html;
    index  index.html index.htm;
    try_files $uri $uri/ /index.html;
  }

  error_page   500 502 503 504  /50x.html;

  location = /50x.html {
    root   /usr/share/nginx/html;
  }

}

```

Using the production Dockerfile, build and tag the Docker image:

```
vagrant@sivakumarvunnam:~/Dockerizing-React-App$ docker build -f Dockerfile -t react-app:prod .

```
Spin up the container:

```
vagrant@sivakumarvunnam:~/Dockerizing-React-App$ docker run -itd --rm -p 80:80 react-app:prod

```
Test with a new Docker Compose file as well called docker-compose.yml:

```
version: '3.7'

services:

  sample-prod:
    container_name: react-app-prod
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - '80:80'
```
Fire up the container:

```
vagrant@sivakumarvunnam:~/Dockerizing-React-App$ docker-compose up -d --build

```
